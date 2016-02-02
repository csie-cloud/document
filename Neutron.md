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

In section `[neutron]` in `/etc/nova/nova.conf`, set neither `domain_id` nor `domain_name`.


When setting up [option 2](http://docs.openstack.org/liberty/install-guide-rdo/neutron-controller-install-option2.html), in `/etc/neutron/neutron.conf`, the `[nova]` section only set
````ini
auth_plugin = password
````

On compute, in `/etc/nova/nova.conf`, in the `[neutron]` section, only set
````ini
# Set flag to indicate Neutron will proxy metadata requests and resolve
# instance ids. (boolean value)
service_metadata_proxy=true

# Shared secret to validate proxies Neutron metadata requests (string value)
metadata_proxy_shared_secret = 024ccc26a55059755d04


url=http://controller:9696
````

In `/etc/neutron/neutron.conf` on the compute node,
````ini
[keystone_authtoken]
auth_uri = http://controller:35357/
identity_uri = http://controller:5000
admin_tenant_name = service
admin_user = neutron
admin_password = [openstack neutron password]
````

# Bug fixing!
````
chown neutron /var/lib/neutron/tmp/
chown neutron /var/lib/neutron/tmp/*
chogrp neutron /var/lib/neutron/tmp/
chogrp neutron /var/lib/neutron/tmp/*
````
