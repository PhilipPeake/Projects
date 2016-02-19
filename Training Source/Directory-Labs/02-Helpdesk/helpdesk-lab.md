Help Desk User - LAB
--------------------

In this LAB we will create an administrative user that has the ability to change
passwords on other user accounts.

Create an admin user called `uid=helpdesk,dc=example,dc=com` with a password of
password:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cd $HOME/ds1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldapmodify -p 1189 -D "cn=directory manager" -w password -a <<+
dn: uid=helpdesk,dc=example,dc=com
objectClass: person
objectClass: inetOrgPerson
objectClass: organizationalPerson
objectClass: top
givenName: Help
uid: helpdesk
cn: Help Desk
sn: Desk
userPassword: password
+
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Then add an aci to your directory server to allow access for the new
administrator as follows:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldapmodify -p 1189 -D "cn=directory manager" -w password -a <<+
dn: dc=example,dc=com
changetype: modify
add: aci
aci: (targetattr="userpassword")(version 3.0; 
 acl "Grant admin access for password changes"; 
 allow (all) userdn="ldap:///uid=helpdesk,dc=example,dc=com";)
+
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Verify the additions using ldapsearch. The first command searches for the entry
containing `uid=helpdesk` and returns it if the search is successful. The second
command searches for the base DN and returns ACIs associated with the entry.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldapsearch -p 1189 -D "cn=Directory Manager" -w password \
    -b dc=example,dc=com "(uid=helpdesk)"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldapsearch -p 1189 -D "cn=Directory Manager" -w password \
    -b dc=example,dc=com -s base "(aci=*)" aci
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add the privilege to the admin account as follows:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldapmodify -p 1189 -D "cn=directory manager" -w password -a <<+
dn: uid=helpdesk,dc=example,dc=com
changetype: modify
add: ds-privilege-name
ds-privilege-name: password-reset
+
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now we can test the helpdesk account to make sure it can change passwords:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldappasswordmodify -p 1189 \
-D "uid=helpdesk,dc=example,dc=com" \
-w password \
--authzID dn:uid=user.1,ou=people,dc=example,dc=com \
--newPassword test123
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To ensure this does not work for other users test this with user.1:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldappasswordmodify -p 1189 \
-D "uid=user.1,ou=people,dc=example,dc=com" \
-w test123 \
--authzID dn:uid=user.2,ou=people,dc=example,dc=com \
--newPassword test123
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
