## Prepare the passwords
### TBA

## OpenStack repository
Since CentOS includes the _extras_ repository by default, we can simply install the package to enable the OpenStack repository.  
`yum install centos-release-openstack-liberty`

### Upgrade
Upgrade the packages on our host, if it's kernel related, consider the necessity before upgrading it.  
`yum upgrade`

### Client tool
Install the OpenStack client and SELinux package to manage the service and the security policies.  
`yum install python-openstackclient openstack-selinux`  

Note: This can take awhile, be patient.

## SQL Database
### Install
We are going to use the same database as most of the tutorials on the Internet.  
`yum install mariadb mariadb-server MySQL-python`

### Configuration
Create a file `/etc/my.cnf.d/mariadb_openstack.cnf` with the following content.  
```
[mysqld]
bind-address = 10.42.0.240
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
```