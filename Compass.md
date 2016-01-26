
## Install figlet manually

Compass require figlet (to show "Compass"). But default repo doesn't include it and neither is Rpoforge. Therefore, we have to install it manually. 
````
wget http://pkgs.repoforge.org/figlet/figlet-2.2.2-1.el6.rf.x86_64.rpm
yum install figlet-2.2.2-1.el6.rf.x86_64.rpm
````

## Install Compass
````
yum -y install git
git clone https://github.com/openstack/compass-core
cd compass-core
./install/install.sh
````
Then it will ask lots of questions
````
Please enter the nic (Example: eth0):
eno2
You have entered eno2
Please enter the ipaddr (Example: ):
10.0.0.250
You have entered 10.0.0.250
Please enter the netmask (Example: ):
255.255.255.0
You have entered 255.255.255.0
Please enter the option_router (Example: 172.16.0.1):

Default option_router '172.16.0.1' chosen
Please enter the ip_start (Example: 10.0.0.100):

Default ip_start '10.0.0.100' chosen
ip start address is 10.0.0.100
Please enter the ip_end (Example: 10.0.0.250):

Default ip_end '10.0.0.250' chosen
there will be at most 150 hosts deployed.
Please enter the nextserver (Example: 10.0.0.250):

Default nextserver '10.0.0.250' chosen
Please enter the nameserver_domains (Example: ods.com):
cloud.csie.ntu.edu.tw
You have entered cloud.csie.ntu.edu.tw
Please enter the nameserver_reverse_zones (Example: unused):

Default nameserver_reverse_zones 'unused' chosen
Please enter the web_source (Example: http://git.openstack.org/openstack/compass-web):
https://github.com/openstack/compass-web
You have entered https://github.com/openstack/compass-web
Please enter the adapters_source (Example: https://gerrit.opnfv.org/gerrit/compass4nfv):
https://github.com/openstack/compass-adapters
````
