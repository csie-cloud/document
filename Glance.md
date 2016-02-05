## Prerequisites
### Create user in database
Enter the database.  
`mysql -u root -p`  

Create the `glance` database with appropriate permissions.  
`CREATE DATABASE glance;`  
`GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'GLANCE_DBPASS';`  
`GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'GLANCE_DBPASS';`

### Create projects and users
Source the administrator credential.   
`source admin-openrc.sh`  

Create the user and add it to `admin` role and the `service` project.
`openstack role add --project service --user glance admin`  
`openstack service create --name glance --description "OpenStack Image Service" image`  

### Create endpoints
Public  
`openstack endpoint create --region RegionOne image public http://controller1:9292`  

Internal
`openstack endpoint create --region RegionOne image internal http://controller1-int: 9292`  

Administration
`openstack endpoint create --region RegionOne image admin http://controller1-admin: 9292`  

## Make a hole the the firewall
On both compute node and the controller. 
````
firewall-cmd --add-port 9292/tcp --zone=internal --permanent
firewall-cmd --reload
````

# Glance

Install glance on controller.

Install as [openstack install guide](http://docs.openstack.org/liberty/install-guide-rdo/glance-install.html).

For configuration, also follow the guide. Except some variables:
In `/etc/glance/glance-api.conf`
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


````

In `/etc/glance-registry.conf`
````ini
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
### Note
* **`admin_tenant_name` should be set to `service`!!!!** 
 (Otherwise authentication will faill!!)
* Error message `No handlers could be found for logger "oslo_config.cfg"` can be ignored.
