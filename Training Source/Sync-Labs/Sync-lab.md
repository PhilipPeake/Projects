##Sync - LAB

This lab will use the sync server to perform bi-directional synchronization between the existing three (replicating) directory servers, and a fourth directory server not part of the existing replication topology.

Begin by installing a bare 4th directory server:

	cd
	unzip -qq /install/UnboundID-DS-5.*.zip
	mv UnboundID-DS/ ds4
	cd ds4
	./setup --cli --acceptLicense -n -a -p 1489 -w password
	
Install the sync server. This follows the same pattern as proxy, in that setup creates
a bare server which doesn't actually do anything. A setup program is used to
create a dsconfig script which actually creates the configuration:

	cd
	unzip -qq /install/UnboundID-Sync-5.*.zip
	mv UnboundID-Sync/ sync
	cd sync
	./setup --cli --acceptLicense -n -p 1689 -w password
	
Run create-sync-pipe-config

* Continue with configuration
* Standard mode [1]
* Bi-Directional [2]
* UnboundID DS source [1]
* Create an internal name for the source (UBID-Source)
* dc=example,dc=com
* \<Enter> to end list of DNs to sync.
* No security
* \<FQDN>:1189 (Taking ds1 as source, could use any)
* \<FQDN>:1289 (server to fail over to if ds1 fails)
* \<Enter> to finish list
* Take proposed DN for sync user.
* Set password for this user (password)
	
* Destination is Unboundid DS [1]
* Name the destination (UBID-Dest)
* dc=example,dc=com
* \<Enter> to terminate list
* No security
* \<FQDN>:1489
* \<Enter> to terminate list
* Take proposed sync user DN
* Password for sync user (password)
	
* Prepare ds1
* cn=Directory Manager
* password
* Yes - prepare server
* Take default expiry time (2 days) for changelog
	
* Prepare ds2
* Use previously entered credentials
* Configure server (yes)
* 2d expiry for changelog
	
* Prepare ds4
* Add sync user (yes)
* Configure server (yes)
* 2d expiry
	
* Create a name for the sync pipe ds1/ds2 -> ds4 (take default)
* No customization
* 1,2,3
	
* Create a name for ds4 -> ds1/ds2 (default)
* No sync class
* 1,2,3
	
* w (write file)
* Apply changes
* LDAP
* cn=Directory Manager
* password
	
Take a look at *sync-pipe-cfg.dsconfig*

Similarly to the configuration file created for proxy, for future deployments
it may be easier and faster to take this file and edit it.

Take a look as ds4, to confirm that it is empty, apart from the root entry
for example.com:

	./bin/ldapsearch -p 1489 -b dc=example,dc=com -D "cn=Directory Manager" \
	-w password "(objectclass=*)"
	
Now use the resync command to sync data from ds1/ds2 to ds4:

	./bin/resync -p UBID-Source_to_UBID-Dest
	
At completion, ds4 will now contain a copy of the data in ds1/ds2.

Verify using ldapsearch against ds4:

	./bin/ldapsearch -p 1489 -b dc=example,dc=com -D "cn=Directory Manager" \
	-w password "(objectclass=*)" | more
	
Now turn on real-time syncing from ds1/ds2 to ds4 and ds4 to ds1/ds2 (start both pipes).

	./bin/realtime-sync start -p 1689 -w password -n \
	--pipe-name UBID-Source_to_UBID-Dest \
	--pipe-name UBID-Dest_to_UBID-Source
	
Changes made to any of the replicating servers (ds1, ds2, ds3) will
by synced to ds4, and any change made to ds4 will be synced to ds1 (or ds2) and
replicated to the replication set.

Make a change on ds1:

	./bin/ldapmodify -p 1189 -D "cn=directory Manager" -w password <<+
	dn: uid=user.0,ou=people,dc=example,dc=com
	changetype: modify
	replace: description
	description: Change to verify sync
	+
	
Read the user.0 entry on ds4:

	./bin/ldapsearch -p 1489 -b dc=example,dc=com -D "cn=Directory Manager" \
	-w password uid=user.0
	
And test in the reverse direction, by making a change on ds4:

	./bin/ldapmodify -p 1489 -D "cn=directory Manager" -w password <<+
	dn: uid=user.1,ou=people,dc=example,dc=com
	changetype: modify
	replace: description
	description: Change to verify sync from ds4
	+
	
Then read the user.1 entry from ds2:

	./bin/ldapsearch -p 1289 -b dc=example,dc=com -D "cn=Directory Manager" \
	-w password uid=user.1
	
Take a look at the *sync/logs/sync* file to see the entries made as changes are detected and synced.
	
	
	
