# INSTALL 389 DIRECTORY SERVER

## INSTALL COMMAND 
````
yum install 389-ds-base    
````

## CREATE FILE INSTANCE

nano /root/instance.inf

````
# /root/instance.inf
[general]
config_version = 2

[slapd]
root_password = pass1234

[backend-userroot]
sample_entries = yes
suffix = dc=inno,dc=local
````

## CREATE INSTANCE COMMAND

````
dscreate from-file /root/instance.inf
## OR
dscreate -v from-file /root/instance.inf
````


## CHECK STATUS
````
dsctl localhost status
````


## FOR CLIENT

````
nano ~/.dsrc

# cat ~/.dsrc
[localhost]
uri = ldaps://localhost
basedn = dc=inno,dc=local
binddn = cn=Directory Manager
# You need to copy /etc/dirsrv/slapd-localhost/ca.crt to your host for this to work.
# Then run /usr/bin/c_rehash /etc/openldap/certs
tls_cacertdir = /etc/openldap/certs/
````

## FOR SERVER

````
nano ~/.dsrc

# cat ~/.dsrc
[localhost]
# Note that '/' is replaced to '%%2f'.
uri = ldapi://%%2fvar%%2frun%%2fslapd-localhost.socket
basedn = dc=inno,dc=local
binddn = cn=Directory Manager
````


## ADD USER
````
dsidm localhost user create --uid sadmin --cn sadmin --displayName 'sadmin user' \
--uidNumber 1000 --gidNumber 1000 --homeDirectory /home/sadmin


dsidm localhost user create --uid wachira --cn wachira --displayName 'Wachira Duangdee' \
--uidNumber 1001 --gidNumber 1001 --homeDirectory /home/wachira
````


## GET USER sadmin
````
dsidm localhost user get sadmin
````

## SET PASSWORD
````
dsidm localhost account reset_password uid=alice,ou=people,dc=example,dc=com

dsidm localhost account reset_password uid=sadmin,ou=people,dc=inno,dc=local
````

## GROUP CREATE
````
dsidm localhost group create


[root@fedora-ansible ~]# dsidm localhost group create
Enter value for cn : serveradmin
Successfully created serveradmin
[root@fedora-ansible ~]#
````

## ADD MEMBER TO GROUP (add user 'sadmin' to groups 'serveradmin' )

````
dsidm localhost group add_member serveradmin uid=sadmin,ou=people,dc=inno,dc=local

dsidm localhost group add_member serveradmin uid=wachira,ou=people,dc=inno,dc=local
````


https://directory.fedoraproject.org/docs/389ds/howto/quickstart.html


## OPTION
````
/var/run/slapd-localhost.socket

ldapwhoami -H ldaps://localhost -D uid=sadmin,ou=people,dc=inno,dc=local -W -x

