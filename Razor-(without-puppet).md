## Install postgresql

[reference](https://wiki.postgresql.org/wiki/YUM_Installation)

To install the latest version of postgresql, add `exclude=postgresql*` in `/etc/yum.repos.d/`'s base and update section.

Then insall the repository,
````
yum localinstall http://yum.postgresql.org/9.5/redhat/rhel-7-x86_64/pgdg-centos95-9.5-2.noarch.rpm
````

Install postgresql
````
yum -y install postgresql95-server
````

Init database and start postgresql
````
/usr/pgsql-9.5/bin/postgresql95-setup initdb
systemctl start postgresql-9.5
````

Create a postgresql user razor and create a database for it
````
su postgres -c 'createuser -P razor'
su postgres -c 'createdb -O razor razor_prd'
````

Edit `/var/lib/pgsql/9.5/data/pg_hba.conf`, changing some `ident` to `md5`, and comment out IPv6 connection.
````
# "local" is for Unix domain socket connections only
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
#host    all             all             ::1/128                 ident
````

Restart postgresql
````
systemctl restart postgresql-9.5
````

To verify configuration, run
````
psql -l -U razor razor_prd
````

## Install razor

## Install PXE

## Install DHCP

