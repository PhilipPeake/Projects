##Proxy Configuration - LAB

Currently, all of our directory instances are configured to be in LOCATION1.
Part of this lab is demonstrating failover/routing between two different
locations.

We need to move one of our directory instances (ds2) to a second location.
To do this, we need to do two things:
	
* Add a new location definition to ds2
* Set ds2 location to that newly defined location.
	
To do this, run the following dsconfig commands in the ds2 directory:

	cd $HOME/ds2
	
	./bin/dsconfig create-location --location-name LOCATION2 -p 1289 \
	--bindPassword password -n
	
	./bin/dsconfig set-global-configuration-prop --set location:LOCATION2 -p 1289 \
	--bindPassword password -n
	
When you run the second command to assign the new location, it will remind you
that you need to re-start to have it take effect:

	./bin/stop-ds -R
	
cd back to the proxy1 directory, and run create-initial-proxy-config.

* Accept the proposed proxy user
* Set the password (Suggest sticking with "password")
* No security setup
* dc=example,dc=com base DN
* No entry balancing
* \<Enter> to finish list of base DNs to proxy.
* Enter LOCATION1
* Enter LOCATION2
* \<Enter> to terminate entry of locations.
* LOCATION1 for this proxy.
	
We have two directory servers in LOCATION1, ds1 and ds3.
We now configure access to those servers:

* \<FQDN>:1189
	
We are now asked if we want to prepare this server.
Preparing basically means testing access, creating the proxy user and adding
required privileges for it to proxy.

* 3) Yes, and all subsequent servers
	
* cn=Directory Manager
* password
* Yes, we do want to create the proxy user.
	
* \<FQDN>:1389 (ds3)
* Yes, use previously entered credentials.
* Yes, add proxy user
* <Enter> to terminate adding servers in LOCATION1
	
Begin adding servers for Location2 (there is only one):

* \<FQDN>:1289
* Yes, create proxy user.
* <Enter> to terminate adding servers in LOCATION2
	
* w (write configuration file)
* Yes, apply the configuration.
	
* 1) LDAP
* cn=Directory Manager
* password
	
The configuration file generated by create-initial-proxy-config will now be applied to the proxy.

Take a look at the generated configuration file: *proxy-cfg.dsconfig*
When performing new deployments of proxy servers, it is often easier to take
a copy of one of these files, and edit it to match your requirements than
it is to use create-initial-proxy-config.

The following diagram may help you to make some sense of the configuration you just built:

![Image](http://vogon.net/images/proxy-config.png)

Explore the configuration with dsconfig to locate the various components.