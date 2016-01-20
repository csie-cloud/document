# What can IPMI do
IPMI stands for _Intelligent Platform Management Interface_ -- it's a standard for monitoring and controlling the server remotely and independently through the BMC (Baseboard Management Controller). We primarily use the IPMI to perform the following task:  
1. Control the power status remotely, i.e. reset, power on.
2. View the system log that's stored in the on-board chip regarding in-chassis sensors.
3. Access the machine without the primary NIC online (through Serial over LAN, SOL).

# How to set up IPMI
Please check [here](http://www.alleft.com/sysadmin/ipmi-sol-inexpensive-remote-console/) out for the complete tutorial. The following are the condensed procedures. 

### Dell
1. Use `Ctrl + E` to enter the configuration page.
2. Enable `IPMI over LAN`. 
3. Configure the LAN parameters, i.e. static IP address, netmask, gateway address, _user access_.
4. To enable SOL, reboot and use `F2` to enter the setup page.
5. Setup the serial port multiplexers on the motherboard properly, avoid conflicts here.
6. To have the GRUB messages and boot time messages being dump to the serial console as well, add the following lines to `/etc/default/grub`, please modify the `ttySx` to the corresponding port in step 5.  
 `GRUB_CMDLINE_LINUX_DEFAULT="console=tty0 console=ttyS1,57600n8"`  
 `GRUB_TERMINAL=serial`  
 `GRUB_SERIAL_COMMAND="serial --speed=57600 --unit=0 --word=8 --parity=no --stop=1"`  
 The service `serial-getty@ttyS1` will automatically get hooked up when the DEFAULT parameters are sent to the kernel during the start up.
7. Run `grub2-mkconfig -o /boot/grub2/grub.cf` after finish modifying the grub template.

### Note
* The maximum valid baud rate for the IPMI is 57600bps, though the interface provides the options for 115200bps.
* Remember to regenerate the `grub.cfg` through GRUB utilities, and _never_ manually modify GRUB related files.
* In _Shared mode_, IPMI can only function transmit data through NIC 1, though it can receive data on both NIC 1 and NIC 2. While in _Fail back_, incoming data stream will be directed to NIC 2 when NIC 1 failed to function properly. The [source](http://lists.us.dell.com/pipermail/linux-poweredge/2006-September/027244.html) of this information.

# How to use IPMI
## Access serial console
* Enable/Disable the SOL ability to the user on specified channel (controlled by IRQ).  
 `ipmitool -H [ip] -U [username] -P [password] payload status`  
 `ipmitool -H [ip] -U [username] -P [password] payload enable 1`  
 `ipmitool -H [ip] -U [username] -P [password] payload disable 1`  
 The configurations above assume that only 1 channel is valid here, and only 1 user with ID 1 is allowed. The user ID can be viewed when the `status` command is issued.
* Start/Stop the console, only 1 user at a time, deactivate the SOL session when finished the operation.  
 `ipmitool -I lanplus -H [ip] -U [username] -P [password] sol activate`  
 `ipmitool -I lanplus -H [ip] -U [username] -P [password] sol deactivate`  

## Control the power
It' rather straight forward, using the following commands.
```
ipmitool -H [ip] -U [username] -P [password] power status
ipmitool -H [ip] -U [username] -P [password] power on
ipmitool -H [ip] -U [username] -P [password] power off
ipmitool -H [ip] -U [username] -P [password] power reset
ipmitool -H [ip] -U [username] -P [password] power soft
```
Please differentiate between hard reset and soft reset, try not to use the former one.