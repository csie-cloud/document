# Neutron

## To create networks that connect to outside world

````
neutron net-create --provider:network_type flat --provider:physical_network external --router:external=true ext-net  
neutron subnet-create  --gateway 172.16.0.1 --name ext-subnet --disable-dhcp --allocation-pool start=172.16.217.150,end=172.16.217.199 ext-net 172.16.0.0/16 
neutron router-create --distributed true demo-router
neutron net-create demo-net
neutron subnet-create --name demo-subnet demo-net 10.0.0.0/24 
neutron router-interface-add demo-router demo-subnet
neutron router-gateway-set demo-router ext-net
````

To clean the networks

````
neutron router-gateway-clear demo-router
neutron router-interface-delete demo-router demo-subnet
neutron router-delete demo-router
neutron net-delete ext-net
neutron net-delete demo-net
````
