## Introduction to Razor

Razor can be seen as a manager of PXE boot service. It can install different OS, using different installation configuration, performing different post-install script by some facts of the client host. The mechanism of it is
* Boot client machine with a microkernel.
* The microkernel will tell razor the facts (hardware informantion) of the client machine.
* Razor give the machine "tags" by the facts of them. The criterial of the tags is defined by yourself. For example, we defined a tag "poweredge_1950". Machines will get this tag if its fact "productname" is "Poweredge 1950".
* A policy is enforced on the server with specific tag. Policy is also defined by yourself. A policy including properties like hostname, "task", "broker", and so on. You can also decide the policy should work on what tags.
* The "task" is performed. The task is usually installing an OS on the host.
* A post-installation script is run to install the "broker". "Broker" is the configuration management appliction like Puppet. The installation script of broker can be customized by yourself.     


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

Install ruby first
````
yum -y install ruby
gem install razor-client
````

## Make Repo

````
razor delete-repo --name centos7-default
razor create-repo --name centos7-default --iso-url http://centos.cs.nctu.edu.tw/7/isos/x86_64/CentOS-7-x86_64-Minimal-1511.iso --task centos/7
````

## Deployment Configuration 

Creator Broker
````
razor create-broker --name puppet --broker-type puppet --configuration server=192.168.217.250
````

Create tag
````
razor create-tag --name controller --rule '["=", ["fact", "productname"], "PowerEdge R610"]'
````

Create policy. First, write policy json file
````json
{
  "name": "centos7-default",
  "repo": "centos7-minimal-1511",
  "task": "centos/7",
  "broker": "puppet",
  "enabled": true,
  "hostname": "controller${id}",
  "root_password": "--iscrypted hashed_secret",
  "max_count": 3,
  "tags": ["controller"]
}
````
Then run
````
razor create-policy --json policy.json 
````
