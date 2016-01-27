# Prerequite

## Install puppet 
````
sudo rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
yum install puppetserver
systemctl start puppetserver
````
Add `/opt/puppetlab/bin` into path.

In `/etc/puppetlab/puppet/puppet.conf`, set puppet master server to itself, add 
````
server = creator-dev # temporary use the name creator-dev
````

## Install postgresql
````
puppet module install puppetlabs/postgresql
````
Make file `/etc/puppetlabs/code/environments/production/manifests/site.pp`:
````
node default {
  class { 'postgresql::globals':
    manage_package_repo => true,
    version             => '9.2',
    }->

    class { 'postgresql::server': }

    postgresql::server::db { 'razor':
      user     => 'razor',
      password => postgresql_password('razor', 'mypass'),
    }    
}
````