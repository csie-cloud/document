# Auto Deployment

We want the whole system can be deployed automatically. In our design, a deployment node, Creator, will do all the dirty work for us. We hope, in the future, we can deploy the whole system by running a few command on Creator, rather than configuring manuallay on each node.  

## Tools used to auto-deployment

### Razor

Refer to our [razor wiki](https://github.com/csie-cloud/wiki/wiki/Razor)

### Puppet
Puppet is a centrallized configuration management tool. The server that holds all the configuration is called the Puppet master, while the server controlled by Puppet master is called Puppet agent. Puppet agent request for new configuration form Puppet master periodical and apply the configuration to the host the agent runs on. Puppet master can give out different configuration to different agent according to some criteria.

We also use r10k to help version control. For more information, refer to our [r10k tutorial](https://github.com/csie-cloud/wiki/wiki/r10k#why-r10k).

## How auto-deployment works

### Before Puppet can take over

The complex configuration dedicated to openstack services are done by Puppet. Howerver, before Puppet can manage those configurations, some requirements must be satisfied:
* Machines with OSs installed.
* Installed puppet agent on nodes to be deployed.
* DNS, DDNS services as the requirement of Puppet.

Partially because of the third requirement, we arranges IP address ranges for different kinds of servers. For example, compute servers should use IP address from `xxx.xxx.xxx.200` to `xxx.xxx.xxx.240`. Therefore, our DNS service on Creator can hard code the relation between IP address and hostname. For example, `xxx.xxx.xxx.200` corresponds to `controller1`, and `xxx.xxx.xxx.235` correspond to `compute36`. A reamain problem is that, how to let server get correct IP address? That will be solve by razor as described below. 

After installation of OS, Razor executes the post installation script to install puppet agent. We do some customization for the script:

1. Modify the repository link to get Puppet 4.3.
2. Config IP address to correct one according to hostname, which is assigned by Razor.
3. Add creator to `/etc/hosts` so puppet agent can talk with puppet master.

After the stage, DHCP service is never required. Therefore, the DHCP service only provide temporary ip addresses for PXE boot.

### After Puppet take over

Puppet will configure network interfaces permanently as well as set up services on the machine.

## Work with Puppet

### code architecture

* The main manifest is on repository [csie-cloud/cloud](https://github.com/csie-cloud/cloud)
* Each category of node (eg. controller, compute, ...) is a module.
* Where should be network configuration be placed in has not been decided. 
Temporarily, it is put in the main manifest. But we may have some choices
  * Put in an dedicated module, and use Hiera to decide which host to use which configuration. 
  * Put in an dedicated module, and use some `if`, `else`, `case` to decide which host to use which configuration.
  * As part of each node module.
* An dedicated module `password` that only contains password.

All except `password` is on public github repository ([csie-cloud](https://github.com/csie-cloud/)). The dedicated module `password` is to collect all sensitive information, so other repository can be public.

### Using r10k

We use r10k to help us do version control. One thing need to be mensioned is that, since r10k does not resolve dependency automatically for the modules specified in Puppetfile, a tool may be needed to generate module dependencies. The tool used for now is [generate-puppetfile](https://github.com/rnelson0/puppet-generate-puppetfile). The basic usage is simple:
````
generate-puppetfile openstack/keystone razorsedge/network
````

### Creating module

Use `puppet module generate 217-<module-name>`

I use `217` as author.