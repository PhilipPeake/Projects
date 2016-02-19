Filtered Logging - Lab
----------------------

In this lab we will create a new log publisher for a specific criteria in the
access log otherwise known as Filtered Logging.

Create a set of result criteria that will match all failed authentication
operations and create a custom access log publisher that will log only
operations matching that criteria. Note that this log will not include messages
for connects or disconnects, and only a single message will be logged per
operation which contains both the request and result details. 

The command below will configure a create-result-criteria object to match the
possible failure states of an authentication. Apply this to ds1:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/dsconfig create-result-criteria \
-p 1189 -D "cn=Directory Manager" -w password -n \
--criteria-name "Failed Auth Result Criteria" \
--type simple \
--set result-code-criteria:selected-result-codes \
--set result-code-value:auth-method-not-supported \
--set result-code-value:authorization-denied \
--set result-code-value:confidentiality-required \
--set result-code-value:inappropriate-authentication \
--set result-code-value:insufficient-access-rights \
--set result-code-value:invalid-credentials \
--set result-code-value:strong-auth-required
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, create the log publisher:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/dsconfig create-log-publisher  \
-p 1189 -D "cn=Directory Manager" -w password -n \
--publisher-name "Failed Authentications Logger"  \
--type file-based-access  \
--set enabled:true  \
--set "result-criteria:Failed Auth Result Criteria"  \
--set include-instance-name:true  \
--set include-requester-ip-address:true  \
--set include-requester-dn:true  \
--set log-file:logs/failed-auths \
--set "rotation-policy:24 Hours Time Limit Rotation Policy" \
--set "rotation-policy:Size Limit Rotation Policy"  \
--set "retention-policy:File Count Retention Policy"  \
--set "retention-policy:Free Disk Space Retention Policy"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Perform a search using a valid user, but provide an incorrect password. Note:
all the test users have the password "password" if you want to try with one of
those.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
./bin/ldapsearch -p 1189 \
-D "cn=directory manager" -w bad-password \
-b dc=example,dc=com "objectclass=*"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Look at the failed-auths file in the logs directory. You may want to run a few
more searches with valid passwords to ensure that these don't get logged.
