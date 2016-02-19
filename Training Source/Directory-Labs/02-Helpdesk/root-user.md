Root User - Lab
---------------

In this lab students will create a new Root User account that can be used to
manage the directory server.

This user will not have any privileges on the directory data itself though and
will only be able to work with the configuration, start, stop, backup, restore,
import and export data. In addition this user can put the server into lockdown
mode.

By creating these users in the main data store this user will be replicated to
all servers once we enable replication so the same root user accounts can be
used across all servers.

When creating root users in the data branch you will also need to create a
Global ACI that allows them to have read/write access to the cn=config branch as
by default only accounts with the bypass-acl privilege would be able to modify
the configuration. We do not want to give that privilege to this account as it
would then be able to bypass ALL access controls in the directory server.

Add the OPER user:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldapmodify -p 1189 -D "cn=directory manager" -w password -a <<+
dn: uid=dsoper1,dc=example,dc=com
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
sn: OPER1
givenName: DS
cn: DS OPER1
userPassword: password
ds-privilege-name: backend-backup
ds-privilege-name: backend-restore
ds-privilege-name: config-read
ds-privilege-name: config-write
ds-privilege-name: disconnect-client
ds-privilege-name: ldif-export
ds-privilege-name: ldif-import
ds-privilege-name: lockdown-mode
ds-privilege-name: server-restart
ds-privilege-name: server-shutdown
+
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next we will want to add an Operations group so we can assign rights to this
group of special users. The Operations group allows you to add more root user
accounts and simply add them to the group for them to get access to the
configuration. They still need the privileges added to their account though,
this could be automated by the use of virtual attributes, but for the purposes
of thislab, we will keep things simple.

Add these groups to the directory:

Add a groups OU

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldapmodify -p 1189 -D "cn=directory manager" -w password -a <<+
dn: ou=groups,dc=example,dc=com
objectClass: top
objectClass: organizationalUnit
ou: groups

dn: cn=Directory Operators,ou=Groups,dc=example,dc=com
objectClass: groupofuniquenames
objectClass: top
uniqueMember: uid=dsoper1,dc=example,dc=com
cn: Directory Operators
+
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next we need to add a global ACI. Global ACI's are stored in the *cn=config*
branch, not in the main data.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/dsconfig set-access-control-handler-prop -p 1189 -w password -n \
--add 'global-aci:(target="ldap:///cn=config")(targetattr="*")(version 3.0; acl "Allow DS OPERS access to config"; allow (all) groupdn="ldap:///cn=DirectoryOperators,ou=Groups,dc=example,dc=com";)'
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/dsconfig set-access-control-handler-prop -p 1189 -w password -n \
--add 'global-aci:(target="ldap:///cn=tasks")(targetattr="*")(version 3.0; acl "Allow DS OPERS access to tasks"; allow (all) groupdn="ldap:///cn=Directory Operators,ou=Groups,dc=example,dc=com";)'
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We can now test this account by using it to build a new index.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/dsconfig create-local-db-index --backend-name userRoot \
-p 1189 -w password -n \
--index-name mobile \
--set index-type:equality
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note that this only creates the index structure, it does not scan the databaseto
populate it. That needs to be done separately, and with a large databasethis
might be better performed during off-peak hours.

We will rebuild it now:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/rebuild-index -p 1189 -D "uid=dsoper1,dc=example,dc=com" \
-w password --baseDN dc=example,dc=com --index mobile --task 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

NOTE: We perform this as a task rather than stopping the server to rebuild.This
example completes quite quickly. With a larger database re-building an indexmay
take hours. It is quite safe to kill the process (cntrl-C) once rebuilding has
started. It will continue to run within the directory server.
