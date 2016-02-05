## Prerequisites
Login to the database.  
`mysql -u root -p`  
The password is configured previously after using the secure script.

Create a database named `keystone` and granted to the user `keystone`.  
`CREATE DATABASE keystone;`  
`GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';`  
`GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';`

## Install
Install the packages through the following command.  
`yum install openstack-keystone http mod_wsgi memcached python-memcached`  

Start the `memcached` service.  
`systemctl enable memcached`  
`systemctl start memcached`  

## Configure
Edit `/etc/keystone/keystone.conf` and complete the following section.  

In the `[DEFAULT]` section, define the administration token and enable the verbose output.
`admin_token = ADMIN_TOKEN`  
`verbose = True`  

In the `[database]` section, configure the database access.  
`connection = mysql://keystone:KEYSTONE_DBPASS@controller1/keystone`  

In the `[memcache]` section, configure the service.  
`servers = localhost:11211`  

In the `[token]` section, configure the UUID token provider and the memcached driver.  
`provider = uuid`  
`driver = memcache`  

In the `[revoke]` section, configure the SQL revocation driver.  
`driver = sql`  