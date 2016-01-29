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

Make postgresql start when server is up
````
systemctl enable razor-server
````

## Install razor

Install razor server
````
yum install http://yum.puppetlabs.com/puppetlabs-release-el-7.noarch.rpm
yum install razor-server
````

Set production database url in `/etc/razor/config.yaml`. The url should look like `jdbc:postgresql:razor?user=razor&password=mypass`. Also, check the variable `repo_store_root`.

Make a directory for razor's broker.
````
mkdir /var/lib/razor/brokers
````

Initialize database for razor
````
razor-admin -e production migrate-database
````

Start razor
````
systemctl start razor-server
systemctl enable razor-server
````

Download the microkernel in `$repo-store` (the variabable in `/etc/razor/config.yaml`)
````
cd /var/lib/razor/repo-store/
curl -L http://links.puppetlabs.com/razor-microkernel-latest.tar -o razor-microkernel-latest.tar
tar xvf razor-microkernel-latest.tar 
rm -f razor-microkernel-latest.tar
````

## Prepare the TFTP server

Install TFTP server
````
yum install tftp-server
````

Download kPXE firmware
````
cd /var/lib/tftpboot
curl http://boot.ipxe.org/undionly.kpxe -o undionly.kpxe
````

Make the iPXE file 
````
curl http://127.0.0.1:8150/api/microkernel/bootstrap?nic_max=4 -o bootstrap.ipxe
````

To make sure each NIC can get IP by DHCP, add `goto dhcp_net1` in section `:dhcp_net0`, and so on (for 4 sections).

Start and enable tftp
````
systemctl start tftp
systemctl enable tftp
````

## Install DHCP
````
yum install dhcp -y 
````
refer to wiki of DHCP for configuration.

## Install Razor client 