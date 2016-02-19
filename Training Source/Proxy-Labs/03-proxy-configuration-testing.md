##Proxy Configuration Testing - LAB

Step1 is to verify that traffic is being sent to both servers in LOCATON1 from
proxy1.

Begin by doing an ldapsearch to verify connectivity between proxy and directory servers.

	cd $HOME/proxy1
	./bin/ldapsearch -p 11389 -b dc=example,dc=com "(uid=user.0)"
	
The next step is to verify that traffic is being distributed as we specified,
balanced across the two servers in LOCATION1. To do this, we need to generate some
traffic, and the easiest way to do that is to run searchrate. It doesn't need to
run long, just enough to generate a few thousand searches (stop it with cntrl-C):

	./bin/searchrate -p 11389 -D "cn=directory manager" -w password \
		-b dc=example,dc=com -f "(uid=user.[0-1999])" -A sn -A uid -t 10
		
Next, run the proxy1 status command (make sure it is the proxy status command
that you run, and not a directory status command):

	./bin/status -p 11389 --maxAlerts 0 -w password -n
	
The --maxAlerts 0 option suppresses listing of recent alerts. These will include
configuration changes, and since we just made quite a few, the alerts list will
likely scroll what you really want to see off the top of the screen.

Look at the "LDAP External Server Op Counts" table, the "search" column. You should see
counts for the directory servers ds1 and ds3. ds2 should have taken no traffic.
By default, the proxy will avoid sending traffic to another location if it can
