# Ephemeral_Config Data Group Options

Site specific configuration is stored in an internal iRule data-group.

## Config Options

`ephemeral_config` contains several options which may be specified to customize to your needs, vs editing the iRules directly. While there are default settings for each, you should specify any settings you use in the event the default values are changed in a later release. _required_ options denote an option that **MUST** be defined in order to work for a particular feature, however a default value will be created if missing.

**NOTE:** Configuration changes only take effect when `RULE_INIT` is called in `APM_ephemeral_auth`, static:: variables will be updated with the values in `epehermal_config` data-group. If you make any changes here, it's important to trigger a call to RULE_INIT for `APM_ephemeral_auth`, otherwise the changes may not take effect.

* **DEBUG** - _integer_ - _required_ - 0 = disabled, 1 = verbose, 2 = extra verbose
* **DEBUG_PASSWORD** - _integer_ - _optional_ - 0 = disabled (default), 1 = enabled
* **RADIUS_SECRET** - _string_ - _required_ - Shared secret for RADIUS clients. Required for RADIUS proxy functionality. Requied for RADIUS operation. Example `radius_secret` (default)
* **RADIUS_ATTRIBUTE** - _integer_ - _optional_ - Enables/disables role / authorization and generation of vendor specific RADIUS attributes. Requires existence and configuration/existence of ephemeral_radprox_radius_attributes, ephemeral_radprox_radius_client, ephemeral_radprox_host_groups. 0=disable (default), 1=enable
* **RADIUS_TESTMODE** - _integer_ - _optional_ - Enable test mode for testing basic access to RADIUS proxy without configuration of Access Policy or other elements. 0 = disabled (default), 1 = enabled. Required for RADIUS proxy functionality. **DISABLE IN PRODUCTION** for testing only.
* **RADIUS_TESTUSER** - _string_ - _optional_ - Account to be used for above RADIUS proxy testing. Required for RADIUS proxy functionality. Example `f5testuser` (default)
* **LDAP_TESTMODE** - _integer_ - _optional_ - Enable test mode for testing basic access to LDAP proxy without configuration of Access Policy or other elements. 0 = disabled (default), 1 = enabled. Required for LDAP proxy testing functionality. **DISABLE IN PRODUCTION** for testing only.
* **LDAP_TESTUSER** - _string_ - _optional_ - LDAP DN to be used for above LDAP proxy testing. Required for LDAP proxy testing functionality. Example `cn=f5testuser,cn=users,dc=f5test,dc=test` (default)

* **ROTATE** - _integer_ - _optional_ - **EXPERIMENTAL** - Ephemeral password rotation. Destroys/Rotates ephemeral password after each use. 0 = disabled (default), 1 = enabled.
* **pwrulesLen** - _integer_ - _optional_ - Password Complexity, minimum password length/characters. Example `8` (default)
* **pwrulesUpCaseMin** - _integer_ - _optional_ - Password Complexity, minimum upper case characters. Example `1` (default)
* **pwrulesLwrCaseMin** - _integer_ - _optional_ - Password Complexity, minimum lower case characters. Example `1` (default)
* **pwrulesNumbersMin** - _integer_ - _optional_ - Password Complexity, minimum numeric characters. Example `1` (default)
* **pwrulesPunctuationMin** - _integer_ - _optional_ - Password Complexity, minimum punctuation/special characters. Example `1` (default)

* **RADIUS_APMAUTH** - _integer_ - _optional_ - Enable authenticating RADIUS users, using an auxiliary APM policy. Requires policy to be specified in _RADIUS_APMAUTH_POLICY_ below.
* **RADIUS_APMAUTH_POLICY** - _string_ - _optional_ - Specify auxiliary APM policy for authenticating RADIUS users.

* **EPHEMERAL_TABLE_TIMEOUT** - _integer_ - _optional_ - Set the timeout (seconds) for the Ephemeral Authentication table. Example `3600` (default/3600)
* **EPHEMERAL_TABLE_LIFETIME** - _integer_ - _optional_ - Set the timeout (seconds) for the Ephemeral Authentication table. Example `0` (default/0=indefinite)

No longer used:
* ~~**BINDDN** - _string_ - _optional_ - LDAP Distingushed Name used to update AD/LDAP passwords. Required for PASSMOD APM iRule Event. Example `CN=Administrator,CN=Users,DC=mydomain,DC=local`~~
* ~~**BINDPWD** - _string_ - _optional_ - LDAP password for above DN. Required for PASSMOD APM iRule Event. Example `Password`~~
* ~~**BINDURL** - _string_ - _optional_ - LDAP password for above DN. Must be an LDAPS URL. Required for PASSMOD APM iRule Event. Example `ldaps://192.168.20.230:636`~~


![screenshot 2017-12-13 11 35 21](https://user-images.githubusercontent.com/1668075/33950217-de901df6-dff9-11e7-9876-49a7fd2f5ad6.png)

Example Settings
