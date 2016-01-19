# Host
### Operating System
CentOS 7, kernel default (3.10)

### Abilities
* Using KVM to host the following enlisted images.
* Enable share folder.

### Service
* sshd

# KVM Configuration
* Set up a NAT network `vm-nat`.
* Run `net-update vm-nat add ip-dhcp-host "<host mac='[mac of vm img-test]' ip='192.168.122.2' />" --live --config` to ensure that vm img-test will always get ip `192.168.122.2`.
* Forward ports **TO BE COMPLETED** to VM's port 67, 69, 4011.

# Template Image
### Operating System
CentOS 7, kernel default (3.10) 

### Service
* sshd 
* syslog
* puppet client


# Puppet VM Image
### Operating System
Base system is derived from the template image.

### Service
* Puppet Master

# Netboot VM Image
### Operating System
Base system is derived from the template image.

### Service
* TFTP server
* PXE loader

