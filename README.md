# Apache2 with multiple ldap server and multiple groups 


```

# Setting:

#   +----------------------+ 
#   |                      | AD-Controller ldap-1
#   | User: al1, gh7       |
#   |                      |
#   +----------------------+
#      #
#      #
#   +-----------------------------------+ 
#   |                                   | Ad-Controller ldap-ad1
#   | User: WK789, ZU567                |
#   |                                   |
#   | Group Bibl: WK789, ZU567, al1     |
#   | Group EDV : ZU567, gh7,   al1     |     
#   |                                   |
#   +-----------------------------------+
#




<AuthnProviderAlias ldap ldap-1>
        AuthLDAPBindDN "myadlogin"
        AuthLDAPBindPassword s3cr3t
        AuthLDAPURL "ldap://ad.mycompany.test/OU=user,DC=ad,DC=mycompany,DC=test?sAMAccountName"
</AuthnProviderAlias>

<AuthnProviderAlias ldap ldap-ad1>
        AuthLDAPBindDN "adservice@ad1.mycompany.test"
        AuthLDAPBindPassword super_secret
        AuthLDAPURL "ldap://ads3.sub.mycompany.test/OU=Benutzer,DC=sub,DC=mycompany,DC=test?sAMAccountName"
</AuthnProviderAlias>


<AuthzProviderAlias ldap-group ldap-group-bibl "CN=Bibl,OU=Gruppen,DC=sub,DC=mycompany,DC=test">
        AuthLDAPBindDN "adservice@ad1.mycompany.test"
        AuthLDAPBindPassword super_secret
        AuthLDAPURL "ldap://ads3.sub.mycompany.test/OU=Benutzer,DC=sub,DC=mycompany,DC=test?sAMAccountName"
        AuthLDAPGroupAttribute member
        AuthLDAPGroupAttributeIsDN on
        AuthLDAPMaxSubGroupDepth 0
</AuthzProviderAlias>

<AuthzProviderAlias ldap-group ldap-group-edv "CN=EDV,OU=Gruppen,DC=sub,DC=mycompany,DC=test">
        AuthLDAPBindDN "adservice@ad1.mycompany.test"
        AuthLDAPBindPassword super_secret
        AuthLDAPURL "ldap://ads3.sub.mycompany.test/OU=Benutzer,DC=sub,DC=mycompany,DC=test?sAMAccountName"
        AuthLDAPGroupAttribute member
        AuthLDAPGroupAttributeIsDN on
        AuthLDAPMaxSubGroupDepth 0
</AuthzProviderAlias>


listen 0.0.0.0:8080
<VirtualHost 0.0.0.0:8080>
    ServerAdmin root
    ServerName your_servername
   
    LogLevel   error
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    CustomLog ${APACHE_LOG_DIR}/access_8080.log combined
    ErrorLog  ${APACHE_LOG_DIR}/error_8080.log


    DocumentRoot /var/www/html

    <Directory /var/www/html >
        Options FollowSymLinks  Indexes
        AllowOverride None

		    AuthName "Login Kennung"
		    AuthType Basic
		    AuthBasicProvider ldap-ad1 ldap-1	
		    <RequireAny>
				    Require ldap-group-edv
				    Require ldap-group-bibl
		    </RequireAny>
		    <RequireAll>
				    Require valid-user
				    Require ip 147.142.106. 127.0.0.1
		    </RequireAll>
		
    </Directory>

</virtualhost>
