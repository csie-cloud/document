# Create a creator

This document describe how to create a create from zero.


## Prerequisite
Install the operating system for the machine. Here we use CentOS7.

## Install and configure Puppet service

Install Puppet server as [this guide](https://github.com/csie-cloud/document/blob/master/Tutorial/Puppet.md).

Before we have DNS server, add the deploy node itself to `/etc/hosts`, so that Puppet can work.
````
127.0.0.1 localhost ... <hostname of deploy node>
````

Here we assume the hostname of deploy node is `deploy`

Tell Puppet agent on the deploy node to contact the Puppet master as deploy node.
````
pupept config set server <hostname of deploy node>
````

## Install and configure r10k

Also install r10k as [this guide](https://github.com/csie-cloud/document/blob/master/Tutorial/r10k.md#install-and-confugure).

## Pull dowm the Puppet script


