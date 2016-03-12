
Before the start of deployment, there are things that must be done manually:
* Make some decisions:
  * How to divide four networks. Maybe use VLAN?
  * The arrangement of machines. Which machine to host what services?
  * For our decision, check [our architecture in wiki](https://github.com/csie-cloud/wiki/wiki/Architecture).
* Plug in the network cables manully to correct position.
* Set up PXE configuration for your servers, so that they can be deployed automatically.
* If you want further support of IPMI, set up IPMI as well.
* Install and set up Creator somehow, which includes install razor service. The tasks, tags, policies of Razor may require some modification due to difference of hardwares and persional customization.
* Pull down Puppet scripts. Some arguments like password have to be set by yourself.
* Done.
