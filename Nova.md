## Prerequisites
### Create user in database
Enter the database on the controller.  
`mysql -u root -p`  

Create the `glance` database with appropriate permissions.  
`CREATE DATABASE nova;`  
`GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'NOVA_DBPASS';`  
`GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'NOVA_DBPASS';`

### Create projects and users
Source the administrator credential.   
`source admin-openrc.sh`  

Create the user and add it to `admin` role and the `service` project.  
`openstack role add --project service --user nova admin`  
`openstack service create --name nova --description "OpenStack Compute" compute`  

### Create endpoints
Public  
`openstack endpoint create --region RegionOne compute public http://controller1:8774/v2/%\(tenant_id\)s`  

Internal  
`openstack endpoint create --region RegionOne compute internal http://controller1-int:8774/v2/%\(tenant_id\)s`  

Administration  
`openstack endpoint create --region RegionOne compute admin http://controller1-admin:8774/v2/%\(tenant_id\)s`  

## Install
Go to the compute node.
### Enable the repo
`yum install centos-release-openstack-liberty`  
`yum upgrade`  

Install Openstack related packages.
`yum install python-openstackclient openstack-selinux`  

# Deprecate
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

## Make a hole on the firewall
On both the controller node and the compute node, edit the file `/etc/sysconfig/network-scripts/ifcfg-[dev name]` of internal network. **(The device may be `management` rather than `enp4s0.2`)**. Add one line
````
ZONE=internal
````

And run:
````sh
systemctl restart network
firewall-cmd --permanent --zone=internal --add-interface=management
firewall-cmd --permanent --zone=internal --add-port 5672/tcp

# For dashboard vnc
firewall-cmd --permanent --zone=public --add-port 6080/tcp
firewall-cmd --reload
```` 

## Prepare openstack


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
