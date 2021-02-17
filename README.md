# ldap

    1  yum -y install openldap compat-openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel
    2  systemctl start slapd
    3  systemctl enable slap
    4  systemctl enable slapd
    5  slappasswd
    6  vim db.ldif
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=field,dc=linuxhostsupport,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldapadm,dc=field,dc=linuxhostsupport,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}JyvKLGw8Kq79IJsbLmacYSA+xdT3DSSD
    7  ldapmodify -Y EXTERNAL -H ldapi:/// -f db.ldif
   10  vim monitor.ldif
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external, cn=auth" read by dn.base="cn=ldapadm,dc=field,dc=linuxhostsupport,dc=com" read by * none
   12  ldapmodify -Y EXTERNAL -H ldapi:/// -f monitor.ldif
openssl req -new -x509 -nodes -out /etc/openldap/certs/myldap.field.linuxhostsupport.com.cert -keyout /etc/openldap/certs/myldap.field.linuxhostsupport.com.key -days 365
chown -R ldap:ldap /etc/openldap/certs
   15  vim certs.ldif
dn: cn=config
changetype: modify
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/myldap.field.linuxhostsupport.com.cert

dn: cn=config
changetype: modify
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/myldap.field.linuxhostsupport.com.key
   16  ldapmodify -Y EXTERNAL -H ldapi:/// -f certs.ldif

   19  slaptest -u
   20  cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
   21  chown -R ldap:ldap /var/lib/ldap
   22  ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
   23  ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
   24  ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
   25  vim base.ldif
dn: dc=field,dc=linuxhostsupport,dc=com
dc: field
objectClass: top
objectClass: domain

dn: cn=ldapadm,dc=field,dc=linuxhostsupport,dc=com
objectClass: organizationalRole
cn: ldapadm
description: LDAP Manager

dn: ou=People,dc=field,dc=linuxhostsupport,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=field,dc=linuxhostsupport,dc=com
objectClass: organizationalUnit
ou: Group
   26  ldapadd -x -W -D "cn=ldapadm,dc=field,dc=linuxhostsupport,dc=com" -f base.ldif
