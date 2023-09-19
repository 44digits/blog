+++
title = "How fast is Azure's Network Peering?"
slug = "how-fast-is-azures-network-peering"
date = "2023-09-18 13:56:28 UTC-07:00"
tags = "azure,terraform"
category = "azure"
link = ""
description = "Testing Azure network peering"
type = "text"
+++

Azure offers a way to link virtual networks called
[Virtual Network Peering](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview).
These links utilize Microsft's backbone infrastructure.
Traffic passing through a virtual network peer should therefore be faster than
traffic on the Internet in general.
**But how fast?**
A test was set up attempting to measure the speed improvement between two
locations using virtual network peering.

<!--TEASER_END-->

##Archicture##

This test looks at the latency of a web application on the US west cost as experienced by a user
in India.
Two virtual machines are used.
The first, set up on the west coast, is a web server hosting a large ramdomly made file - **the server**.
The second virtual machine is in India running a rever proxy pointing to the web server - **the proxy**.
Both machines have virtual IPs and both machines are connected through a virtual network peer.

Theoritically a **user** in India accessing the _server_ directly, and downloading the large file should take longer
than accessing the _proxy_.
_The user_ in this experiement is a vm in India created in AWS.
Using a different cloud company ensures that the user is independent of Microsoft's internal network.

[![Network peering](/images/azure-network-peering-img01.thumbnail.png)](/images/azure-network-peering-img01.png)

The client vm contains a bash script `networktest` which measures the time taken to download a given list of URLs "n" number of times.
```
networktest <number of downloads> <URL01> <URL02> ... <URLn>
```
The script also prints out the client's ip address and looks up the city from `ipinfo.io`.
A "wrapper" bash script is also created which calls networktest with the URLs of the random file
pointing directly to the server, and the URL through the reverse proxy.

##Usage##

The code is availabe here: [https://github.com/44digits/CloudRegionSpanning/](https://github.com/44digits/CloudRegionSpanning/)
Usage reqires the installation of the following software:
* Terraform [https://developer.hashicorp.com/terraform](https://developer.hashicorp.com/terraform)
* Azure CLI: [https://learn.microsoft.com/en-us/cli/azure/](https://learn.microsoft.com/en-us/cli/azure/)
* AWS CLI: [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

Once complete create the necessary AWS and Azure accounts.
Environment variables will also need to be set for the Credentials.  Specifically:

```
ARM_CLIENT_ID
ARM_CLIENT_CERTIFICATE_PATH
ARM_CLIENT_CERTIFICATE_PASSWORD
ARM_TENANT_ID
ARM_SUBSCRIPTION_ID
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

Steps:

 * Clone the above github repository and `cd` into the `CloudRegionSpanning` directory.
 * Verify the terraform code with: `terraform verify`
 * Prepare to install with: `terraform plan -out testplan.out`
 * Execute the plan with: `terraform apply testplan.out`

At this point terraform will set up all the virtual machines, create the virtual networks,
and configure the client in AWS.  Once it has completed terraform will print out
the IPs of the server, the proxy, and the IP needed to access the client.  For example:
```
public_ip_address_proxy = "20.204.12.174"
public_ip_address_server = "20.172.8.98"
public_ip_client = "ec2-13-127-122-49.ap-south-1.compute.amazonaws.com"
resource_group_name = "RegionSpanningTest_azure_rg"
```

Log into the AWS client machine with: `ssh -i key/aws_key.pem ec2-user@<public_ip_client>`<br />
And execute the test with: `/tmp/networktest-run`

After experimenting with the tests the infrastructure on both Azure and AWS can be removed with:
```terraform destroy```

##Results##

The above script prints out the results.  For example:
```
Networktest:

Client IP: 13.127.122.49
Client Location: IN / Maharashtra / Mumbai

Destination IP: 20.172.8.98
Destination Location: US / Arizona / Phoenix
..........
Runtime: 35.753 seconds
Avg task time: 3.575 seconds

Destination IP: 20.204.12.174
Destination Location: IN / Maharashtra / Pune
..........
Runtime: 29.466 seconds
Avg task time: 2.947 seconds

Done.
```

These results indicate that downloading the sample file directly from the US based server
required about 3.58 seconds, while accessing the same file, via network peering,
required only 2.95 seconds.
That is a speed-up of 17.5%.

## Conclusion ##

Azure's network peering really does provide a speed up for users in remote locations.
But the speed up is not free.
This pattern requires a separate vm, or maybe container,
to work as a proxy server along with a public IP address.
Fortunately setting up Virtual Network Peering is free.
