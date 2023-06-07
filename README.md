# About
This iRulesLX Application authenticates and authorizes requests from RADIUS/LDAP clients (routers, switches, Linux devices, web servers, etc...) or anything that can authenticate to a RADIUS or LDAP server.

I'm working to consolidate all of the configuration guidence, tips, and troubleshooting to this repository. Keep an eye out for updates and subscribe. 

# Features
- RADIUS authentication and authorization using RADIUS attributes.
- Supports LDAP clients when used with a LDAP/AD server as a pool member.
- Integrates with WebSSH2 on BIG-IP v0.1.3 solution (original use case) but can work with any protocol / service as long as you can pass a password to that service.
- Ephemeral password rotation (experimental). Changes password each time a portal resource is requested, this needs some refinement.
- AD/LDAP Password injection (experimental). Injects ephemeral password into an LDAP/AD environment for resources which can not point their authorization to the BIG-IP LDAP(S) proxy. For example, domain joined AD systems.
- RADIUS auth to local APM database. See https://github.com/billchurch/ephemeral_auth/issues/4 for more details

# Requirements
- APM, iRules LX, and Privleged User Auth licensed and provisioned

## RADIUS Proxy Requirements
- Currently authorization is only available with RADIUS. RADIUS screening requires a working RADIUS server behind the RADIUS VIP. Users to bypass interception (and sent direct to real RADIUS server) may be stored in ephemeral_RADIUS_bypass data-group.

## LDAP(S) Proxy Requirements
- A valid user must exist in LDAP (currently) however authentication requests (bind) will be intercepted. DNs to bypass interception may be stored in ephemeral_LDAP_bypass data-group.

# Operational Modes
There are 3 major modes of operation, and all 3 may work together:

- RADIUS Closed Circuit (no external directory authentication or authorization, meant for tactical or networks with no external connectivity)
- RADIUS w/ Directory Integration (integrates with either local or external directory to authenticate AND authorize users) with option to bypass RADIUS users to RADIUS server specified in pool attached to VIP
- LDAP(S) Screening (requires integration with an existing LDAP deployment, responds only to BIND queries and all other queries are passed to LDAP server pool member(s))

# LDAP Proxy User Feature
The LDAP Proxy User Feature allows user who authenticate with Ephemeral Auth to also run queries to the real LDAP server without being authenticated to that server. Since the bind request is intercepted, the real LDAP server doesn't know the ephemeral auth user is ever authenticated. This allows servers and resources which run queries after the user authenticates, in the context of that user.

See more in [LDAP Proxy User Feature](/config%20details/ldap%20proxy%20user.md)

# CREDGEN feature
v0.2.16 provides a URL entrypoint /credgen to be published on the APM Webtop.

See [Credgen](/config%20details/credgen.md) for full configuration details.

# RADIUS Fallback to APM local user authentication
Starting with ephemeral_auth release 0.2.11, you may authenticate users to the local user database. This will happen **AFTER** an attempt to authenticate with Ephemeral Auth and **ONLY** if the following options are defined in the `ephemeral_config` data group. See detailed information in [RADIUS Fallback to local auth](/config%20details/RADIUS%20fallback%20to%20local%20auth.md)

# RADIUS Attribute Support
Radius attribute support allows for the inclusion for a arbitrary attributes in the response of the Access-Request. See detailed information in [RADIUS Attribute Support](/config%20details/RADIUS%20attribute%20support.md)

# Ephemeral_Config Data Group Options
Details at [Ephemeral_Config Data Group Options](/config%20details/Ephemeral_Config%20Data%20Group%20Options.md)

# General Troubleshooting
## Log files to tail

APM (full session information from webtop to radius/ldap auth)
```tail -f /var/log/apm```

APM (limit to just radius authentication)
```tail -f /var/log/apm | grep -i radius```

APM (limit to LDAP operations)
```tail -f /var/log/apm | grep -i ldap | grep -i proxy```

# PUA Installation and Configuration Video Playlist
https://www.youtube.com/watch?v=y8baLZaY2xE&list=PLz46SWmvZj2QednEE6u_lNYtPIAVKDowO

