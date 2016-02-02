**Some steps may be missed!**

Follow the [official installation guide](http://docs.openstack.org/liberty/install-guide-rdo/neutron-controller-install.html), and choose [option 2](http://docs.openstack.org/liberty/install-guide-rdo/neutron-controller-install-option2.html). 

Some configurations are different from the guide:

In `/etc/neutron/neutron.conf`, set
````ini
auth_uri = http://controller:5000

identity_uri = http://controller:35357
admin_suer = neutron
admin_password = openstack_neutron
admin_tenant_name = service
````

When setting up [option 2](http://docs.openstack.org/liberty/install-guide-rdo/neutron-controller-install-option2.html), in `/etc/neutron/neutron.conf`, the `[nova]` section only set
````ini
auth_plugin = password
````

On compute, in `/etc/nova/nova.conf`, in the `[neutron]` section, only set
````ini
url=http://controller:9696
````


