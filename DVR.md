# DVR

This document assume that you have installed neutron successfully (refer to openstack installation guide). 

Refer to this [guide](http://docs.openstack.org/liberty/networking-guide/scenario-dvr-ovs.html)

## Prerequisite

Refer to [this site](http://solomon.ipv6.club.tw/Course/SDN/howto_install_ovs_on_centos7.html) and [openvswitch official github](https://github.com/openvswitch/ovs/blob/master/INSTALL.Fedora.md).

### Build openvswitch package

(I built the package on Creator)

Install packages for building the package 
````
yum install -y rpm-build autoconf openssl-devel python-twisted-core python-zope-interface PyQt4 groff graphviz gcc kernel-devel
````

Create the directory for building package
````
mkdir -p ~/rpmbuild/SOURCES 
````

Download the LTS version of openvswitch
````
cd ~/rpmbuild/SOURCES
curl http://openvswitch.org/releases/openvswitch-2.3.2.tar.gz -o openvswitch-2.3.2.tar.gz
tar zxvf openvswitch-2.3.2.tar.gz
````

Fix simbolic link first
````
ln -s /usr/src/kernels/3.10.0-327.4.5.el7.x86_64/ /lib/modules/3.10.0-327.el7.x86_64/build
````

Start to build (here I use --with dpdk to enable dhdk support.)
````
cd openvswitch-2.3.2.tar.gz
rpmbuild --with dpdk -bb rhel/openvswitch-fedora.spec
rpmbuild -bb -D "kversion 3.10.0-327.el7.x86_64" rhel/openvswitch-kmod-fedora.spec
````
()


## Controller node

Since some configurations have been set in Neutron stage, `/etc/neutron/neutron.conf` need not be modified.

In `/etc/neutron/plugins/ml2/ml2_conf.ini`
````ini 
# original: mechanism_drivers = linuxbridge,l2population
mechanism_drivers = openvswitch,l2population 

# Add two lines in [securitygroup] section
[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
enable_security_group = True

# Config sections
[ovs]
local_ip = 10.42.0.241 # TUNNEL_INTERFACE_IP_ADDRESS
bridge_mappings = vlan:br-vlan,external:br-ex # External ?

[agent]
l2_population = True
tunnel_types = vxlan
enable_distributed_routing = True
arp_responder = True

[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
enable_security_group = True
enable_ipset = True
````

=====
Note: can not realize what that means in the guide
````
Start the following services:

    Server
````
=====

Create file `/etc/sysctl.d/50-neutron-dvr-conf`
````ini
net.ipv4.ip_forward=1
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.all.rp_filter=0
````
====

To load new kernel configuration, run
````
sysctl -p
````
====
Edit `/etc/neutron/l3_agent.ini`, replace `[default]` section with
````ini
[DEFAULT]
verbose = True
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
use_namespaces = True
external_network_bridge =
router_delete_namespaces = True
agent_mode = dvr_snat
````
(it will overwrite `interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver` )

====

Edit `/etc/neutron/dhcp_agent.ini`, replace `[default]` section with 
````ini
[DEFAULT]
verbose = True
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
use_namespaces = True
dhcp_delete_namespaces = True

# optional
dnsmasq_config_file = /etc/neutron/dnsmasq-neutron.conf
````

====

`/etc/neutron/metadata_agent.ini` has been configured well, so did not touch it.

====
