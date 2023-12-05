+++
title = "Coding Environment for the Google Foobar Challenge"
slug = "coding-environment-for-the-google-foobar-challenge"
date = "2023-12-04 16:20:38 UTC-08:00"
tags = "python,foobar,docker"
category = "python"
link = ""
description = "Setting up a Python 2 coding environment with Jupyter Lab and Docker"
type = "text"
+++


The Google Foobar Challenge is a series extremely interesting and fun coding problems.
Lots has been written about the individual levels and problems.
It seems to be a recruiting tool but there is no statement from Google about the
Challenge and it's purpose.
It it very well designed and covers a wide range of computer science tops.
Earlier this year I received an invitation and have
been slowly working through the levels.
If you get an invite take it!

<!--TEASER_END-->

Working on the Google Foobar Challenge is tough on many fronts.
Only one of two coding languages can be used: Java or Python.
Furthermore you are restricted to older versions of these languages.
If you choose Python you need to use version 2.7 and with limited libraries.
Creating a Python coding environment for this Challenge is not easy
because Python 2 is no longer being maintained.
It was
[sunsetted](https://www.python.org/doc/sunset-python-2/)
back in 2020 and is no longer available in some
[Linux distributions](https://wiki.debian.org/Python).

Containerization provides a flexible and convenient way to isolate software
that you want to keep separate from your main operating system.
There are lots of coding tools for Python and
[JupyterLab](https://jupyterlab.readthedocs.io/en/stable/)
is my favorite, providing an excellent environment for prototyping code.

- [https://github.com/44digits/dockerfiles/tree/master/foobar](https://github.com/44digits/dockerfiles/tree/master/foobar)

The link above points to my
`Dockerfile` for creating a JupyterLab container with both Python3 and Python2 kernels.
[`pylint`](https://github.com/pylint-dev/pylint)
and
[`flake8`](https://github.com/pycqa/flake8)
are also installed in case Google gives extra points for conforming to PEP8!

Installation steps:

1. Build the container with:<br>
`docker build -t foobar .`

1. Run the container with:<br>
`docker run --rm --name foobar -v ./notebooks:/code/notebooks -p 8888:8888 foobar`

1. Access JupyterLab in the container with a browser: [http://localhost:8888/lab](http://localhost:8888/lab)

JupyterLab `.ipynb` notebooks are stored in the `notebooks` directory
located in the same directory as the `Dockerfile`.
Docker will create it if it does not exist.
