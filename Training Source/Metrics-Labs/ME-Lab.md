##Install Metrics Engine

Unzip metrics engine distribution.
Rename to "metrics"
Run setup:

* Install Postgres: yes
* Postgres port: 5432
* Postgres data: pgsql_data
* Postgres admin password: password
* Postgres account: metricsengine
* Change password: no
* FQDN: \<FQDN>
* *** On AWS the installation may query this - respond yes
* ME Root user: cn=Directory Manager
* Password for directory manager: password
* Port: 8443
* LDAP port: 2389
* LDAPS: no
* Start TLS: no
* Self-signed certificate [1]
* Specify specific interface: no
* Memory:3 (Manual)
* Memory: 2g
* Memory too small: yes
* Start server: yes
* Set up server: 1


Use dsconfig to create a Location (LOCATION1).
In global configuration set this as the ME location.

```
./bin/dsconfig -n -p 2389 -D "cn=Directory Manager" \
	-w password create-location \
	--location-name LOCATION1
	
./bin/dsconfig set-global-configuration-prop -n \
	-p 2389 -D "cn=Directory Manager" -w password \
	--set location:LOCATION1
```
	
Restart

Verify that install is working by opening *https://\<FQDN>:8443/view/ldap-dashboard*

Click on LOCATION1 and you should see some stats (not many...) for the metrics engine itself (yes, the Metrics Engine keeps statistics on itself).

Using the monitored-servers tool is the easiest way to set up monitoring
of a set of replicating directory servers since it uses the information
it discovers in the first server that you specify to add all of the others.

To see what the actions will be, start by using the *--dry-run* option:

```
./bin/monitored-servers add-servers -p 2389 \
	--bindDN "cn=directory manager" \
	--bindPassword password \
	--monitoringUserBindPassword password \
	--remoteServerHostname localhost \
	--remoteServerPort 1189 \
	--remoteServerBindPassword password \
	--dry-run
```

When you are satisfied with what it is doing, run again without the dry-run
option to ass the monitoring user to all the directory servers and configure
monitoring of these servers within the metrics engine.

Give the metrics engine a few moments to begin collecting data, then return to viewing *http://\<FQDN>:8443/view/ldap-dashboard*

Add the sync server:

```
./bin/monitored-servers add-servers -p 2389 \
	--bindDN "cn=directory manager" \
	--bindPassword password \
	--monitoringUserBindPassword password \
	--remoteServerHostname localhost \
	--remoteServerPort 1689 \
	--remoteServerBindPassword password
```
	
Allow a few moments for the metrics engine to begin gathering data, then refresh the browser and use the pull-down menu to view sync data.
Add proxies:

```
./bin/monitored-servers add-servers -p 2389 \
	--bindDN "cn=directory manager" \
	--bindPassword password \
	--monitoringUserBindPassword password \
	--remoteServerHostname localhost \
	--remoteServerPort 11389 \
	--remoteServerBindPassword password
```
	
Again, allow time for collection of data to begin and refresh the view
of the metrics engine dashboard, and look at proxy data via the
pull-down menu.

Apply a light load, with everything running on the same server it will be easy
to drive the CPU usate to 100%:

```
cd ../proxy1
./bin/modrate -p 11389 --bindDN "cn=directory manager" \
	--bindPassword password \
	--entryDN "uid=user.[1-1999],ou=people,dc=example,dc=com" \
	--attribute description \
	--valueLength 12 \
	--numThreads 1 -r 10
```

Look at the stats for proxy (LOCATION1 in this example), directories and sync.

You may notice that there are some servers that appear in "Unknown locations".
You may want to add location information to those servers.

