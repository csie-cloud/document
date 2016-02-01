
# Glance

Install glance on controller.

Install as [openstack install guide](http://docs.openstack.org/liberty/install-guide-rdo/glance-install.html).

For configuration, also follow the guide. Except some variables:
In `/etc/glance/glance-api.conf`
````
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


````

In `/etc/glance-registry.conf`
````
# Instead of setting auth_url
identity_uri=http://controller:35357

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

````
** `admin_tenant_name` should be set to `service`!!!! ** (Otherwise authentication will faill!!)

# Fail step

````
# glance-manage db_sync glance
No handlers could be found for logger "oslo_config.cfg"
````
````
glance image-create --name "cirros" \
  --file cirros-0.3.4.img \
  --disk-format qcow2 --container-format bare \
  --visibility public --progress

503 Service Unavailable: The server is currently unavailable. Please try again at a later time. (HTTP 503)

````

