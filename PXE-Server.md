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
subnet 10.0.1.0 netmask 255.255.255.0 {
        range 10.0.1.100 10.0.1.150;

        option domain-name-servers 10.0.1.250;
        option routers 10.0.1.250;

        default-lease-time 600;
        max-lease-time 7200;

        # PXE SERVER IP
        next-server 10.0.1.250; # DHCP server ip (Creator-Dev)
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

## TFTP Server
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

## Kickstart

Use anaconda-ks.cfg as kickstart config file. 
````
cp /root/anaconda-ks.cfg /var/ftp/ks.cfg
````