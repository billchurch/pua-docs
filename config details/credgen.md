# Overview

The CREDGEN feature allows you to generate an ephemeral credential to be used without the SSO feature. Some potential use cases for this could be:
- Java based clients like Cisco's ADSM
- Provide `sudo` credentials during an inline session
- Router privilege escalation (`enable` for instance)
- SSH _jump host_ functionality, jumping from one server to another without looping back through the BIG-IP
- Any others?

Normally, when using the BIG-IP APM console, the resource protected by Ephemeral Auth will be signed into automatically and you will not need to know or enter the password for this session. The WebSSH2 feature provides a way to replay credentials, however other applications if asked for the credential have no way to easily provide this password for the user.

CREDGEN satisfies this requirement but allowing a user to generate a credential on-demand for a short life time (_default 60 seconds_) which is configured by the `CREDGEN_TIMEOUT` feature in the `ephemeral_config` data group. The use of this feature is optional and must be enabled by setting the optional `CREDGEN`data group key to 1 in the `ephemeral_config` data group.

# Requirements
- Ephemeral Auth v0.2.16
- APM Webtop
- bootstrap css files (_optional_ provides styling for credgen html)

# Configuration
## Enable the feature
1. Set the `CREDGEN` data group key to `1` in the `ephemeral_config` data group
2. trigger a reload of RULE_INIT for `APM_ephemeral_auth` iRule, typically just putting a comment in this iRule, saving, and selecting `Reload from Workspace...` will satisfy this
3. This will enable the /credgen URL to be called in an APM webtop

## Create Webtop link in APM
1. Access >> Webtops : Webtop Links >> Create
  - Name: `CredGen`
  - Link Type: `Application URI`
  - Application URI: `https://%{session.server.network.name}/credgen`
  - Image: ![key](https://user-images.githubusercontent.com/1668075/48260266-d3323180-e3e8-11e8-9c82-0bff3e5b7800.png)
 
Screenshot:
![image](https://user-images.githubusercontent.com/1668075/48260185-8189a700-e3e8-11e8-9ac1-7392c2542d73.png)

## Upload APM Hosted Content (_optional_)
This provides the styling for the HTML page, not required but makes it MUCH nicer.
1. Download and extract [css-files.zip](https://github.com/billchurch/f5-pua/files/2565670/css-files.zip) locally
2. Navigate to Access >> Webtops : Hosted Content : Manage Files
3. Upload both bootstrap.css and bootstrap-theme.css 
  - Select each file
  - ensure the filename is the same (do not change)
  - File destination folder should be left blank, this will default to `/public/share/`
  - Secure Level should be `session`
4. Once each file is created, edit each one and change the `Mime Tyle` to `Cascading Style Sheets (CSS)` 
5. Navigate to Access >> Webtops : Hosted Content : Manage Access
  - Select the Webtop(s) that contains the CredGen webtop link created above

Screenshots:

![image](https://user-images.githubusercontent.com/1668075/48260691-6324ab00-e3ea-11e8-9f1d-dc5aff47812f.png)
![image](https://user-images.githubusercontent.com/1668075/48260699-6b7ce600-e3ea-11e8-9230-dd79c76dbcf1.png)
![image](https://user-images.githubusercontent.com/1668075/48260712-76377b00-e3ea-11e8-98a2-2a246bd542f7.png)


# Testing
1. Once configured, log in to the Web top that contains the CredGen Web top link.
2. Click the CredGen link
3. A new tab should open and display the user name and ephemeral password of a resource
4. You should be able to click "COPY" on each of these and those items will be added to your clipboard for pasting into another application or web page. **Note:** that you must copy and paste one item at a time. You may also manually type the credentials, however passwords might be using special characters which can prove difficult to easily replicate.
5. In this example the "Fidelis" admin form is purposely NOT using SSO for demo purposes, so it requires manual entry of a user name and password. 

**Note:** it can be helpful to first open the site you want to connect to, then open the GenCred site in a window beside the site you wish to authenticate to to make the copy/paste operation a bit easier.

## Screenshots:

### Webtop:

![image](https://user-images.githubusercontent.com/1668075/48260960-7dab5400-e3eb-11e8-8607-4a2a2bfcc742.png)

### CredGen Site:
![image](https://user-images.githubusercontent.com/1668075/48260928-58b6e100-e3eb-11e8-8681-5dc5508b0a3b.png)

### Fidelis Login Form besides CredGen site:
![image](https://user-images.githubusercontent.com/1668075/48261081-d1b63880-e3eb-11e8-9a72-2e002d0f00a0.png)

