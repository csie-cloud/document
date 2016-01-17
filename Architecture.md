# Architecture

## Overview

The architecture is designed to utilize the resources we have in maximum and provide a secure and high-available infrastructure that can be deployed easily. Some services are run on a single server together in order to reduce the number of phisical servers needed. But if the scale is going larger and larger in the future, it is very possible to isolate them and even balance their load using softwares like HAProxy. When any one controller node fail, software like Pacemaker will react instantly, which ensures that there is no single point of failure. Duplicate storage node also provide sufficient redundant. To esure the service is stable all the time, there is a development enviroment aside from the production one to experiment new features before deploying to production. The two systems are completely seperated, but configure management tool like Puppet facilitate us to syncronized them if needed. 

## Development Enviroment

Developemet enviroment is a miniature of production one. This system is consisted of two physical server. One for Creator, the server that provides auto-deployment. The other one host lots of virtual machines to simulate the production enviroment. The simulated production enviroment enable us to experiment new configuration or new feature without under the risk of crash production enviromment. And the isolated Creator server can let us experiment auto deployment one bare phisical machine moreover.
![](http://csie-cloud.github.io/wiki/images/dev-env.svg)

## Production Enviroment 
![Production](http://csie-cloud.github.io/wiki/images/arch.svg)