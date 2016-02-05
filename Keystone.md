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

## Configure Keystone
Edit `/etc/keystone/keystone.conf` and complete the following section.  

In the `[DEFAULT]` section, define the administration token and enable the verbose output.
`admin_token = ADMIN_TOKEN`  
`verbose = True`  

In the `[database]` section, configure the database access.  
`connection = mysql://keystone:KEYSTONE_DBPASS@controller1-int/keystone`  

In the `[memcache]` section, configure the service.  
`servers = localhost:11211`  

In the `[token]` section, configure the UUID token provider and the memcached driver.  
`provider = uuid`  
`driver = memcache`  

In the `[revoke]` section, configure the SQL revocation driver.  
`driver = sql`  

### Apply
Populate the Identity service database.  
`su -s /bin/sh -c "keystone-manage db_sync" keystone`  

## Apache HTTP server
### Server name
Edit `/etc/httpd/conf/httpd.conf` and configure the `ServerName` option to reference this controller node.  
`ServerName controller1-admin`  

Note: The server name here points to the primary access URI, therefore, it should be `controller1-admin`, since it has the most privileges.

### The WSGI content
Create `/etc/httpd/conf.d/wsgi-keystone.conf` with the following content.  
```
Listen 5000
Listen 35357

<VirtualHost *:5000>
    ServerAlias controller1
    ServerAlias controller1-int

    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        <IfVersion >= 2.4>
            Require all granted
        </IfVersion>
        <IfVersion < 2.4>
            Order allow,deny
            Allow from all
        </IfVersion>
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    <IfVersion >= 2.4>
      ErrorLogFormat "%{cu}t %M"
    </IfVersion>
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        <IfVersion >= 2.4>
            Require all granted
        </IfVersion>
        <IfVersion < 2.4>
            Order allow,deny
            Allow from all
        </IfVersion>
    </Directory>
</VirtualHost>
```

Note: `ServerAlias` for `controller1` and `controller1-int` are added for different level of endpoint to access port 5000.

### Start the service
`systemctl enable httpd`  
`systemctl start httpd`

## Create service entity and API endpoints
### Setup reusable variables
`export OS_TOKEN=ADMIN_TOKEN`  
`export OS_URL=http://controller1-admin:35357/v3`  
`export OS_IDENTITY_API_VERSION=3`

Note: The `OS_URL` should use the administration network.

### Create the service
`openstack service create --name keystone --description "OpenStack Identity" identity`

### Create endpoints
Public  
`openstack endpoint create --region RegionOne identity public http://controller1:5000/v2.0`  

Internal
`openstack endpoint create --region RegionOne identity internal http://controller1-int:5000/v2.0`  

Administration
`openstack endpoint create --region RegionOne identity admin http://controller1-admin:35357/v2.0`  

## Create projects, users and roles
### Administrative operations
`openstack project create --domain default --description "Admin Project" admin`  
`openstack user create --domain default --password-prompt admin`  
`openstack role create admin`  
`openstack role add --project admin --user admin admin`

### Services
This project will serve for all the services in this cloud.
`openstack project create --domain default --description "Service Project" service`

### Demo
`openstack project create --domain default --description "Demo Project" demo`  
`openstack user create --domain default --password-prompt demo`  
`openstack role create user`  
`openstack role add --project admin --user demo user`
