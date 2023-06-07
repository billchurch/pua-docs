# Summary
Radius attribute support allows for the inclusion for a arbitrary attributes in the response of the Access-Request.

# Enabling
The feature must first be enabled by configuring the `RADIUS_ATTRIBUTE` variable in the `ephemeral_config` data group to `1`:

It's also helpful to set `EPHEMERAL_DEBUG` to `2` for additional details.

![image](https://user-images.githubusercontent.com/1668075/53433941-82f4d480-39c3-11e9-9660-a075bea1eb75.png)

**NOTE:** be sure to **Add** and **Update** the data group and **Trigger a reload**, see Issue #28.

# Configuring
## Definitions
- Radius Client - the server/router/switch asking the BIG-IP for authentication
- Radius Attribute - An arbitrary list of key/values to be sent back to the Radius Client
- Client Type - Vendor / Radius Config Type

## Data group descriptions
There are 3 data groups used by this feature, all prefaced with `ephemeral_radproxy_`

- ephemeral_radprox_host_groups - A mapping of Radius Clients, to LDAP Groups, to Privilege Levels
- ephemeral_radprox_radius_attributes - A mapping of Radius Attributes to Client Types
- ephemeral_radprox_radius_client - A Mapping of Radius Clients to Client Types


### ephemeral_radprox_host_groups
Maps a RADIUS client to a list of group:privelege level, delimited by semicolons. Most of the work will be done here.

It is expected that the APM session variable: `session.ldap.last.attr.memberOf` is populated with the groups that would be matched to these entries. There should be nothing special you have to do here, just ensure that `session.ldap.last.attr.memberOf` contains a group that you want matched below. 

If using Local DB auth as your source of truth for authentication, the script will also check for `session.localdb.groups`, however using APM Local DB auth is not covered in this issue.

Roughly line 141 of `radius_proxy` cleans these groups up to remove extra spaces and characters. 

#### This would be a single entry

tmsh list ltm data-group internal ephemeral_radprox_host_groups:
`192.168.99.80 { data "CN=RouterEnable,CN=Users,DC=mydomain,DC=local:15" }`

tmui:
![image](https://user-images.githubusercontent.com/1668075/53441517-feab4d00-39d4-11e9-8af0-4dc739cf2e3c.png)

plain string:
string: `192.168.99.80`
value: `CN=RouterEnable,CN=Users,DC=mydomain,DC=local:15`

What this is saying is, for the Radius Client 192.168.99.80:
-  The LDAP group "CN=RouterEnable,CN=Users,DC=mydomain,DC=local" gets privilege level "15" 

While, Cisco typically uses numbers, other vendors may use strings and this is permitted. 

**NOTE:** The characters `;` and `:` are used a delimiters here and are off-limits.

#### Multiple Entries for a single host

tmsh list ltm data-group internal ephemeral_radprox_host_groups:
`192.168.99.80 { data "CN=RouterEnable,CN=Users,DC=mydomain,DC=local:15;CN=RouterView,CN=Users,DC=mydomain,DC=local:1" }`

tmui:
![image](https://user-images.githubusercontent.com/1668075/53441469-eaffe680-39d4-11e9-9fac-7861a49cd338.png)

plain string:
string: `192.168.99.80`
value: `CN=RouterEnable,CN=Users,DC=mydomain,DC=local:15;CN=RouterView,CN=Users,DC=mydomain,DC=local:1`

What this is saying is, for the Radius Client 192.168.99.80:
-  The LDAP group "CN=RouterEnable,CN=Users,DC=mydomain,DC=local" gets privilege level "15"
-  The LDAP group CN=RouterView,CN=Users,DC=mydomain,DC=local" gets privilege level "1"

### ephemeral_radprox_radius_attributes
Maps a RADIUS client type to an extended radius attribute (for authorization). <<<VALUE>>> will be substituted for the value derived from the ephemeral_radprox_host_groups.

#### Example of an entry

tmsh list ltm data-group internal ephemeral_radprox_radius_attributes:
`CISCO { data "[['Vendor-Specific', 9, [['Cisco-AVPair', 'shell:priv-lvl=<<<VALUE>>>']]]]" }`

tmui:
![image](https://user-images.githubusercontent.com/1668075/53441564-1aaeee80-39d5-11e9-82ac-0210c3399325.png)

plain string:
string: `CISCO`
value: `[['Vendor-Specific', 9, [['Cisco-AVPair', 'shell:priv-lvl=<<<VALUE>>>']]]]`

This will be enumerated and sent back to the client as something like this:

`[['Vendor-Specific', 9, [['Cisco-AVPair', 'shell:priv-lvl=15']]]]`


Where 15 is the level derived from the ephemeral_radprox_host_groups datagroup based on the RADIUS client and LDAP group.

You may modify these entries, or create new ones. It's recommended to copy an existing entry instead of overwriting it since these have been well tested. **The name is absolutely arbitrary** and would only need to be **paired up with a RADIUS Client** in `ephemeral_radprox_radius_client` below.

### ephemeral_radprox_radius_client
Maps a RADIUS client to a attribute type listed in ephemeral_radprox_radius_attributes.

#### Example of an entry

tmsh list ltm data-group internal ephemeral_radprox_radius_client:
`192.168.99.82 { data CISCO }`

tmui:
![image](https://user-images.githubusercontent.com/1668075/53441613-30bcaf00-39d5-11e9-8341-11b1b93a3e33.png)

This says that any Access-Requests from the Radius Client 192.168.99.82 will use the CISCO RADIUS attributes as defined in `ephemeral_radprox_radius_attributes`

If not specified, the client will be assigned the attribute specified in DEFAULT in the `ephemeral_radprox_radius_attributes datagroup`.

# Troubleshooting
Log events are generated when `EPHEMERAL_DEBUG` is set to `2`

A successful event looks something like this:
`tail -f /var/log/apm | grep RADIUS_PROXY`
```
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: UNKNOWN_PROFILE:Common: RADIUS_PROXY: 192.168.99.82 Valid packet received from 192.168.99.82.
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 username = joe.user, password = 2Y_.4vGm b64encode(sha256)= pr1xq/K+u8PODwd/3fgtqz2QYAy8bP1z31k125Nbt50=
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 static::EPHEMERAL_RADIUS_ATTRIBUTE: 1
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 ATTRIBUTE SUPPORT ENABLED
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 hostGroups for 192.168.99.82 = CN=RouterEnable,CN=Users,DC=mydomain,DC=local:15 CN=RouterView,CN=Users,DC=mydomain,DC=local:1
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 userGroups: CN=RouterView,CN=Users,DC=mydomain,DC=local CN=RouterEnable,CN=Users,DC=mydomain,DC=local {CN=Domain Users,CN=Users,DC=mydomain,DC=local}
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 radiusClientType: CISCO
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 radiusClientType: [['Vendor-Specific', 9, [['Cisco-AVPair', 'shell:priv-lvl=<<<VALUE>>>']]]]
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 level:1
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 level-now:1
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 level:15
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 level-now:15
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 level:
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 level-now:15
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 radiusClientType: [['Vendor-Specific', 9, [['Cisco-AVPair', 'shell:priv-lvl=15']]]]
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 Access-Accept for joe.user from 192.168.99.82 
Feb 26 13:08:18 pua-bigip notice tmm1[17376]: 01490567:5: /Common/sample_pua_policy:Common:ae1ad063: RADIUS_PROXY: 192.168.99.82 Access-Accept for "joe.user" from client undefined to host 192.168.99.82
```

