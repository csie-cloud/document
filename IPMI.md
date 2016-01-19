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
6. To have the GRUB messages and boot time messages being dump to the serial console as well, add the following lines to `/etc/default/grub`, please modify the ttySx to the corresponding port in step 5.
```
GRUB_CMDLINE_LINUX_DEFAULT="console=tty0 console=ttyS1,57600n8"
GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --speed=57600 --unit=0 --word=8 --parity=no --stop=1"
```
The service `serial-getty@ttyS1` will automatically get hooked up when the DEFAULT parameters are sent to the kernel during the start up.
7. Run `grub2-mkconfig -o /boot/grub2/grub.cf` after finish modifying the grub template.

# How to use IPMI

## Access Serial Console

`ipmitool -I lanplus -H [ip] -U ipmi-admin -P [password] sol activate`