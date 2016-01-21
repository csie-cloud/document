# PXE Server
~~[Reference](http://www.unixmen.com/install-pxe-server-centos-7/)~~ This reference is obsolete, not to mention some of its articles use _super_ unsafe method to configure the service.  
Use this [one](http://geekface.ca/fedora/?q=pxe) instead.

## Prequisite

Install packages required
````
yum install dhcp tftp tftp-server syslinux wget vsftpd
````

## DHCP Server

Edit `/etc/dhcp/dhcpd.conf`
````
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp*/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
# option definitions common to all supported networks...
ddns-update-style interim;
ignore client-updates;

authoritative;
allow booting;
allow bootp;
allow unknown-clients;

# A slightly different configuration for an internal subnet.
subnet 172.16.217.0 netmask 255.255.0.0 {
range 172.16.217.141 172.16.217.154;

option domain-name-servers 140.112.30.12;
option domain-name-servers 140.112.30.21;
option domain-name-servers 8.8.8.8;
option routers 172.16.0.1;

default-lease-time 600;
max-lease-time 7200;

# PXE SERVER IP
next-server 172.16.217.140; #  DHCP server ip ( Creator-Dev )
filename "pxelinux.0";
}
````

To specify which interface to listen to, in `/etc/sysconfig/dhcpd` add
````
DHCPDARGS="eno2";
````

For better safety, block DHCP on `eno1`
```` 
iptables -I INPUT -i eno1 -p udp --dport 67 --sport 68 -j DROP
iptables -I OUTPUT -o eno1 -p udp --dport 68 --sport 67 -j DROP
````
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