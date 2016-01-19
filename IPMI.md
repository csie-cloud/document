# What can IPMI do

# How to set up IPMI

Refer to this [tutorial](http://www.alleft.com/sysadmin/ipmi-sol-inexpensive-remote-console/).

But note
* The maximum baud rate is 57600
* To start serial console, run `systemctl enable serial-getty@ttyS1`
* COM1 corresponds to ttyS0, and COM2 corresponds to ttyS1.
* To setup grub, set `GRUB_CMDLINE_LINUX_DEFAULT="console=tty0 console=ttyS1,57600n8"`
 in`/etc/default/grub`. And run `grub2-mkconfig -o /boot/grub2/grub.cfg`.

# How to use IPMI

## Access Serial Console

`ipmitool -I lanplus -H [ip] -U ipmi-admin -P [password] sol activate`
