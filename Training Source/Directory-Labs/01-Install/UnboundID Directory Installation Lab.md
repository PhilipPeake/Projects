UnboundID Directory Installation Lab
------------------------------------

Unzip directory distribution and rename to ds1:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ unzip /install/UnboundID-DS-5*.zip
$ mv UnboundID-DS ds1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Repeat for ds2:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ unzip -qq /install/UnboundID-DS-5*.zip
$ mv UnboundID-DS ds2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Change directory to ds1 and run setup:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ cd ds1
$ ./setup --cli --acceptLicense
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   Accept the FQDN of the host.

-   Accept “cn=Directory Manager” as root user.

-   Use password “password”.

-   No SCIM

-   Port: 1189

-   No LDAPS

-   No StartTLS

-   Do not restrict listening to a specific interface.

-   Accept dc=example,dc=com as base DN.

-   Load auto generated data [4]

-   2000 entries.

-   Manual memory configuration [4]

-   512m

-   Auto prime – no

-   Start when setup complete.

-   Set up with above configuration [1]

Set up ds2 using non-interactive mode:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ cd ../ds2
$ ./setup --cli --acceptLicense \
   -n -a -p 1289 -w password --maxHeapSize 512m 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
