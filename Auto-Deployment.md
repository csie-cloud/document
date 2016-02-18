# Auto Deployment

We want the whole system can be deployed automatically. In our design, a deployment node, Creator, will do all the dirty work for us. We hope, in the future, we can deploy the whole system by running a few command on Creator, rather than configuring manuallay on each node.  

## Tools used to auto-deployment

### Razor

Refer to our [razor wiki](https://github.com/csie-cloud/wiki/wiki/Razor)

### Puppet
Puppet is a centrallized configuration management tool. The server that holds all the configuration is called the Puppet master, while the server controlled by Puppet master is called Puppet agent. Puppet agent request for new configuration form Puppet master periodical and apply the configuration to the host the agent runs on. Puppet master can give out different configuration to different agent according to some criteria.

### r10k
r10k is a tool that help us manage the Puppet manifests and modules with git. With r10k, we can set up repositories for each module. r10k help us pull them down and put them into right place automatically. What modules to be pull down is decided by the dependency stored in file `Puppetfile` of each module. That sounds like `puppet module install` can also do the same thing. But the most attractive part is that, those dependencies can be a few git repository, while the `puppet module install` can only download dependency from puppet forge. So the result is that, you can have lots of modules stored in lots of repository on github. When you want to deploy, you only have to set up the main repository location, and then a sigle command `r10k deploy environment -p` will pull down all the modules you need! Another feature of r10k is that r10k will set up environments for each branch. So it is possible to set up a "test" environment without affecting the production one. 

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



