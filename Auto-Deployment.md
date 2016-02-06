# Auto Deployment

We want the whole system can be deployed automatically. In our design, a deployment node, Creator, will do all the dirty work for us. We hope, in the future, we can deploy the whole system by running a few command on Creator, rather than configuring manuallay on each node.  

## Tools used to auto-deployment

### Razor

Refer to our [razor wiki](https://github.com/csie-cloud/wiki/wiki/Razor)

### Puppet
Puppet is a centrallized configuration management tool. The server that holds all the configuration is called the Puppet master, while the server controlled by Puppet master is called Puppet agent. Puppet agent request for new configuration form Puppet master periodical and apply the configuration to the host the agent runs on. Puppet master can give out different configuration to different agent according to some criteria.

## How auto-deployment works

### Before Puppet can take over

The complex configuration dedicated to openstack services are done by Puppet. Howerver, before Puppet can manage those configurations, there must be a proper enviroment with
* Machines with hosts installed.
* Installed puppet agent on nodes to be deployed.
* DNS, DDNS services as the requirement of Puppet.

Partially because of the third requirement, we arranges IP address ranges for different kinds of servers. For example, compute servers should use IP address from `xxx.xxx.xxx.200` to `xxx.xxx.xxx.240`. Therefore, our DNS service on Creator can hard code the relation between IP address and hostname. For example, `xxx.xxx.xxx.200` corresponds to `controller1`, and `xxx.xxx.xxx.235` correspond to `compute36`. A reamain problem is that, how to let server get correct IP address. That will be solve by razor as described below. 

After installation of OS, Razor executes the post installation script to install puppet agent. We do some customization to the script:

1. Modify the repository link to get Puppet 4.3.
2. Config IP address to correct one according to hostname, which is assigned by Razor.
3. Add creator to `/etc/hosts` so puppet agent can talk with puppet master.

After the stage, DHCP service is never required. Therefore, the DHCP service only provide temporary ip address for PXE boot.



