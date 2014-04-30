# Getting Started With GearD

## Background on Getting Started

This document is focused on how to use GearD to deploy complex container based applications. It is 
intended for users that will try to deploy complex applications. Therefore it is not intended to 
show you how to install and run a simple example.  

It is assumed that the reader has some basic knowledge of using Docker. For more information on 
getting started with Docker please refer to Fedora's [Getting started with Docker]<https://fedoraproject.org/wiki/Getting_started_with_docker> and ... (some link here to docker run examples)

Before diving into GearD it's important to understand some of the more advanced features of Docker, 
including integration with SELinux and Systemd. The next three sections look at:

* SELinux
* Integrated logging with the host
* Systemd integration

## SELinux enforcing is important

The SELinux integration with Docker provides greater containment. Consider an example where a user 
runs a container and bind mounts `/etc/shadow`. Because this file contains valuable password
information and a docker container intentionaly runs as root, then this file might be compromised
by the unauthorized Docker user.

However SELinux allows us to enforce some rules on containers and even though the container runs as 
root, restrictions can be applied.

Start two VMs (host1 and host2) that are running RHEL 7, or Fedora, with Docker. Install Docker and 
GearD on both hosts:

    # yum install -y docker-io geard

Show selinux is enabled on both nodes by running `getenforce` on both.

    # getenforce

Set enforcement to permissive on host2:
 
    # setenforce permissive

Now run a fedora container and show the running processes:

    # docker run -t -i fedora:20 bash
    
    bash-4.2# ps -eZ

Notice the SELinux policy associated with the process.

Exit and rerun the container but this time bind mount `/etc/shadow` into container. Do this on each 
host. Remember, host2 has selinux in permissive mode. Exit when done.

    # docker run -i -t -v /etc/shadow:/etc/shadow fedora:20 bash
    bash-4.2# cat /etc/shadow
    bash-4.2# id
    bash-4.2# exit


Notice that on  the host (host2) that has selinux disabled you are able to access `/etc/shadow`. 
However you are not able to access `/etc/shadow` on host1 where selinux is enforced. 

SELinux provides more containement to Docker containers.

## Logging to the hosts log files

When deploying complex applications it is important to be able to monitor and debug some of the 
complex interactions between the various components in the different containers that are running the
application. Instead of examining each containers log file it would be more convenient if container
log information was showing up on a single log file on the host. The recommended way to do this is to
use the systemd journal.

First run a container with the systemd journal bind mounted:

    # docker run -i -t -v /dev/log:/dev/log fedora:20 bash

Now write a message to the systemd journal and exit:

    bash-4.2# logger "Log from inside my awesome container"
    bash-4.2$ exit

Search for the message in the host's systemd journal:

    # journalctl -b | grep -i "Log from inside my awesome"

You will see your message in the host's SystemD journal.

## SystemD integration

