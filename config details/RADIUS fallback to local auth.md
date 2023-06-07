# Radius fallback to APM local user authentication
Starting with ephemeral_auth release 0.2.11, you may authenticate users to the local user database. This will happen **AFTER** an attempt to authenticate with Ephemeral Auth and **ONLY** if the following options are defined in the `ephemeral_config` data group.

* **RADIUS_APMAUTH** - _integer_ - _optional_ - Enable authenticating RADIUS users, using an auxiliary APM policy. Requires policy to be specified in _RADIUS_APMAUTH_POLICY_ below.
* **RADIUS_APMAUTH_POLICY** - _string_ - _optional_ - Specify auxiliary APM policy for authenticating RADIUS users.

## Prerequisites and Configuration
1. Local DB Instance Configured and populated with at least one user

   ![image](https://user-images.githubusercontent.com/1668075/37670974-0678217c-2c41-11e8-9606-2090442fb74e.png)
   ![image](https://user-images.githubusercontent.com/1668075/37670985-12d38916-2c41-11e8-94be-4a9843a1ff1e.png)
   ![image](https://user-images.githubusercontent.com/1668075/37670996-1a695f98-2c41-11e8-8f6b-dc829c521662.png)

2. Separate APM Policy referencing the Local DB Instance

   ![image](https://user-images.githubusercontent.com/1668075/37670954-f891520e-2c40-11e8-8025-135567c76ca6.png)
   ![image](https://user-images.githubusercontent.com/1668075/37675031-91daf664-2c4a-11e8-92c2-490833b8d2a7.png)
   ![image](https://user-images.githubusercontent.com/1668075/37672333-1cb4b98e-2c44-11e8-85cc-6274fdc6a5ff.png)
   ![image](https://user-images.githubusercontent.com/1668075/37670924-dfa37538-2c40-11e8-9196-db89959c2327.png)
   ![image](https://user-images.githubusercontent.com/1668075/37670936-e937b67c-2c40-11e8-89bc-d650447027f6.png)
   ![image](https://user-images.githubusercontent.com/1668075/37675074-b29ab254-2c4a-11e8-9da4-014f79219a17.png)

3. Configuration of the ephemeral_config data group with RADIUS_APMAUTH := 1 and RADIUS_APMAUTH_POLICY := _policy_name_ _Note: "/Common/" prefix is required_

   ![image](https://user-images.githubusercontent.com/1668075/37671057-3e856278-2c41-11e8-9ab3-b32a5a772e3a.png)

## Successful Log Event
```
# tail -f /var/log/apm
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: UNKNOWN_PROFILE:Common: RADIUS_PROXY: 192.168.99.81 Valid packet received from 192.168.99.81.
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490530:5: (null):Common:252f6b65: New session from client IP 192.168.99.81 (ST=/CC=/C=) at VIP 192.168.20.230 Listener /Common/radius_vip (Reputation=Unknown) created by iRule /Common/ephemeral_auth_plugin/radius_proxy
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 Attempting authentication to APM policy for user radiususer
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 APM policy Accept user radiususer
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 ATTRIBUTE SUPPORT ENABLED
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 hostGroups for 192.168.99.81 = CN=RouterEnable,CN=Users,DC=mydomain,DC=local:15 CN=RouterView,CN=Users,DC=mydomain,DC=local:1
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 userGroups: enable
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 radiusClientType: DEFAULT
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 radiusClientType: [['Vendor-Specific', 9, [['Cisco-AVPair', 'shell:priv-lvl=<<<VALUE>>>']]]]
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 level:
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 level-now:0
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 Access-Accept for radiususer from 192.168.99.81 
Mar 20 13:44:12 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:252f6b65: RADIUS_PROXY: 192.168.99.81 Access-Accept for "radiususer" from client macbook-pro.billchurch.me to host 192.168.99.81
```

## Failed Log Event (invalid username or password)
```
# tail -f /var/log/apm
Mar 20 13:47:24 pua-13-hf2 notice tmm2[31526]: 01490567:5: UNKNOWN_PROFILE:Common: RADIUS_PROXY: 192.168.99.81 Valid packet received from 192.168.99.81.
Mar 20 13:47:24 pua-13-hf2 notice tmm2[31526]: 01490530:5: (null):Common:e2389cd5: New session from client IP 192.168.99.81 (ST=/CC=/C=) at VIP 192.168.20.230 Listener /Common/radius_vip (Reputation=Unknown) created by iRule /Common/ephemeral_auth_plugin/radius_proxy
Mar 20 13:47:24 pua-13-hf2 notice tmm2[31526]: 01490567:5: /Common/localAuth:Common:e2389cd5: RADIUS_PROXY: 192.168.99.81 Attempting authentication to APM policy for user radiususer
Mar 20 13:47:24 pua-13-hf2 notice tmm2[31526]: 01490567:5: /Common/localAuth:Common:e2389cd5: RADIUS_PROXY: 192.168.99.81 APM policy Deny user radiususer
Mar 20 13:47:24 pua-13-hf2 notice tmm2[31526]: 01490567:5: /Common/localAuth:Common:e2389cd5: RADIUS_PROXY: 192.168.99.81 Access-Reject for radiususer from 192.168.99.81 APM authentication method failed: deny
Mar 20 13:47:24 pua-13-hf2 notice tmm2[31526]: 01490567:5: /Common/localAuth:Common:e2389cd5: RADIUS_PROXY: 192.168.99.81 Access-Reject for "radiususer" from client macbook-pro.billchurch.me to host 192.168.99.81
```

## Failed Log Event (account lockout)

```
# tail -f /var/log/apm
Mar 20 14:24:07 pua-13-hf2 notice tmm3[31526]: 01490567:5: UNKNOWN_PROFILE:Common: RADIUS_PROXY: 192.168.99.81 Valid packet received from 192.168.99.81.
Mar 20 14:24:07 pua-13-hf2 notice tmm3[31526]: 01490530:5: (null):Common:be9d531e: New session from client IP 192.168.99.81 (ST=/CC=/C=) at VIP 192.168.20.230 Listener /Common/radius_vip (Reputation=Unknown) created by iRule /Common/ephemeral_auth_plugin/radius_proxy
Mar 20 14:24:07 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:be9d531e: RADIUS_PROXY: 192.168.99.81 Attempting authentication to APM policy for user radiususer
Mar 20 14:24:07 pua-13-hf2 notice apmd[6059]: 01490000:5: modules/LocaldbStore/LocaldbStoreAgent.cpp func: "localdblib_log_callback()" line: 11 Msg: Login for user radiususer, instance /Common/radius_users rejected - Account locked out.
Mar 20 14:24:07 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:be9d531e: RADIUS_PROXY: 192.168.99.81 APM policy Deny user radiususer
Mar 20 14:24:07 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:be9d531e: RADIUS_PROXY: 192.168.99.81 Access-Reject for radiususer from 192.168.99.81 APM authentication method failed: deny lockout
Mar 20 14:24:07 pua-13-hf2 notice tmm3[31526]: 01490567:5: /Common/localAuth:Common:be9d531e: RADIUS_PROXY: 192.168.99.81 Access-Reject for "radiususer" from client macbook-pro.billchurch.me to host 192.168.99.81
```

# Extending to other authentication methods
While, not documented here. The LocalDB Auth instance in this APM policy may be replaced with **ANY** APM auth process that can take a username and password.