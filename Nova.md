# Prerequite

## Set up network

Set up interfaces with VLAN...

## Install NTP client
````
yum install chrony
````

In `/etc/chrony`, use Taiwan NTP servers
````
server tock.stdtime.gov.tw
server watch.stdtime.gov.tw
server time.stdtime.gov.tw
server clock.stdtime.gov.tw     
server tick.stdtime.gov.tw
````

## Prepare openstack

Enable openstack repository
````bash
yum install centos-release-openstack-liberty
yum upgrade
````

Install Openstack related packages
````sh
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
````ini
rpc_backend = rabbit

auth_strategy=keystone


````

````ini
rabbit_host = controller
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
```

Refer to `http://docs.openstack.org/liberty/install-guide-rdo/nova-compute-install.html`