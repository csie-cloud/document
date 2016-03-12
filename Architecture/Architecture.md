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

This network is for inter-communication between openstack services as well tunnled traffic form compute instances. We plan to use VXLAN for the tunneling.

### Public Network

This network is VLAN 7 of CSIE, which connect to CISE's NAT. This network provide virtual routes on compute nodes with access to public network. This is the only network that connects to outside world. 

### Storage Network

This network carries  Ceph replication traffic. 

## Hostnames
We encounter a problem while configurating those service. All services, including those for deploying, for storage, for inner API calls, refer other servers with hostname. If each server have only one hostname, and each hostname of course corresponding to single IP, then all service will communicate with each other in single network. For example, both Puppet, Nova-compute may referred "controller1" to controller host. So, in our DNS zone file, there have to be a record correspond "controller1" to an IP address, say 192.168.217.240, the IP of administration network. So the traffic between both "Puppet-Master, Pupper-Agent" and "Nova-compute, Keystone" go through the administration network. That's fine for Puppet, since the administration network is there for Puppet. Howerver, the traffic between Nova-compute and Keystone should goes through Management network. 
Therefore, we come up a solution: more than one hostname (FQDN) may refer to a single server, but each hostname corresponds to a logical network interface (such as virtual interface for VLAN tag). The rule is that,
* the hostname used in administration network is
  * `creator` if the host is creator, since creator only connect to administration network.
  * `xxx${id}-ipmi` if it is a ipmi interface
  * `xxx-host${id}`. Ex. `compute-host1`.
* the hostname used in administration network is `xxx${id}`. Ex. `compute1`.
* In public network, only few hosts require hostname, and the hostnames are public.

