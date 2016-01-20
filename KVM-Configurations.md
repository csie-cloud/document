# KVM Network
[Netwroking Reference](http://wiki.libvirt.org/page/Networking#NAT_forwarding_.28aka_.22virtual_networks.22.29)

To ensure one VM behind KVM NAT can always get sepcific IP:
````
net-update vm-nat add ip-dhcp-host "<host mac='[mac of vm img-test]' ip='[desired ip]' />" --live --config
```` 

To forward port, run
````
iptables -t nat -A PREROUTING -d [Host_ipaddr] -p tcp --dport [Host_port] -j DNAT --to [Guest_ipaddr]:[Guest_port]
iptables -I FORWARD -d [Guest_ipaddr]/32 -p tcp -m state --state NEW -m tcp --dport [Guest_port] -j ACCEPT
````

Or modify this script
````
#!/bin/bash
# used some from advanced script to have multiple ports: use an equal number of guest and host ports

# Update the following variables to fit your setup
Guest_name=img-test
Guest_ipaddr=192.168.122.2
Host_ipaddr=172.16.217.140
Host_port=(  '10022' '10067' )
Guest_port=( '22' '67' )

if [ "${1}" = "add" ]; then
        length=$(( ${#Host_port[@]} - 1 ))
        for i in `seq 0 $length`; do
                iptables -t nat -A PREROUTING -d ${Host_ipaddr} -p tcp --dport ${Host_port[$i]} -j DNAT --to ${Guest_ipaddr}:${Guest_port[$i]}
                iptables -I FORWARD -d ${Guest_ipaddr}/32 -p tcp -m state --state NEW -m tcp --dport ${Guest_port[$i]} -j ACCEPT
        done
fi

if [ "${1}" = "del" ]; then
        length=$(( ${#Host_port[@]} - 1 ))
        for i in `seq 0 $length`; do
             iptables -t nat -D PREROUTING -d ${Host_ipaddr} -p tcp --dport ${Host_port[$i]} -j DNAT --to ${Guest_ipaddr}:${Guest_port[$i]}
             iptables -D FORWARD -d ${Guest_ipaddr}/32 -p tcp -m state --state NEW -m tcp --dport ${Guest_port[$i]} -j ACCEPT

        done
fi
````

# Share directory

[Reference](http://rabexc.org/posts/p9-setup-in-libvirt)

Use `virsh-edit [domain name]` to add (in devices section)
````
<filesystem type='mount' accessmode='passthrough'>
 <source dir='/tmp/shared'/> 
 <target dir='tag'/>
</filesystem>
````

To mount the directory, 
````
mount -t 9p -o trans=virtio,version=9p2000.L tag /mnt/shared/
````




