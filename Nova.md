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

## On controller
### Install packages
`yum install openstack-nova-api openstack-nova-cert openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler python-novaclient`

### Configure
Edit the `/etc/nova/nova.conf`.  

In the `[database]` section, modify  
`connection = mysql://nova:NOVA_DBPASS@controller1-int/nova`  

In the `[DEFAULT]` section, modify  
```
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.42.0.240
network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.NeutronLinuxBridgeInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver
enabled_apis=osapi_compute,metadata
```

In the `[oslo_messaging_rabbit]` section, modify  
```
rabbit_host = controller1-int
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
```

In the `[keystone_authotken]` section, modify
```
auth_uri = http://controller1:5000
identity_uri = http://controller1-admin:35357
admin_user = nova
admin_password = NOVA_PASS
admin_tenant_name = service
```

In the `[vnc]` section, modify  
`host = controller1`  

In the `[oslo_concurrency]` section, modify  
`lock_path = /var/lib/nova/tmp`  

### Reload the configs
`su -s /bin/sh -c "nova-manage db sync" nova`

### Start the service 
`systemctl enable openstack-nova-api.service openstack-nova-cert.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service`
`systemctl start openstack-nova-api.service openstack-nova-cert.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service`

## On compute node
### Enable the repo
`yum install centos-release-openstack-liberty`  
`yum upgrade`  

Install OpenStack client related packages.  
`yum install python-openstackclient openstack-selinux`  

### Install packages
`yum install openstack-nova-compute sysfsutils`  

### Configure
In the `[DEFAULT]` section, modify  
```
rpc_backend = rabbit
auth_strategy = keystone
my_ip = 10.42.0.200
network_api_class = nova.network.neutronv2.api.API
security_group_api = neutron
linuxnet_interface_driver = nova.network.linux_net.NeutronLinuxBridgeInterfaceDriver
firewall_driver = nova.virt.firewall.NoopFirewallDriver
enabled_apis=osapi_compute,metadata
```

In the `[keystone_authotken]` section, modify
```
auth_uri = http://controller1:5000
identity_uri = http://controller1-admin:35357
admin_user = nova
admin_password = NOVA_PASS
admin_tenant_name = service
```

In the `[oslo_messaging_rabbit]` section, modify  
```
rabbit_host = controller1-int
rabbit_userid = openstack
rabbit_password = RABBIT_PASS
```

In the `[vnc]` section, modify  
```
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller1:6080/vnc_auto.html
```

In the `[glance]` section, modify  
`host = controller1-int`   

In the `[oslo_concurrency]` section, modify  
`lock_path = /var/lib/nova/tmp`  

### Test for hardware
Test for the hardware acceleration support.
`egrep -c '(vmx|svm)' /proc/cpuinfo`  
If the return value is greater or equal then one, then the hardware supports hardware acceleration. Otherwise, modify the `[libvirtd]` section in `/etc/nova/nova.conf` as  
`virt_type = qemu`  
to enable QEMU instead of KVM.

### Start the service
`systemctl enable libvirtd openstack-nova-compute`  
`systemctl start libvirtd openstack-nova-compute`

# Deprecate
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

## Specify the IP that RabbitMQ listen to
In `/etc/rabbitmq/rabbitmq.conf`
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
