# LDAP Proxy User Feature
The LDAP Proxy User Feature allows a user who authenticates with Ephemeral Auth to also run queries to the real LDAP server without being authenticated to that server. Since the bind request is intercepted, the real LDAP server doesn't know the ephemeral auth user is ever authenticated. This allows servers and resources which run queries after the user authenticates, in the context of that user.

IMPORTANT: The service account (adminDN) used for this operation should have the least privleges needed for the queries to be run. DO NOT use an admin or an account which has more privledges than a typical user.

This is configured by copying `./workspace/extensions/ephemeral_auth/config.json.sample` to `./workspace/extensions/ephemeral_auth/config.json` and modifying the appropraite entries

## Config options

- ldapServer
  - URL - _string_ - LDAP url, prefix with ldap:// or ldaps:// if ssl. Port defaults to 389 (ldap) and 636 (ldaps). You may force a port by appending the port as ":_port_". Examples: `ldap://192.168.10.100` , `ldaps://192.168.10.100`, `ldaps://192.168.10.100:1234`
  - adminDN - _string_ - DN of admin user to perform searches on behelf of Anonymous or Ephemeral Auth users. May be full DN or _user@realm_ format
  - adminPassword - _string_ - password for `adminDN`
  - baseDN - _string_ - baseDN for searches, can be left blank (default)
  - tlsOptions
    - rejectUnauthorized - _boolen_ - If ldaps certificate is not trusted, reject connection. Default `false`
    - reconnect - _boolen_ - Reconnect after failure. Default `false`