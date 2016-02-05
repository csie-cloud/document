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
`connection = mysql://keystone:KEYSTONE_DBPASS@controller1/keystone`  

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
`ServerName controller1`  

### The WSGI content
Create `/etc/httpd/conf.d/wsgi-keystone.conf` with the following content.  
```
Listen 5000
Listen 35357

<VirtualHost *:5000>
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

### Start the service
`systemctl enable httpd`  
`systemctl start httpd`