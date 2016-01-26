# Why
We need PXE server to deploy the OS to bare metals on the rack. BIOS of each machine is preconfigured to use their designated NIC through either IPMI or direct interface with the interface. Rest of the steps are based on [this](http://geekface.ca/fedora/?q=pxe) tutorial.

# How
## Prequisite
Use the following command to install the required packages.  
`yum install dhcp tftp-server syslinux vsftpd`

## Setup the DHCP service
Edit `/etc/dhcp/dhcpd.conf` to contain the following lines at the beginning.
```
ddns-update-style interim;
ignore client-updates;

authoritative;
allow booting;
allow bootp;
allow unknown-clients;
```

Next, we have to designate the details of our internal network.
```
allow booting;
allow bootp;
allow unknown-clients;

# A slightly different configuration for an internal subnet.
subnet 10.0.0.0 netmask 255.255.255.0 {
	# range 10.0.0.100 10.0.0.249;

	option domain-name-servers 10.0.0.250;
	option routers 10.0.0.250;

	default-lease-time 600;
	max-lease-time 7200;

    host Controller0{
         hardware ethernet f0:4d:a2:0a:50:2f;
         fixed-address 10.0.0.200;
         option host-name "Controller0";
    }             

	# PXE SERVER IP
	next-server 10.0.0.250; # DHCP server ip (Creator-Dev)
	filename "pxelinux.0";
}

```

To specify which interface to listen to, in `/etc/sysconfig/dhcpd` add
```
DHCPDARGS=eno2
```

At last, we enable the DHCP server and open the DHCP port on the firewall. 
`systemctl enable dhcpd`
`systemctl start dhcpd`
`firewall-cmd --add-service=dhcp --permanent`

### Note
In our environment, we are sharing a switch with rest of the machines in R215. If the DHCP packets leak to the actual production environment, we are risking ruin every machines. Therefore, we enforced some rules using `iptables`, to block the DHCP packets on `eno2`.
```
iptables -I INPUT -i eno2 -p udp --dport 67 --sport 68 -j DROP
iptables -I OUTPUT -o eno2 -p udp --dport 68 --sport 67 -j DROP
```
Though, after reviewing the dump from `iptables`, the rules aren't there... Anyway, I'll leave it here.

## FTP service
We use `xinetd` to hook up the vsFTPd service, instead of using `systemd`. Therefore, create a file called `vsftpd` under `/etc/xinetd.d`, with the following contents.
```
service ftp
{
	socket_type	= stream
	wait		= no
	user		= root
	server		= /usr/sbin/vsftpd
	log_on_success	+= HOST DURATION
	log_on_failure	+= HOST
	disable		= no
}
```

We have to configure two of the parameters in `/etc/vsftpd/vsftpd.conf`
```
anonymous_enable=YES
anon_root=/var/ftp
```
The folder `/var/ftp` will be the folder that holds all our images.

After download the ISO in `/tmp`, we extract it and copy the contents into our FTP server.
```
mkdir -p /tmp/extract_iso
mount -t iso9660 -o loop centos7-x86_64-1511.iso /tmp/extract_iso
mkdir -p /var/ftp/centos/7/x86_64
cp -avr /tmp/extract_iso* /var/ftp/centos/7/x86_64/
```

At last, enable the firewall for the FTP port.
`firewall-cmd --add-service=ftp --permanent`

## TFTP service
We use `xinetd` to hook up the TFTP service, instead of using `systemd`. Therefore, create a file called `tftp` under `/etc/xinetd.d`, with the following contents.
```
service tftp
{
	socket_type		= dgram
	protocol		= udp
	wait			= yes
	user			= root
	server			= /usr/sbin/in.tftpd
	server_args		= -s /var/lib/tftpboot
	disable			= no
	per_source		= 11
	cps			= 100 2
	flags			= IPv4
}
```
Do notice that the `server_args` should point to the *boot* images during booting, instead of the full OS image (the one that includes all the repos).

We then open the TFTP port on the firewall and make it permanent.  
`firewall-cmd --add-service=tftp --permanent`

We now dump all the required boot files into the folder, in this case, it's `/var/lib/tftpboot`. Assuming the ISO image itself is extracted already, and placed under the FTP's shared folder create in previous section.
```
mkdir /var/lib/tftpboot
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot
mkdir -p /var/lib/tftpboot/images/centos/7/x86_64
cp /var/ftp/centos/7/x86_64/images/pxeboot/initrd.img /var/lib/tftpboot/images/centos/7/x86_64
cp /var/ftp/centos/7/x86_64/images/pxeboot/upgrade.img /var/lib/tftpboot/images/centos/7/x86_64
cp /var/ftp/centos/7/x86_64/images/pxeboot/vmlinuz /var/lib/tftpboot/images/centos/7/x86_64
```

At last, we copy the boot menu into the TFTP boot folder.
```
cp /usr/share/syslinux/vesamenu.c32 /var/lib/tftpboot
```

## Enable the services
After configuring the `xinetd` related services, start them up at once using  
`systemctl start xinetd`

## PXE Boot Menu
The boot menu will control the loading sequence and the default behaviour, since we want the server to install the image when it enters PXE (which means the primary hard drive failed to the state that no MBR is detectable).  
First create a file called `default` to hold the menu.
```
mkdir /var/lib/tftpboot/pxelinux.cfg
touch /var/lib/tftpboot/pxelinux.cfg/default
```
The following is our `/var/lib/tftpboot/pxelinux.cfg/default`.
```
# Default menu during selection
DEFAULT vesamenu.c32

# Setup the default view
MENU COLOR BORDER 0 #ffffffff #00000000 std
MENU COLOR TITLE 0 #ffffffff #00000000 std
MENU COLOR SEL 0 #ffffffff #ff000000 std
MENU TITLE PXE Boot Menu
PROMPT 0
TIMEOUT 100
ONTIMEOUT server_template

# Local boot
LABEL local
MENU LABEL Boot local HDD
LOCALBOOT 0

# Server template
LABEL server_template
MENU LABEL Server template (CentOS 7 x86_64)
KERNEL images/centos/7/x86_64/vmlinuz
APPEND initrd=images/centos/7/x86_64/initrd.img ks=ftp://10.0.1.250/centos/7/x86_64/ks/ks.cfg
```
The `ks.cfg` is the kickstart file that will get passed into the instal media for automation, which is enlisted in the next section.

## Kickstart
One may use `anaconda-ks.cfg` as the kickstart config file, but we came across some network setup issue, so I decide to rewrite the kickstart file.
```
# Install OS instead of upgrade
install

# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'

# Reboot after installation 
reboot

# Root password
rootpw --iscrypted $6$D5lcLkG5LVX6HSab$iXjh1A7d3sGZImrpDWq8lTmUGFGADiWhMGs3UfubtM4DLFtC9OkUqlL9zjYPGltPTljuLXJm.jmO03Vrc5C4g/

# System timezone
timezone Asia/Taipei --isUtc

# Use network installation
url --url="ftp://10.0.1.250/centos/7/x86_64"

# System language
lang en_US.UTF-8

# Network configuration
network --bootproto=dhcp

# System authorization information
auth --enableshadow --passalgo=sha512

# Use graphical install
#graphical
firstboot --disable

# System services
services --enabled="chronyd"

# System bootloader configuration
bootloader --append=" crashkernel=auto" --location=mbr

# Partition the system
autopart --type=lvm
clearpart --all --initlabel

%packages
@^minimal
@core
chrony
kexec-tools
openscap
openscap-scanner
scap-security-guide
%end

%addon org_fedora_oscap
content-type = scap-security-guide
profile = standard
%end

%addon com_redhat_kdump --enable --reserve-mb='auto'
%end
```