(ERROR = ldap_sasl_bind(SIMPLE): Can't contact LDAP server (-1) )

LDAPTLS_CACERT=/etc/dirsrv/slapd-localhost/ca.crt ldapwhoami -H ldaps://localhost -D uid=sadmin,ou=people,dc=inno,dc=local -W -x

To make this permanent, put 'TLS_CACERT /etc/dirsrv/slapd-localhost/ca.crt' into '/etc/openldap/ldap.conf'
````

## ADD DESCRIPTION 'sadmin administrator user'

````
dsidm localhost user modify sadmin "add:description:sadmin administrator user"

dsidm localhost user modify sadmin "replace:description:New Description"
dsidm localhost user modify sadmin "delete:description:New Description"

dsidm localhost user get sadmin
````

## CHECK PLUGIN 'memberof' INSTALL  
````
dsconf localhost plugin memberof status

Enter password for cn=Directory Manager on ldaps://localhost:
Plugin 'MemberOf Plugin' is disabled
````


## ENABLE PLUGIN 
````
dsconf localhost plugin memberof enable

Enter password for cn=Directory Manager on ldaps://localhost:3636:
Enabled plugin 'MemberOf Plugin'
````

## RESTART COMMAND 
````
dsctl localhost restart
Instance "localhost" has been restarted
````

## SET SCOPE PLUGIN
````
dsconf localhost plugin memberof set --scope dc=inno,dc=local

[root@fedora-ansible ~]# dsconf localhost plugin memberof set --scope dc=inno,dc=local
Successfully changed the cn=MemberOf Plugin,cn=plugins,cn=config
[root@fedora-ansible ~]#
````

## MODIFY 
````
dsidm localhost user modify wachira add:objectclass:nsmemberof

dsidm localhost user get wachira

dsconf localhost plugin memberof fixup dc=inno,dc=local
````

````
[root@fedora-ansible ~]# dsconf localhost plugin memberof set --scope dc=inno,dc=local
Successfully changed the cn=MemberOf Plugin,cn=plugins,cn=config
[root@fedora-ansible ~]# ^C
[root@fedora-ansible ~]# dsidm localhost user modify wachira add:objectclass:nsmemberof
Successfully modified uid=wachira,ou=people,dc=inno,dc=local
[root@fedora-ansible ~]# dsidm localhost user get wachira
dn: uid=wachira,ou=people,dc=inno,dc=local
cn: wachira
displayName: wachira User
gidNumber: 1001
homeDirectory: /home/wachira
objectClass: top
objectClass: nsPerson
objectClass: nsAccount
objectClass: nsOrgPerson
objectClass: posixAccount
objectClass: nsmemberof
uid: wachira
uidNumber: 1001
userPassword: {PBKDF2_SHA256}AAAIAKScDbrAn9MDRS2GfGeSJUmg640TglWU5Bya8EQ0jhqQ9C7qhJM1gO/AIKoh0WTtjlx9pEagRoWyG7jSTRFeFdkO2RETx0/YabRTSdD89X5WanDxmyxnoEcOZgJkguiyC2CwDpSkYF+IawICzyL4om      IxAEQMj+8CGnH8RlZHXC/OLp2rJonIq7paN/sRlhVlRH22tGYWKMEJ2wVjbgW+mCOfBsAW5W+JeXVUaUGwFSneFxORf3pF3QZ3zfDItwVV+0R261fBvgW9OIJ9uSMhlgVUDEorcpnRjE/KjtLSntvFd3Dx60BkiqtVSu8Z4e1RUnUQ8zIOBMD1v      hteiWkOhHBtW6SsDxkApPD96Pt/asd92+15kg/HFYCXOjNuQ0m5jtMQyJHsZWMf31Vqof+O3S4Ir1cD7tKDfyzmk8puDe8v
````

````
[root@fedora-ansible ~]# dsconf localhost plugin memberof fixup dc=inno,dc=local
Adding fixup task entry...
Successfully added task entry "cn=memberOf_fixup_2022-09-22T22:11:12.025033,cn=memberOf task,cn=tasks,cn=config". This task is running in the background. To track its progress you can       use the "fixup-status" command.
[root@fedora-ansible ~]# dsidm localhost user get wachira
dn: uid=wachira,ou=people,dc=inno,dc=local
cn: wachira
displayName: wachira User
gidNumber: 1001
homeDirectory: /home/wachira
memberOf: cn=serveradmin,ou=groups,dc=inno,dc=local
objectClass: top
objectClass: nsPerson
objectClass: nsAccount
objectClass: nsOrgPerson
objectClass: posixAccount
objectClass: nsmemberof
uid: wachira
uidNumber: 1001
userPassword: {PBKDF2_SHA256}AAAIAKScDbrAn9MDRS2GfGeSJUmg640TglWU5Bya8EQ0jhqQ9C7qhJM1gO/AIKoh0WTtjlx9pEagRoWyG7jSTRFeFdkO2RETx0/YabRTSdD89X5WanDxmyxnoEcOZgJkguiyC2CwDpSkYF+IawICzyL4om      IxAEQMj+8CGnH8RlZHXC/OLp2rJonIq7paN/sRlhVlRH22tGYWKMEJ2wVjbgW+mCOfBsAW5W+JeXVUaUGwFSneFxORf3pF3QZ3zfDItwVV+0R261fBvgW9OIJ9uSMhlgVUDEorcpnRjE/KjtLSntvFd3Dx60BkiqtVSu8Z4e1RUnUQ8zIOBMD1v      hteiWkOhHBtW6SsDxkApPD96Pt/asd92+15kg/HFYCXOjNuQ0m5jtMQyJHsZWMf31Vqof+O3S4Ir1cD7tKDfyzmk8puDe8v


[root@fedora-ansible ~]#

ldapsearch -x -b "dc=inno,dc=local"

ldapsearch -x -b "ou=people,dc=inno,dc=local"

389-console -a http://192.168.6.12:9830


https://visualstudio.microsoft.com/visual-cpp-build-tools/

dsidm localhost user modify wachira "replace:displayName:Wachira Duangdee"

pip install sparse_dot_topn
````


