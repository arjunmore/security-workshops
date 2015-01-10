#### Hue config changes needed to make Hue work on a LDAP-enbled, kerborized cluster

-  Edit the kerberos principal to hadoop user mapping to add Hue
Under Ambari > HDFS > Configs > hadoop.security.auth_to_local, add hue entry below. If the other entries do not exist, add them all
```
        RULE:[2:$1@$0]([rn]m@.*)s/.*/yarn/
        RULE:[2:$1@$0](jhs@.*)s/.*/mapred/
        RULE:[2:$1@$0]([nd]n@.*)s/.*/hdfs/
        RULE:[2:$1@$0](hm@.*)s/.*/hbase/
        RULE:[2:$1@$0](rs@.*)s/.*/hbase/
        RULE:[2:$1@$0](hue/sandbox.hortonworks.com@.*HORTONWORKS.COM)s/.*/hue/        
        DEFAULT
```

- allow hive to impersonate users from whichever LDAP groups you choose
```
hadoop.proxyuser.hive.groups = users, hr 
```
- restart HDFS via Ambari

- Edit /etc/hue/conf/hue.ini by uncommenting/changing properties to make it kerberos aware
	- Change all instances of "security_enabled" to true
	- Change all instances of "localhost" to "sandbox.hortonworks.com" 
	- Make below edits to the file:
	```	
	hue_keytab=/etc/security/keytabs/hue.service.keytab
	hue_principal=hue/sandbox.hortonworks.com@HORTONWORKS.COM
	kinit_path=/usr/bin/kinit
	reinit_frequency=3600
	ccache_path=/tmp/hue_krb5_ccache	
	beeswax_server_host=sandbox.hortonworks.com
	beeswax_server_port=8002
	```
	
- restart hue
service hue restart

- confirm Hue works. 
http://sandbox.hortonworks.com:8000     
   
- Logout as hue user and notice that we can not login as LDAP user (e.g. paul/hortonworks)

- Make changes to /etc/hue/conf/hue.ini to set backend to LDAP:
    ```
	backend=desktop.auth.backend.LdapBackend
	pam_service=login
	base_dn="DC=hortonworks,DC=com"
	ldap_url=ldap://ldap.hortonworks.com
	ldap_username_pattern="uid=<username>,cn=users,cn=accounts,dc=hortonworks,dc=com"
	create_users_on_login=true
	user_filter="objectclass=person"
	user_name_attr=uid
	group_filter="objectclass=*"
	group_name_attr=cn
	```
	
- Restart Hue
```
service hue restart
```

- You should now be able to login to Hue on kerborized cluster using an LDAP-defined user:
  - login to Hue as ali/ali or hr1/hr1 and notice that FileBrowser, HCat, Hive now work