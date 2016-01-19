# What can IPMI do

# How to set up IPMI

Refer to this [tutorial](http://www.alleft.com/sysadmin/ipmi-sol-inexpensive-remote-console/).

But note
* The maximum baud rate is 57600
* To start serial console, run `systemctl enable serial-getty@ttyS1`

# How to use IPMI

## Access Serial Console

`ipmitool -I lanplus -H [ip] -U ipmi-admin -P [password] sol activate`
