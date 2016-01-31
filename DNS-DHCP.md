# DHCP setting

DHCP setting is in `/etc/dhcp/dhcpd.conf`
````
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp*/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#

ddns-updates on;
ddns-update-style interim;
update-static-leases on;
authoritative;
include "/etc/rndc.key";
allow unknown-clients;
use-host-decl-names on;
default-lease-time 600;

option domain-name "cloud.csie.ntu.edu.tw";
ddns-domainname "cloud.csie.ntu.edu.tw.";
ddns-rev-domainname "in-addr.arpa.";
option domain-name-servers 192.168.217.250;

zone cloud.csie.ntu.edu.tw. {
     primary 192.168.217.250;
     key rndc-key;
}

zone 217.168.192.in-addr.arpa. {
     primary 192.168.217.250;
     key rndc-key;
}

subnet 192.168.0.0 netmask 255.255.0.0 {
	range 192.168.217.50 192.168.217.99;
	option subnet-mask 255.255.0.0;
	option domain-name-servers 192.168.217.250;
 
	host controller1-ipmi {
		hardware ethernet f0:4d:a2:0a:50:35;
		fixed-address 192.168.215.200;
        	ddns-hostname "controller1-ipmi";
		#option host-name "controller1-ipmi";
	}
    
	host controller1 {
      		hardware ethernet f0:4d:a2:0a:50:2d;
       		fixed-address 192.168.217.200;
        	ddns-hostname "controller1";
        	#option host-name "controller1";
  	}

	if exists user-class and option user-class = "iPXE" {
		filename "bootstrap.ipxe";
	} else {
		filename "undionly.kpxe";
	}
	next-server 192.168.217.250;
}
````

# DNS Setting

`/etc/named.conf`
````

options {
	listen-on port 53 { 127.0.0.1; 192.168.217.250; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { localhost; 192.168.0.0/16; };

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	forwarders {                     
        	140.112.30.21;
        	140.112.30.12;
        	8.8.8.8;
    	};
        
	dnssec-enable yes;
	dnssec-validation yes;

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.iscdlv.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "cloud.csie.ntu.edu.tw" IN {
  type master;
  file "zones/cloud.csie.ntu.edu.tw.zone";
  allow-update { key rndc-key; };               
};

zone "217.168.192.in-addr.arpa" IN {
  type master;
  file "zones/cloud.csie.ntu.edu.tw.rev.zone";
  allow-update { key rndc-key; };               
};

# include "/etc/named.rfc1912.zones";
# include "/etc/named.root.key";
include "/etc/rndc.key";
````