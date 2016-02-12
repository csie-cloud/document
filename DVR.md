# DVR

Note:
* This document assume that you have installed neutron successfully (refer to openstack installation guide). 
* This document install network node and controller node on single machine.
* The configuration only supports tenant network type VXLAN. Using VLAN may require different configuration. 

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
````
It is said that openvswitch has been merged into linux kernel mainline, so building openvswitch kernel module is not needed.

## Controller node

Install packages 
````
yum install openvswitch openvswitch-2.3.2-1.el7.centos.x86_64.rpm
yum install openstack-neutron-openvswitch
````

In `/etc/neutron/neutron.conf` add one line in `[default]` section make distributed default when creating router
````ini
[default]
router_distributed = True
````

In `/etc/neutron/plugins/ml2/ml2_conf.ini`
````ini 
# original: mechanism_drivers = linuxbridge,l2population
mechanism_drivers = openvswitch,l2population 

# Add two lines in [securitygroup] section
[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
enable_security_group = True

# Config sections
[ml2_type_flat]
flat_networks = external

[ovs]
local_ip = 10.42.0.241 # TUNNEL_INTERFACE_IP_ADDRESS
bridge_mappings = external:br-ext #

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

In `/etc/neutron/plugins/ml2/openvswitch_agent.ini`, config
````ini
[ovs]
...
local_ip = 10.42.0.241
bridge_mappings = external:br-ext

[agent]
...
tunnel_types = vxlan
l2_population = True
arp_responder = True
enable_distributed_routing = True
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
sysctl -p /etc/sysctl.d/50-neutron-dvr-conf
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

Start and restart some services
````
systemctl start openvswitch neutron-openvswitch-agent
systemctl restart neutron-server neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent
````

====

**Open VSwitch bridege `br-ext` have to be created by yourself!!**
Commands may like this
````
ovs-vsctl add-br br-ext
ovs-vsctl add-port br-ext eth0
````
where eth0 is the interface connecting to outside world.

## Compute node

Install packages
````
yum install openvswitch-2.3.2-1.x86_64.rpm
yum install openstack-neutron-openvswitch

# For openvswitch 
mkdir /etc/openvswitch
semanage fcontext -a -t openvswitch_rw_t "/etc/openvswitch(/.*)?"
restorecon -Rv /etc/openvswitch 
````

====

Create file `/etc/sysctl.d/50-neutron-dvr.conf`
````ini
net.ipv4.ip_forward=1
net.ipv4.conf.default.rp_filter=0
net.ipv4.conf.all.rp_filter=0
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
````

To load the kernel configuration, run
````
sysctl -p /etc/sysctl.d/50-neutron-dvr-conf
````
====

Create file `/etc/neutron/plugins/ml2/ml2_conf.ini`
````ini
[ovs]
local_ip = 10.42.0.200
bridge_mappings = vlan:br-vlan,external:br-ext

[agent]
l2_population = True
tunnel_types = vxlan # Turn off gre here
enable_distributed_routing = True
arp_responder = True

[securitygroup]
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
enable_security_group = True
enable_ipset = True
````
====


In `/etc/neutron/plugins/ml2/openvswitch_agent.ini`, config
````ini
[ovs]
...
local_ip = 10.42.0.200
#bridge_mappings = external:br-ext
# ^ don't know if is needed
[agent]
...
tunnel_types = vxlan
l2_population = True
arp_responder = True
enable_distributed_routing = True
````


====

In `/etc/neutron/l3_agent.ini`
````ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.OVSInterfaceDriver 
# ^ Default value is empty

use_namespaces = True
# ^ Uncomment it

external_network_bridge =
# ^ Original value is br-ex

router_delete_namespaces = True
# ^ Uncomment it

agent_mode = dvr
# ^ Default value is legacy
````

====

`/etc/neutron/metadata_agent.ini` is find, did not modify it.

====

Start and restart some services
````
systemctl start openvswitch neutron-openvswitch-agent 
systemctl restart neutron-l3-agent neutron-metadata-agent
````

Stop linuxbridge-agent
````
systemctl stop neutron-linuxbridge-agent
````

## Verify operation

Output of `neutron agent-list | grep compute1`
````
| 3ab7d18c-2b35-4163-be33-1f43cecfced2 | Linux bridge agent | compute1    | xxx   | True           | neutron-linuxbridge-agent |
| 9d17cb81-f05a-4826-9aef-174d2c2200d6 | Open vSwitch agent | compute1    | :-)   | True           | neutron-openvswitch-agent |
| a1722bde-9367-4005-ad2e-8a93e1b34618 | Metadata agent     | compute1    | :-)   | True           | neutron-metadata-agent    |
| b101540e-919e-48c8-812a-bf125a350d6f | L3 agent           | compute1    | :-)   | True           | neutron-l3-agent          |
````

