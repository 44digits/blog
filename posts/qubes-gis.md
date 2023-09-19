+++
title = "Qubes GIS Client & Server"
slug = "qubes-gis"
date = 2023-08-20
tags = "gis,qgis,geoserver,qubes,bash,postgis"
category = "qubes"
#link = 
description = "Installation script for QGIS, Geoserver and PostGIS"
type = "text"

+++


TL;DR: This bash script creates a desktop GIS AppVM Qube and a server AppVM Qube within QubeOS. 
All software is installed into a single TemplateVM. 
The software includes QGIS, PostGIS, GeoServer and other utilities. 
Firewall/iptables rules are setup to enable the desktop Qube to reach to server Qube. 
This script will work with both Debian and Fedora TemplateVMs.

<!--TEASER_END-->

 * [https://gist.github.com/44digits/e8452c4d3b796da29168f90e9edaf620](https://gist.github.com/44digits/e8452c4d3b796da29168f90e9edaf620)


TS;WM: About four years ago I switched to QubesOS. 
While I am still not sure if I’m more productive, 
the switch has forced me to learn a lot of things I didn’t expect to learn 
and has definitely made me a better Linux administrator. 
New users are forced to consider all of their workflows and applications, 
and decide how they should be compartmentalized on dedicated Qubes. 
I have leaned about iptables, the Xen hypervisor, code signing with gpg, 
and how to write bash scripts like this to automate installations and automate system management. 
The switch has definitely made my system more secure and better organized.

## Qubes OS

In brief, “Qubes” are virtual machines. They have their own operating system, applications, user space, memory, CPU, and other resources. They are also strongly isolated using xen-based virtualization. There are different types or classes of Qubes. AppVMs are used to run applications, work on documents, browse, and so on. What makes them more secure is that they inherit their operating system, and /usr/bin applications, from a TemplateVM at boot time. These directories can be modified, but changes are not persistent. Multiple AppVMs can use the same TemplateVM but they cannot permanently alter it.
GIS on Qubes

For GIS, and certainly for GIS software development, Qubes offers some great features:

    Strict security model
    All disks are encrypted
    Qubes can be quickly cloned and backed up
    Qube templates can be easily upgraded
    Multiple Qubes can use the same template, streamlining operating system updates, and standardizing development environments
    “Disposable” Qubes can be created for testing external applications and plugins without affecting existing software and security.

For example, temporary AppVMs can be used for testing QGIS plugins in a secure environment without having access to data or other resources.

The tight security model of Qubes OS comes with some challenges:

    Strict security model between Qubes complicates some workflows - i.e.: copying text and files between Qubes
    Qubes are isolated and cannot communicate with each other: IPtables rules must be added to both the network Qube and the server Qube in order to enable networking
    Qube state cannot be preserved: Qubes need to be shutdown when the host computer is shutdown

## Creating GIS Qubes

This bash script is designed to navigate some of these challenges. 
A TemplateVM is created containing both desktop and server software. 
The software list includes the following packages:

    Geoserver
    PostgreSQL
    PostGIS
    QGIS
    pgAdmin

Two AppVMs are created: the desktop GIS Qube and the server GIS Qube. 
IPtables rules are created to allow the desktop GIS Qube to access the Server GIS Qube. 
These rules are made persistent by adding them to the startup scripts of the GIS server and the NetVM Qube. 
The script also defines the PostGIS and GeoServer configuration directories as 
writable on the server to ensure they are preserved after shutdown.

## Installing and Running

In order to install and run this script it must be downloaded and run in dom0. 
Download via a web browser into any Qube with internet access. 
To transfer the script to dom0, and make it executable, use the commands:

    qvm-rum -p <source qube> 'cat path/to/download/make_gis_qubes' > make_gis_qubes
    chmod +x make_gis_qubes

Before running, review the global variables at the top of the script. Specifically:

    MASTERTEMPLATE: your master template for AppVMs, for example Debian-9 or Fedora-30
    GISDESKTOPVM: name of GIS desktop Qube
    GISSERVERVM: name of GIS server Qube
    GEOSERVERURL: the URL for Geoserver download
    GISSOFTWARE_DEBIAN / GISSOFTWARE_FEDORA: add packages to install here if you are using Debian or Fedora

Finally remeber it is risky to run anything in Dom0; it can affect the security of your whole system. 
Please ensure you fully understand this script before using it.
