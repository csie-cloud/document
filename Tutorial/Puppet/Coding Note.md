# Coding Note

Here are some notes about coding with those puppet module.

## neutron openvswitch issues

The ovs setting done by class `::neutron::agents::ml2::ovs` 
including create bridges, adding interface to bridge, set brodge ip,
are done with class `neutron::plugins::ovs::port` and `neutron::plugins::ovs::bridge`.
There is no much thing to take care about `neutron::plugins::ovs::bridge`, which create
ovs bridge. But problems often occurs when using `neutron::plugins::ovs::port` directly 
or indirectly. 

One thing is that, put `"br-int:br-em2"` in argument `bridge_uplinks` of class 
`::neutron::agents::ml2::ovs` may cause problem. Because before adding a port 
to an ovs bridge, the bridge must exists. However, before class 
`::neutron::agents::ml2::ovs` is applied, `br-int` does not exist. 
Therefore, as the class being applied for the first time,
the bridge `br-int` does not exist if not create explicitly in the code. And what 
argument `bridge_uplinks` of class `::neutron::agents::ml2::ovs` do is add port to the 
ovs bridge in the map. So something like `"br-int:br-em2"` can not be directly put in the 
`bridge_uplinks`.

Also ovs port settings must be applied after network interfaces are set. And VLAN ports (line eno2.42)
and  their parent device (like eno2) can not be add to bridge in paralled. The bottom of ovs
port creation is done with the resource type `ovs_port` of module `vswitch`. If every thing work
harmoniously, this resorce type will set the bridge ip address accoring to the original device
setting in `/etc/sysconfig/network-scripts/ifcfg-***`. Therefore, the first thing to make it work is
that the interface is set well before ovs port setting is applied. However, the resouce type checks
if the interface is up before reading the setting of the interface. So, there is something interesting.
As the ovs port setting is being applied, it will bring down and up both the bridge and the interface.
When a VLAN's parent device is down, the VLAN virtual device is of course down. So if they are set in 
parallel, when VLAN is being applied, since its parent is down, it will not setting its bridge ip! That's
why a VLAN port and its parent port can not be config in paralled. Here my solution is write like below, 
which ensure they are not applied at the same time.

````puppet
  neutron::plugins::ovs::port{ 'br-int:eno2.42':
    subscribe => Class['::network_config'],
    before => Vs_port['eno2'],
  }
````