Output of `neutron agent-list | grep controller2`
````
| ba6360af-f61d-4ad5-ad2c-05c8ba7a1867 | Metadata agent     | controller2 | :-)   | True           | neutron-metadata-agent    |
| ea3e50e9-2a8f-46b3-ad11-1dd0d9f1488c | DHCP agent         | controller2 | :-)   | True           | neutron-dhcp-agent        |
| edeebdab-539d-41a9-a1fe-dd023cae172c | L3 agent           | controller2 | :-)   | True           | neutron-l3-agent          |
| fa878b89-702e-422d-91df-6f0c7e1c5ad0 | Open vSwitch agent | controller2 | :-)   | True           | neutron-openvswitch-agent |
````

#### Create initial external network

Commands are like below
````
source admin-openrc.sh
neutron net-create ext-net --router:external   --provider:physical_network external --provider:network_type flat
neutron subnet-create ext-net 172.16.0.0/16 --allocation-pool   start=172.16.217.100,end=172.16.217.200 --disable-dhcp   --gateway 172.16.0.1
neutron net-create vm-net --provider:network_type vxlan --tenant-id <id-of-project-demo>
neutron router-create --distributed True gate --tenant-id <id-of-project-demo>
neutron router-gateway-set gate ext-net
````

For more detail, refer to [official guide](http://docs.openstack.org/liberty/networking-guide/scenario-dvr-ovs.html#verify-network-operation)

After an instance connect to the demo subnet, and every thing goes well, then `ovs-vsctl show` should output something like:
````
    Bridge br-ext
        Port phy-br-ext
            Interface phy-br-ext
                type: patch
                options: {peer=int-br-ext}
        Port br-ext
            Interface br-ext
                type: internal
        Port "eno2"
            Interface "eno2"
    Bridge br-int
        fail_mode: secure
        Port int-br-ext
            Interface int-br-ext
                type: patch
                options: {peer=phy-br-ext}
        Port "tap280050a7-91"
            tag: 5
            Interface "tap280050a7-91"
                type: internal
        Port patch-tun
            Interface patch-tun
                type: patch
                options: {peer=patch-int}
        Port "sg-1b13faff-40"
            tag: 5
            Interface "sg-1b13faff-40"
                type: internal
        Port br-int
            Interface br-int
                type: internal
        Port "eno2.42"
            Interface "eno2.42"
        Port "qg-5310b590-05"
            tag: 3
            Interface "qg-5310b590-05"
                type: internal
        Port "qr-85928959-10"
            tag: 5
            Interface "qr-85928959-10"
                type: internal
    Bridge br-tun
        fail_mode: secure
        Port "vxlan-0a2a00c8"
            Interface "vxlan-0a2a00c8"
                type: vxlan
                options: {df_default="true", in_key=flow, local_ip="10.42.0.241", out_key=flow, remote_ip="10.42.0.200"}
        Port br-tun
            Interface br-tun
                type: internal
        Port patch-int
            Interface patch-int
                type: patch
                options: {peer=patch-tun}
    ovs_version: "2.3.2"
````