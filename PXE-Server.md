# PXE Server

[Reference](http://www.unixmen.com/install-pxe-server-centos-7/)

Install packages required
````
yum install dhcp tftp tftp-server syslinux wget vsftpd
````

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
next-server 172.16.217.140; #  DHCP server ip
filename "pxelinux.0";
}
````