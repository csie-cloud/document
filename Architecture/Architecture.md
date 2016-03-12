# Architecture
The architecture refer to [mirantis's example architecture](https://docs.mirantis.com/openstack/fuel/fuel-7.0/reference-architecture.html)

## Network architecture
![Architecture](http://csie-cloud.github.io/document/images/arch-dvr.svg?)

There are four networks in the system.
* Management
* Public
* Management
* Storage

Each network is a VLAN in the on-rack switch. Since some of our machines have only 2 NICs, we use VLAN tags to seperate the networks on the NICs that connect to more than one network. The switch's port for the NIC that connects to more than one network will "redirect" the packets to corresponding VLAN according to the VLAN tag on the packet. 

## Networks

### Administration Network

This network is for PXE boot, IPMI and Puppet service when auto deployment. Each interface in this network get IP address in `192.168.0.0/16`. On Creator, the DHCP server give out IP address accroding to MAC address. Each server of specific purpose (ex. compute node) will get specific IP address. Since Puppet require both DNS and reverse DNS to work, there is also a DNS service on Creator. The DNS server only provide resolution of hostname to IP addressed that are used by Puppet, namely `192.168.217.0/24`.

### Management Network

This network is for communication between openstack components as well tunnled traffic form compute instances. We plan to use VXLAN for the tunneling.

### Public Network

This network is VLAN 7 of CSIE, which connect to CISE's NAT. This network provide virtual routes on compute nodes with access to public network. This is the only network that connects to outside world. 

### Storage Network

This network carries  Ceph replication traffic. 

## Hostname

Each  refer to an interface. And since a host often has three or four interface, multiple FQDN refer to a host. That is because we want traffic of different kinds of service goes on the network they should go on. Take `compute1` as example, our naming rule is that:
  * `compute1`: interface connecting to public network. (172.16.217.0/16)
  * `compute1-int`: interface connecting to management (internal) network. (10.42.0.0/24)
  * `compute1-admin`: interface connecting to administration network. (192.168.217.0/24)
  * `compute1-storage`: interface connecting to storage network. (10.41.0.0/24)
So when config openstack message service that exchange message between components like neutron, nova, url should be the internal one. While API endpoints or dashboard interface url should be the public one.

## Domain name

Our domain name is `cloud.csie.ntu.edu.tw`.
