# Puppet

## Installation

(For puppet 4.3.)

### Install puppet server 
````
sudo rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
yum install puppetserver
systemctl start puppetserver
systemctl enable puppetserver
````
Add `/opt/puppetlab/bin` into path.

### Install puppet agent

````
sudo rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
sudo yum install puppet-agent
systemctl start puppet
systemctl enable puppet 
````
Add `/opt/puppetlab/bin` into path.

## Configure Puppet

Puppet configuration can be modified by either modifying the configuration file or using command
````
puppet config set <key> <value>
````

To see where the configuration file locates in,
````
puppet config print confdir
````

There are a few variables worth a look
* server (only on puppet agent)
* certname
* codedir
   
Usually, only `server` need to be modified. The server variable specifies the name of Puppet master server. If the cert name of puppet server is FQDN, then the agent's `server` should also be FQDN.

## Sign cert

Signing cert is simple for Puppet 4.3. To see the cert signing request list
````
puppet cert list
````

To sign a cert
````
puppet cert sign <cert-name>
````

Cert name usually is FQDN, like `compute.cloud.csie.ntu.edu.tw`.

If a Puppet agent is not in the request list, a command can be run on agent to trigger a request
````
puppet agent -t
````

This command can also be used to trigger an update.
 

