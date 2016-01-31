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

## Install Nova
````
yum install openstack-nova-compute sysfsutils
````

## Config Nova
Edit `/etc/nava/nova.conf`, uncomment the lines
````
rpc_backend = rabbit

auth_strategy=keystone


````

````
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
```

Refer to `http://docs.openstack.org/liberty/install-guide-rdo/nova-compute-install.html`