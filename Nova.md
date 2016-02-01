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

## Install rabbitmq on the controller node

Refer to [official document](http://docs.openstack.org/liberty/install-guide-rdo/environment-messaging.html). 

To specify the IP that RabbitMQ listen to, in `/etc/rabbitmq/rabbitmq.conf`
````
[
  {rabbit, [
    {tcp_listeners, [{"10.0.217.200", 5672}]}
  ]}
].
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
Basically refer to [the official configuration guide](http://docs.openstack.org/liberty/install-guide-rdo/nova-compute-install.html). Except some variables:

In `/etc/nava/nova.conf`,
````ini
[keystone_authtoken]
...
auth_uri = http://controller:5000

# Instead of setting auth_url
identity_uri=http://controller:35357

# set auth_strategy instead of auth_plugin
auth_strategy = password

# Not found
# project_domain_id = default
# user_domain_id = default
# project_name = service
# Set those instead
# Service username. (string value)
admin_user=glance

# Service user password. (string value)
admin_password=[openstack glance user password]

# Service tenant name. (string value)
admin_tenant_name=service
```

