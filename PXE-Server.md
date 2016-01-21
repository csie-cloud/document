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

### Note
In our environment, we are sharing a switch with rest of the machines in R215. If the DHCP packets leak to the actual production environment, we are risking ruin every machines. Therefore, we enforced some rules using `iptables`, to block the DHCP packets on `eno2`.
```
iptables -I INPUT -i eno2 -p udp --dport 67 --sport 68 -j DROP
iptables -I OUTPUT -o eno2 -p udp --dport 68 --sport 67 -j DROP
```
Though, after reviewing the dump from `iptables`, the rules aren't there... Anyway, I'll leave it here.

## TFTP Server

Enable tftp server. In `/etc/xinetd.d/tftp`
````
disable                 = no
server_args		= -s /tftpboot
````

Set up TFTP server boot files
````
mkdir -p /tftpboot
chmod 777 /tftpboot
cp -v /usr/share/syslinux/pxelinux.0 /tftpboot
cp -v /usr/share/syslinux/menu.c32 /tftpboot
cp -v /usr/share/syslinux/memdisk /tftpboot
cp -v /usr/share/syslinux/mboot.c32 /tftpboot
cp -v /usr/share/syslinux/chain.c32 /tftpboot
mkdir /tftpboot/pxelinux.cfg
mkdir -p /tftpboot/netboot/
````


Mount image to ftp (with SELinux on)
````
mount -o context=system_u:object_r:public_content_t:s0 /var/tftp/CentOS-7-x86_64-Minimal-1511.iso pub/
````

Prepare image to tftp
````
cp /var/ftp/pub/images/pxeboot/vmlinuz /tftpboot/netboot/
cp /var/ftp/pub/images/pxeboot/initrd.img /tftpboot/netboot/
````

Create PXE menu in `/tftpboot/pxelinux.cfg/default`
````
default menu.c32
prompt 0
timeout 30
MENU TITLE unixme.com PXE Menu
LABEL centos7_x64
MENU LABEL CentOS 7 X64
KERNEL /netboot/vmlinuz
APPEND  initrd=/netboot/initrd.img  inst.repo=ftp://172.16.217.140/pub  ks=ftp://172.16.217.140/ks.cfg
````

## Kickstart

Use anaconda-ks.cfg as kickstart config file. 
````
cp /root/anaconda-ks.cfg /var/ftp/ks.cfg
````