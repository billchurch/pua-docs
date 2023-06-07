# Example TLS Configuration for Solaris 10

Simplified Network Diagram
![screenshot 2018-03-20 00 10 46](https://user-images.githubusercontent.com/1668075/37635647-3ad1324c-2bd3-11e8-9cb0-f472a592fbfe.png)

Solaris 10 Configuration:
```
# ldapclient mod -a authenticationMethod=tls:simple -a defaultServerList=ldaps.pua.lab -a preferredServerList=ldaps.pua.lab -a certificatePath=/var/ldap -a followReferrals=false
```

```
# ldapclient list
    NS_LDAP_FILE_VERSION= 2.0
    NS_LDAP_BINDDN= CN=F5 Service Account,CN=Users,DC=mydomain,DC=local
    NS_LDAP_BINDPASSWD= {NS1}41fa88f3a945c411403f13ea
    NS_LDAP_SERVERS= ldaps.pua.lab
    NS_LDAP_SEARCH_BASEDN= DC=mydomain,DC=local
    NS_LDAP_AUTH= tls:simple
    NS_LDAP_SEARCH_REF= FALSE
    NS_LDAP_SERVER_PREF= ldaps.pua.lab
    NS_LDAP_CACHETTL= 0
    NS_LDAP_CREDENTIAL_LEVEL= proxy
    NS_LDAP_SERVICE_SEARCH_DESC= passwd:DC=mydomain,DC=local?sub
    NS_LDAP_SERVICE_SEARCH_DESC= group:DC=mydomain,DC=local?sub
    NS_LDAP_ATTRIBUTEMAP= shadow:userpassword=userPassword
    NS_LDAP_ATTRIBUTEMAP= shadow:shadowflag=shadowFlag
    NS_LDAP_ATTRIBUTEMAP= passwd:loginshell=loginShell
    NS_LDAP_ATTRIBUTEMAP= passwd:homedirectory=unixHomeDirectory
    NS_LDAP_ATTRIBUTEMAP= passwd:uidnumber=uidNumber
    NS_LDAP_ATTRIBUTEMAP= passwd:gidnumber=gidNumber
    NS_LDAP_ATTRIBUTEMAP= passwd:gecos=cn
    NS_LDAP_ATTRIBUTEMAP= group:gidnumber=gidNumber
    NS_LDAP_ATTRIBUTEMAP= group:memberuid=memberUid
    NS_LDAP_ATTRIBUTEMAP= group:userpassword=userPassword
    NS_LDAP_OBJECTCLASSMAP= shadow:shadowAccount=user
    NS_LDAP_OBJECTCLASSMAP= passwd:posixAccount=user
    NS_LDAP_OBJECTCLASSMAP= group:posixGroup=group
    NS_LDAP_HOST_CERTPATH= /var/ldap
```

The DN used in `NS_LDAP_BINDDN` **MUST** be in the ephemeral_ldap_bypass data group for proper operation.

Solaris LDAP Troubleshooting 
https://docs.oracle.com/cd/E19253-01/816-4556/setupproblems-1/index.html

Initializing an LDAP Client
Reference only, don't use "init" on an existing or working system.
https://docs.oracle.com/cd/E23824_01/html/821-1455/clientsetup-49.html#clientsetup-proc-54


# Troubleshooting: 

## Tips
Before changing anything, if the Solaris server has a working configuration do this

1. `ldapclient list`
Save the output of this command! This will display the current configuration, which is essentially /var/ldap/ldap_client_file
**NS_LDAP_BINDDN** is the service account that will need to be entered in 'ephemeral_ldap_bypass' 
2. `ldaplist passwd <userid>` 
This will lookup a userid in the current config, to prove it's working already. If nothing comes back, don't make any changes, figure out what's up
3. `/usr/sfw/bin/certutil -L -d /var/ldap`
This is the certificate database utility. If LDAPS/TLS is already configured on this server, there should be a certificate database in /var/ldap (*.db files). If it's not, you'll need to create it. The database location can be elsewhere and would be revealed in `ldapclient list | grep NS_LDAP_HOST_CERTPATH` if it's specified. If none is specified, /var/ldap is default. I would specify this anyway when time comes. 

## Modifying server to use BIG-IP Ephemeral Auth LDAP Screening Proxy 
Ensure LDAP/LDAPS VIPs are configured with the appropriate pool members and client/serverssl profiles. There should be a valid certificate signed by a CA with the DNS name of the LDAPS Proxy VIP as it's common name.

### LDAP / 389 / unencrypted
If Solaris system is NOT currently using TLS auth (`ldapclient list | grep NS_LDAP_AUTH` returns `NS_LDAP_AUTH= simple`) you can switch the LDAP hostname to the big-ip with this command:

> `ldapclient mod -a defaultServerList=_<BIG-IP VIP/DNS Name>_ -a preferredServerList=_<BIG-IP VIP/DNS Name>_ -a followReferrals=false`

NOTE: `-a followReferrals=false` is highly recommended to prevent the Server from "breaking out" of the screening process.

Once this is complete you can again run this command `ldaplist passwd <userid>` ensure it returns the same value as the previous run. If not, back out the changes and investigate. 

### LDAPS / 636 / encrypted
This is a little more complex. If the current installation is using TLS (`ldapclient list | grep NS_LDAP_AUTH` returns `NS_LDAP_AUTH= tls:simple`) we need to determine where the certificate database for ldapclient is being stored (`ldapclient list | grep NS_LDAP_HOST_CERTPATH`) if this returns empty, see the section below "Creating the certificate database". We then need to `scp` the Base64 CA certificate of the CA which has signed the cert on the LDAPS Proxy virtual server clientssl profile to the Solaris box, and then import that into the database.

1. SCP the CA certificate to the Solaris box to /tmp/my.ca.txt

2. Import the CA to the certificate database located at /var/ldap using this command:

  `/usr/sfw/bin/certutil -A -n "My CA" -i /tmp/my.ca.txt a -t CT -d /var/ldap`

3. List the certificate database to confirm installation

  `/usr/sfw/bin/certutil -L -d /var/ldap/`

4. Verify certificate database is operating properly, and that certificate installed on the LDAPS Proxy vs is being trusted

  `ldapsearch -b DC=mydomain,DC=local -Z -P /var/ldap/cert8.db -h _<BIG-IP VIP/DNS Name>_ -p 636 -D _<proxy DN>_ -w _<proxy password>_ uid=<some valid userid> distingueshedName`

This will force a connection over 636 (TLS) and use the certificate database to validate the certificate.

5. Modify current ldapclient configuration to point to new BIG-IP LDAPS VIP and enable TLS:

  `ldapclient mod -a authenticationMethod=tls:simple -a defaultServerList=_<BIG-IP VIP/DNS Name>_ -a preferredServerList=_<BIG-IP VIP/DNS Name>_ -a certificatePath=/var/ldap -a followReferrals=false`

6. Once this is complete you can again run this command `ldaplist passwd <userid>` ensure it returns the same value as the previous run. If not, back out the changes and investigate. I've found that there can be a delay when running this right after the configuration has changed, you may need to run it a few times before reverting.

#### Creating the certificate database
If this database does not exist, it can be created with the command:

  `/usr/sfw/bin/certutil -N -d /var/ldap/`

## Troubleshooting
Did you disable referrals? You should...

  `ldapclient mod -a followReferrals=false`

## Using TLS?

### Cipher Mismatch
If you're using TLS, it's possible your clientssl profile might be negotiating ciphers too strong for some versions of Solaris. Check /var/log/ltm (on the BIG-IP) for messages such as this:

```
Feb  1 08:12:14 pua-bigip warning tmm[9244]: 01260009:4: Connection error: ssl_select_suite:8181: no shared ciphers (40)
Feb  1 08:12:14 pua-bigip warning tmm[9244]: 01260026:4: No shared ciphers between SSL peers 192.168.99.86.32962:192.168.20.230.636.
Feb  1 08:12:14 pua-bigip warning tmm[9244]: 01260013:4: SSL Handshake failed for TCP 192.168.99.86:32962 -> 192.168.20.230:636
Feb  1 08:12:14 pua-bigip warning tmm1[9244]: 01260009:4: Connection error: ssl_select_suite:8181: no shared ciphers (40)
Feb  1 08:12:14 pua-bigip warning tmm1[9244]: 01260026:4: No shared ciphers between SSL peers 192.168.99.86.32963:192.168.20.230.636.
Feb  1 08:12:14 pua-bigip warning tmm1[9244]: 01260013:4: SSL Handshake failed for TCP 192.168.99.86:32963 -> 192.168.20.230:636
```

You can create a clientssl profile on the ldaps_proxy virtual that matches the tls settings of the Solaris server...

This is an example of the default profiles used by Solaris 10:
![](https://user-images.githubusercontent.com/1668075/52125293-fcceb380-25f9-11e9-9894-dd533a194faa.png)

## Certificate not trusted

## Log Files
- `tail -f /var/log/adm` _(will show any ldap connection errors)_
- `/usr/lib/ldap/ldap_cachemgr -g`
- `tail -f /var/ldap/cachemgr.log` _(usually not too informative)_

## Solaris Reference Site:
https://docs.oracle.com/cd/E23824_01/html/821-1455/clientsetup-66.html#scrolltoc

[Solaris, Active Directory, and SSL](https://docs.oracle.com/cd/E19261-01/820-2761/aarjc/index.html)

[Monitoring Packet Transfers With the snoop Command. Alternative to tcpdump on Solaris](https://docs.oracle.com/cd/E23824_01/html/821-1453/gexkw.html)


## Other tips

when running ldapclient, try prepending "-vvv" to your commands:

`ldapclient -vvv mod -a authenticationMethod=tls:simple -a defaultServerList=ldaps.pua.lab -a preferredServerList=ldaps.pua.lab -a certificatePath=/var/ldap -a followReferrals=false`

The DN used in `NS_LDAP_BINDDN` **MUST** be in the ephemeral_ldap_bypass data group for proper operation.
 