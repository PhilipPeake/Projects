## Proxy Installation - LAB

The interactive command-line installation method provides a command line interface and a wizard style driven way to install the Directory Proxy Server. This method will prompt for required information.

Unzip the proxy distribution and move to proxy1.
Repeat for proxy2.

	cd
	unzip -qq /install/UnboundID-Proxy-5*.zip
	mv UnboundID-Proxy proxy1
	unzip -qq /install/UnboundID-Proxy-5*.zip
	mv UnboundID-Proxy proxy2

At this point, we will only be doing the basic setup of the proxy.  
**The proxying configuration will performed later**.

	cd $HOME/proxy1
	./setup --cli --acceptLicense
 
* No existing topology
* Take FQDN or use localhost
* (In AWS the name does not match the internal IP -- accept anyway)
* cn=Directory Manager
* Set password (suggest sticking with "password")
* No SCIM/HTTP
* Port 11389
* No SSL/LDAPS/StartTLS
* Accept all NICs
* No entry balancing
* Manual JVM heap config - 512m
* Start on completion
* Run setup
* Leave config until later - option 3 (Quit)
	 
The basic proxy is now running, but there is no proxying configuration. That will be added later, and proxy 2 will also be configured later.