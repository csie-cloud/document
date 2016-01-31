# Prerequite

## Set up network

Set up interfaces with VLAN...

## Install NTP client
````
yum install chrony
````
** Maybe more configuration is needed. **

## Prepare openstack

Enable openstack repository
````
yum install centos-release-openstack-liberty
yum upgrade
````

Install Openstack related packages
````
yum install python-openstackclient
yum install openstack-selinux
````

# Nova

## 