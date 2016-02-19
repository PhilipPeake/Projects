Replication LAB
---------------

In previous LABs we have made changes to ds1, such as adding indexes etc. To
make life simple, we want the directory instances to have identical
configurations.

The easiest way to achieve this, is to shut down ds1, remove it, and re-install
it. This will be faster than undoing previous changes.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cd $HOME
ds1/bin/stop-ds
rm -rf ds1

unzip -qq /install/UnboundID-DS-5*.zip
mv UnboundID-DS ds1

cd ds1
./setup --cli --acceptLicense -n -d 2000 -p 1189 \
    -w password --maxHeapSize 512m -L
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For the first replication setup, use the interactive mode. This gives a clearer
picture of what is happening.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/dsreplication -p 1189 - 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The menus/selections can be a bit intimidating at first exposure.

- Manage the Topology [2]

- select Enable replication -- add a server) [1]

- Continue

-   Proposed hostname should be correct.

-   LDAP connection (we didn't configure SSL) [1]

-   Specify the port for ds1 (1189)

- Global administrator name. At this point, we have not created an admin account, so we have to use cn=Directory Manager.

- Password for admin (stick with "password")

- The second server hostname will be the same (both are on the same machine in
this case, in the real world, they will be on different hosts).

- LDAP [1]

-   Specify the port for ds2 (1289) [Look carefully here!!!]

- Global administrator name. At this point, we have not created an admin account, so we have to use cn=Directory Manager.

- "password"

-   Choose which base DN to replicate (there is only one in this case).

-   We will not be performing entry balancing.

-   Remember from the table in the installation module, we will be assigning
    replicationports similarly to LDAP ports (1190, 1290, 1390 ... etc). So for
    ds1 (first server)we will use 1190.

-   Let all servers be WAN gateway with default priority (5).

-   Assign a location. This is just a name, usually the name of the data
    center,but it can be anything -- suggestion: LOCATION1
    
- Replication port for server 2 [1290]

- Wan gateway [yes]

- Gateway priority [5]

-   Second server is in the same location.

Next we have to specify a global administrator account this is used by various
tools that might modify configuration which needs to be consistent across all
instances in the replication topology.

-   Global account name: admin

-   Password: (stick with "password")

When replication setup completes, quit dsreplication.

At this point, replication is established, but won't go into operation until
both servers contain identical data. replication will then work to ensure that
both remain identical.

- Quit dsreplication [q]

Check the replication status:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/dsreplication status -p 1189 --adminUID admin \
    -w password --no-prompt --showall --displayservertable
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note that we used the global admin account, because this is guaranteed to have
access to all servers in the topology.

Take a look at the replication log (*logs/replication*)

You will see notification that generation IDs are not equal, and so replication
is currently suspended.

You will also notice that admin and schema data has been exported - this is used
to sync both directories with identical data in these backends. You don't have
to set up replication for schema and admin data, its automatic.

The next step is to initialize ds2 with the user data from ds1.

	./bin/dsreplication

-   Initialize over the network (option 3).
- Continue

Initialize one server, or all servers. In this case its the same, but in
general, as we add an additional server we will initialize just that one.

-   Select 1 (single server)

- Select source server (this host)
- LDAP [1]

-   Select the server containing the data (port 1189).
- Admin user [admin]

- password"

- Select source server (port 1198) [1]

- Admin user (second server) [admin]
- password

-   Select the backend to replicate (only one choice here).

-   Select the server who's backend database you want to overwrite (1289)

-   Quit dsreplication when finished.

- Global admin [admin]
- password
- Yes - overwrite [yes]
- Wuit when complete [q]


Check logs/replication on ds2 and you will see the messages indication
initialization.

Re-run the dsreplication status command. You will now see both servers having
the same generation ID and entry count.

We will now test replication. Start by modifying user.1999 to replace the attribute description with text
indicating a modification made from ds1:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldapmodify -p 1189 -D "cn=directory manager" -w password <<+
dn: uid=user.1999,ou=People,dc=example,dc=com
changeType: modify
replace: description
description: This is a modify done on DS1
+
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Check that this change has replicated to ds2:

```
./bin/ldapsearch -p 1289 -D "cn=directory manager" -w password \
    -b dc=example,dc=com -s sub "(uid=user.1999)" dn description
```

Next  modify user.1998, indicating a change made from
ds2:

```
./bin/ldapmodify -p 1289 -D "cn=directory manager" -w password <<+
dn: uid=user.1998,ou=People,dc=example,dc=com
changeType: modify
replace: description
description: This is a modify done on DS2
+
```

Use ldapsearch to check on ds1 to ensure the change was reflected there:

```
./bin/ldapsearch -p 1189 -D "cn=directory manager" -w password \
    -b dc=example,dc=com -s sub "(uid=user.1999)" dn description
```

We will now add a third server, and add it to the established replication
topology.

Create ds3 (unzip ..., mv ..., setup - port 1389):

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cd
unzip -qq /install/UnboundID-DS-5.*.zip
mv UnboundID-DS/ ds3
cd ds3
./setup --cli --acceptLicense -n -a -p 1389 -w password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add to replication topology in non-interactive mode:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/dsreplication enable --host1 127.0.0.1 --port1 1189 \
--bindDN1 "cn=Directory Manager" --bindPassword1 password \
--replicationPort1 1190 \
--host2 127.0.0.1 --port2 1389 \
--bindDN2 "cn=Directory Manager" --bindPassword2 password \
--replicationPort2 1390 --baseDN dc=example,dc=com --adminUID admin \
--adminPassword password --no-prompt --ignoreWarnings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note that we are cheating here slightly by using localhost (127.0.0.1). In
practice you will specify the hostnames of your instances. In our case, all
instances are listening to localhost, so we can just use that. dsreplication
will complain about using the loopback port. You may want to replace 127.0.0.1
with the FDQN of your machine. If you do this, you can omit the --ignoreWarnings
flag.

Run dsreplication status again to see that the third server has been added to
the topology (but not yet initialized).

We are going to initialize ds3 manually, using a binary backup taken from ds1.
In the ds1 directory, you should have a "backups" directory from previous labs.
If not, just create it. If it does exist, remove the contents (just to avoid any
confusion about which backup is which)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cd $HOME/ds1
rm -rf backups/*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Take a binary backup:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/backup --backupDirectory backups --backUpAll
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

List the contents of "backups", you will see on directory for each of the server
backends.

You can't restore all of this into ds3. Some of it is specific to the config of
ds1, and only makes sense in a restoration of ds1. What we need to initialize
ds3 are the userRoot and replicationChanges backends.

If we had configured encryption of the userRoot backend, you would need to take
the encryption-settings backend too, otherwise the restored database would be
unreadable (the whole point of encryption).

Restore ds3 using ds1 backup:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cd $HOME/ds3
./bin/stop-ds
./bin/restore --backupDirectory ../ds1/backups/userRoot/
./bin/restore --backupDirectory ../ds1/backups/replicationChanges
./bin/start-ds
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When running stop-ds, you may receive some warning notices about generation IDs
being out of sync.

Run dsreplication status again. You will now see all three instances with the
same generation ID and the same entry count.

You will also see that ds3 does not have a location, and because its in its own
location it is also acting as a replication gateway.

Fix this with dsconfig:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/dsconfig create-location --location-name LOCATION1 \
    -p 1389 --bindPassword password -n

./bin/dsconfig set-global-configuration-prop --set location:LOCATION1 \
    -p 1389 --bindPassword password -n
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Check replication status again. Now we have two systems that think they are the
replication gateway. This should resolve when replication traffic flows, but the
easiest way to resolve this is to re-start ds3:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/stop-ds -R
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
