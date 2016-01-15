# Host
* Using KVM to host the following enlisted images.
* Enable share folder.
* sushi

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