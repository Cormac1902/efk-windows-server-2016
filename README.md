# EFK for Windows Server 2016
## Description
This allows you to deploy an EFK Stack on a Windows 2016 Server running version 1607, in Windows containers, using Docker.

## Installation
1. Make changes for NAT on the last line of the Compose file and for each individual container if necessary (see file).
1. Run build.bat.

It may be the case that the containers will not be able to see each other - if this happens, I was able to fix it with the following steps:
1. Comment out the `network:` section from each container in the Compose file.
1. Remove the created network (`efk_network` by default).
1. Run `docker-compose up --build`.
1. Once the above step has finished, you should be able to un-comment the `network:` sections in the Compose file to assign static internal IP addresses to the containers.