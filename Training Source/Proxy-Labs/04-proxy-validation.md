##Second Proxy Setup and Failover Test - LAB

Setting up proxies serving the same set of directory servers is much easier
for the second and subsequent proxies. You specify an existing proxy and
the new server will fetch the configuration from that.

Besides the basic port and directory manager user/password, the only
other information we have to supply is the location and directory manager
credentials of the server:

	cd $HOME/proxy2
	./setup -n --acceptLicense -p 12389 -D "cn=Directory Manager" \
		-w password --peerHostName <FQDN> \
		--peerPort 11389 --location LOCATION2
	
We can now validate this server in the same way that we did the first.
begin by verifying that you can perform an ldapsearch via this proxy:

	./bin/ldapsearch -p 12389 -b dc=example,dc=com "(uid=user.0)"
	
If that succeeds, run a few thousand searches with searchrate:

	./bin/searchrate -p 12389 -D "cn=directory manager" -w password \
	-b dc=example,dc=com -f "(uid=user.[0-1999])" -A sn -A uid -t 10
	
(If the system becomes too slow, you may want to kill the searchrate job
and re-start it with fewer threads. The VMs used for training are much less
capable than a typical deployment platform).

Then run the proxy2 status command.

	./bin/status -p 12389 --maxAlerts 0 -w password -n
	
Examining the output, you will see that all requests have been directed to
its "local" directory server (ds2).

Now we can test failover. To do this we will again need traffic through proxy1:
	
	./bin/searchrate -p 11389 -D "cn=directory manager" -w password \
	-b dc=example,dc=com -f "(uid=user.[0-1999])" -A sn -A uid -t 10

Open a second terminal session, and while searchrate is running, run a couple of 
successive status commands on proxy1 to verify that traffic is going only to ds1 and
ds3 and that counts for both are increasing.

	./bin/status -p 11389 --maxAlerts 0 -w password -n
	
* Stop ds1, and do a couple more runs of status on proxy1.
You will see a high priority alert indicating that one of the external servers
(ds1) has become unavailable, and that traffic is only flowing to ds3.

* Stop ds3.

* Run proxy1 status a couple more times to determine that traffic has now
failed over to ds2 (LOCATION2).

Re-start ds1.

* Run proxy1 status a couple of times to verify that traffic has switched back
to that server (LOCATION1).

----

If time permits, you may want to repeat the same validation on proxy2.


