## Prerequisites
Login to the database.  
`mysql -u root -p`  
The password is configured previously after using the secure script.

Create a database named `keystone` and granted to the user `keystone`.  
`CREATE DATABASE keystone;`  
`GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS';`  
`GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS';`