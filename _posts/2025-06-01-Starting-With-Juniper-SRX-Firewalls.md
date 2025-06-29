---
title: Starting with Juniper SRX Firewalls
date: 2025-06-01 15:00:00 -0700
categories: [JUNIPER, JUNOS OS, SRX]
tags: [juniper]     # TAG names should always be lowercase
---

![]({{ site.baseurl }}/images/services/juniper_logo.png)

## Packet Mode Processing

The SRX basically operates like a Router.

```
set security forwarding-options family mpls mode packet-based
```

- [[SRX] How to change forwarding mode for IPv4 from 'flow based' to 'packet based'](https://supportportal.juniper.net/s/article/SRX-How-to-change-forwarding-mode-for-IPv4-from-flow-based-to-packet-based?language=en_US)

## Logical Packet Flow

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/logical-packet-flow-diagram.png)

## Branch SRX Series Factory-Default configuration

- Interface `ge-0/0/0` is set for **untrust zone** and to get IP via DHCP
- Default IP address on `fxp0.0`: **192.168.1.1/24**
- Interface `irb.0` is the **trust zone** with address **192.168.2.1/24**
- All other ports are configured as Layer 2

## Security Zones

Interfaces can pass and accept traffic only if assigned to **non-null zone**.

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/zones-01.png)

Types of Zones

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/types-of-zones.png)

Creating a Zone

```
set security zones security-zone untrust interface ge-0/0/0.0
set security zones security-zone trust interface ge-0/0/1.0
set security zones security-zone dmz interface ge-0/0/2.0
set security zones security-zone Server interface ge-0/0/3.0
set security zones security-zone VPN interface st0.0

set security zones functional-zone management interface ge-0/0/3.0
```

Allow SSH and FTP inbound the Security Zone called "HR" and destined to the SRX

```
set security zones security-zone HR host-inbound-traffic system-services ssh
set security zones security-zone HR host-inbound-traffic system-services ftp
```

Show commands

```
show security zones <NAME>
```

Junos-Host Zone CLI Configuration

Allow specific hosts from **SRC_ZONE** to SSH to the SRX

```
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match source-address host1
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match destination-address any
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match application junos-ssh
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match dynamic-application none
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management match url-category none
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-management then permit
```

Block all other SSH attempts from other hosts

```
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match source-address any
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match destination-address any
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match application junos-ssh
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match source-identity any
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match dynamic-application none
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block match url-category none
set security policies from-zone <SRC_ZONE> to-zone junos-host policy ssh-block then deny
```

## Screen Objects

Generate alarms without dropping packets.

```
set security screen ids-option TEST alarm-without-drop
```

Screen Objects are evaluated only on the Ingress Zone.

Screen Types

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/screen-types.png)

Screen Categories

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/screen-categories.png)

Configuring Screen Options

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/screen-config.png)

## Address Objects

- Zone Address Objects

Address objects that are tied to a specific zone
May only be used in security policies where the zone is referred

- Global Address Objects

Define objects in a global address book to avoid duplicate entries for multiple zones
Can be used by all security policies
All objects must be unique

### Creating Address Objects with the CLI

IP address

```
set security address-book PRIVATE address HOST1 192.168.1.35
```

Wildcard address

```
set security address-book PRIVATE address HOST1 wildcard-address 192.168.0.12/255.255.0.255
```

Domain name address

```
set security address-book PRIVATE address HOST1 dns-name www.host1.com
```

Range address

```
set security address-book PRIVATE address HOST1 range-address 192.168.1.100 to 192.168.1.150
```

To configure Global address books

```
set security address-book global
```

```
set security address-book TRUST_ADDRESSES description "Address objects for the trust zone" address HR-PRINTER-05 description "PRINTER IN HR BUILDING C ROOM 05" 192.168.50.45
```

Create and Address Set

```
set security address-book TRUST_ADDRESSES address-set HR-PRINTERS address HR-PRINTER-1
```

Global Address Book attached to a specific Zone

```
set security address-book TRUST_ADDRESSES attach zone TRUST
```

## Service Objects

Display pre-defined Service Security Objects

```
show configuration groups junos-defaults applications
```

Create Custom Applications

```
set applications application MyFTP description "FTP with smaller timer" application-protocol ftp protocol tcp desination-port 21 inactivity-timeout 300
```

Application Sets

```
set applications application-set access application junos-ping
set applications application-set access application junos-ssh
set applications application-set access application junos-https
```

## Configuring Security Zones (LAB)

**NOTE:** The `host-inbound-traffic interface` settings overrides the `host-inbound-traffic zone`.

```
set security zones security-zone UNTRUST interfaces ge-0/0/0.0

delete security zones security-zone TRUST host-inbound-traffic system-services all
set security zones security-zone TRUST host-inbound-traffic system-services ssh
set security zones security-zone TRUST host-inbound-traffic system-services https
set security zones security-zone TRUST interfaces ge-0/0/1.0
set security zones security-zone TRUST interfaces ge-0/0/1.0 host-inbound-traffic system-services telnet

set security zones security-zone DMZ interfaces ge-0/0/2.0
```

## Configuring Screens (LAB)

```
set security screen ids-option DMZ-SCREEN icmp large

set security zones security-zone DMZ screen DMZ-SCREEN
set security zones security-zone DMZ host-inbound-traffic system-services ping
```

```
show security screen statistics zone DMZ
```

## Configuring Global Addresses And Address Sets (LAB)

```
set security address-book global address INTERNET-HOST 172.31.15.1/32

set security address-book DMZ-BOOK address DMZ-NET 10.10.102.0/24
set security address-book DMZ-BOOK address WebServer01 10.10.102.11/32
set security address-book DMZ-BOOK address WebServer02 10.10.102.12/32
set security address-book DMZ-BOOK address WebServer03 10.10.102.13/32
set security address-book DMZ-BOOK attach zone DMZ

set security address-book DMZ-BOOK address-set WebServerSet address WebServer01
set security address-book DMZ-BOOK address-set WebServerSet address WebServer02
set security address-book DMZ-BOOK address-set WebServerSet address WebServer03
```

## Configuring Service Applications and Applications Sets (LAB)

```
set applications application MY-APP protocol tcp destination-port 2020 activity-timeout 300

set applications application-set WebServerAppSet description "Applications for the Web Server"
set applications application-set WebServerAppSet application MY-APP
set applications application-set WebServerAppSet application junos-http
set applications application-set WebServerAppSet application junos-https
```

```
run show configuration groups junos-defaults applications
```

## Security Policies

Security Policies are examined in the following order:

1. Zone policies
2. Global policies
3. Default policy

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/security-policies-and-transit-traffic.png)

Zone security policy example:

```
set security policies from-zone UNTRUST to-zone TRUST policy POLICY-2 match source-address any destination-address any application any
set security policies from-zone UNTRUST to-zone TRUST policy POLICY-2 then deny

set security policies from-zone UNTRUST to-zone TRUST policy RULE-2 match source-address any desination-addresa any application any
set security policies from-zone UNTRUST to-zone TRUST policy RULE-2 then deny
```

Global security policy example:

```
set security policies global policy GLOBAL-1 match source-address any destination-address any application any
set security policies global policy GLOBAL-1 then deny
```

Default security policy example:

Default is deny all traffic.

```
set default-policy permit-all
```

Troubleshoot

```
show security policies
```

## Security Policy Components

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/security-policy_match_criteria.png)

Configuring policy actions example:

```
set security policies from-zone UNTRUST to-zone TRUST policy TEST then permit
set security policies from-zone UNTRUST to-zone TRUST policy TEST then deny
set security policies from-zone UNTRUST to-zone TRUST policy TEST then reject
```

```
set security policies from-zone TRUST to-zone UNTRUST policy RULE-1 match source-address any destination-address any application any
set security policies from-zone TRUST to-zone UNTRUST policy RULE-1 then permit
set security policies from-zone TRUST to-zone UNTRUST policy RULE-1 then log session-init
set security policies from-zone TRUST to-zone UNTRUST policy RULE-1 then log session-close
```

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/working-with-security-rules-cli.png)

```
set security policies from-zone TRUST to-zonee UNTRUST policy POLICY-1 match source-address any destination-address any application any
set security policies from-zone TRUST to-zonee UNTRUST policy POLICY-1 then permit

set security policies from-zone TRUST to-zonee UNTRUST policy RULE-2 match source-address any destination-address any application [junos-ftp junos-ftp-data]
set security policies from-zone TRUST to-zonee UNTRUST policy RULE-2 then deny
```

## Unified Security Policies

Example of unified security policy

```
set security policy match source-address any destination-address any dynamic-application junos:FACEBOOK-ACCESS url-category Enhanced_Social_Web_Facebook
```

AppFW

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/appfw.png)

Unified Security Policy Evaluation

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/unified-security-policy-evaluation.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/visualizing-unified-security-policy-evaluation.png)

Configuring the Pre-ID Default Policy

```
set security policies pre-id-default-policy then session-timeout icmp 4
set security policies pre-id-default-policy then log session-init
```

### Configuring Security Policies

```
edit security address-book global
set address TRUST-NET 10.10.101.0/24
set address DMZ-NET   10.10.102.2/24

top
edit security policies
edit from-zone TRUST to-zone UNTRUST policy TRUST-INTERNET-ACCESS
set match source-address TRUST-NET
set match destination-address any
set match application any
set then permit
set then count

top
edit security policies
edit from-zone TRUST to-zone DMZ policy TRUST-DMZ-ACCESS
set match source-address TRUST-NET
set match destination-address DMZ-NET
set match match application [ junos-ftp junos-ping ]
set then permit

up 1
edit policy TRUST-DMZ-BLOCK
set match source-address TRUST-NET
set match destination-address DMZ-NET
set match application junos-ssh
set then deny
set then log session-init

up 2
edit from-zone UNTRUST to-zone DMZ policy UNTRUST-DMZ-ACCESS
set match source-address any
set match destination-address DMZ-NET
set match application junos-ssh
set then permit
set then log session-init session-close
```

```
show security policies
show security policies from-zone trust to-zone dmz
show log message | match TRUST-DMZ-BLOCK | match RT_Flow
show log message | match UNTRUST-DMZ-ACCESS | match RT_Flow
show security policies policy-name TRUST-INTERNET-ACCESS detail
```

## IPS policies

Part 1: Deploying
    - Install License
    - Download and Install intrustion prevention system (IPS) signature database
    - Optionally, download and install policy templates

Part 2: Applying
    - Configure IPS policy
        - Policy templates can be used to assist with building a new policy
        - Templates can be customized
    - Configure security policy to send traffic for IPS evaluation

### Manually download and update the signature database

```
set security idp security-package url https://services.netscreen.com/cgi-bin/index.cgi
request security idp security-package download full-update
request security idp security-package download status
request security idp security-package install
```

### Automatically download and update the signature database

```
set security idp security-package url https://services.netscreen.com/cgi-bin/index.cgi
set security idp security-package automatic interval 48 start 2021-01-01.23.59.00
set security idp security-package automatic enable
```

### IPS Templates and Policies

```
request security idp security-package download policy-templates
request security idp security-package install policy-templates
set system scripts commit file templates.xls
set security idp active-policy recommended
```

### Referencing IPS Policy

```
set from-zone UNTRUST to-zone SERVER policy IPS-SEC then permit application-services idp-policy IPS_POLICY
```

## Integraded User Firewall

Active Directory Configuration

```
edit services user-identification active-directory-access
set authentication-entry-timeout 5
set wmi-timeout 10
set domain juniper.net user administrator password lab123
set domain juniper.net domain-controller DC1 address 172.16.10.253
set domain juniper.net user-group-mapping ldap base DC=juniper,DC=net
set domain juniper.net ip-user-mapping discovery-method wmi
```

Configure Access Profile

```
set access profile LDAP-UFW authentication-order ldap
set access profile LDAP-UFW authentication-order password
set access profile LDAP-UFW ldap-options base-distinguished-name CN=Users,DC=juniper,DC=net
set access profile LDAP-UFW ldap-options search search-filter sAMAcountName=
set access profile LDAP-UFW ldap-options search admin-search distinguished-name CN=Administrator,CN=Users,DC=juniper,DC=net
set access profile LDAP-UFW ldap-options search admin-search password lab123
```

Security Policy

```
edit security policies from-zone trust to-zone untrust policy test
set match source-address any
set match destination-address any
set match application any
set match source-identity authenticated-user
set then permit firewall-authentication user-firewall access-profile LDAP-UFW
set then permit firewall-authentication user-firewall domain juniper.net
```

## Implementing Security Services (LAB)

### IP Signature Database

```
show security idp security-package-version
```

```
set security idp security-package url <URL>
request security idp security-package download full-update
request security idp security-package download status
request security idp security-package install
request security idp security-package install status

set security idp security-package automatic enable interval 24 start-time 2021-09-15.15:30
```

### IPS Policies

```
request security idp security-package download policy-templates
request security idp security-package download status

request security idp security-package install policy-templates
request security idp security-package install status
```

```
set system scripts commit file templates.xsl

show security idp
copy security idp idp-policy Recommended to idp-policy IJSEC-IPS

edit security idp idp-policy IJSEC-IPS
delete rulebase-ips rule TCP/IP
delete rulebase-ips rule ICMP
delete rulebase-ips rule FTP
delete rulebase-ips rule Malware

top
delete system scripts

edit security policies
set from-zone untrust to-zone dmz policy IPS-FW-Rule match source-address any destination-address any application any
set from-zone untrust to-zone dmz policy IPS-FW-Rule then permit application-services idp-policy IJSEC-IPS
```

### Creating an Active Directory Profile

```
set services user-identification active-directory-access authentication-entry-timeout 30
set services user-identification active-directory-access wmi-timeout 10

set services user-identification active-directory-access domain juniper.net user-group-mapping ldap base DC=juniper,DC=net user administrator password lab123@Lab
set services user-identification active-directory-access domain juniper.net user administrator password lab123@Lab
set services user-identification active-directory-access domain juniper.net domain-controller DC1 address 172.16.1.253
```

```
show services user-identification active-directory-access domain-controller status
show services user-identification active-directory-access statistics ip-user-mapping
```

### Security Policy

```
ediit security policies
set from-zone Trust to-zone Server policy UserFW match source-identity authenticated-user
set from-zone Trust to-zone Server policy UserFW match source-address any
set from-zone Trust to-zone Server policy UserFW match destination-address any
set from-zone Trust to-zone Server policy UserFW match application any
set from-zone Trust to-zone Server policy UserFW then permit
```

```
show services user-identification active-directory-access active-directory-authentication-table all
```

## UTM (Unified Threat Management)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/utm-design-overview.png)

Create Custom Objects

```
set security utm custom-objects url-pattern allow-list value goodcompany.com value partners.com value 90.79.129.7
```

Antispam Default Configuration

```
edit security utm default-configuration
set anti-spam address-whitelist allow-list address-blacklist reject-list type sbl sbl custom-tag-string "***SPAM***" sbl-default-server spam-action tag-subject
```

Antispam Feature Profile

```
edit security utm feature-profile
set anti-spam sbl profile AS-Profile-1 sbl-default-server custom-tag-string "***SPAM***" spam-action block
```

Antispamm Policy

```
edit security utm
set utm-policy "Anti SPAM" anti-spam smtp-profile AS-Profile-1
```

Advanced Security Policy Settings

```
edit security policies
set from-zone Trust to-zone Untrust policy AS-Policy then permit application-services utm-policy "Anti Spam"
```

Monitoring Antispam

```
show security utm anti-spam status
show security utm anti-spam statistics
```

## Antivirus

Creating URL Pattern Allowlist

```
edit security utm custom-objects
set url-pattern Whilelist value *.goodsite.com value *.juniper.net value *.microsoft.com
set custom-url-category URL-Whitelist value Whitelist
```

Creating MIME Allowlist

```
edit security utm custom-objects
set mime-pattern MIME-Whitelist value image/ value video/
```

Avira Antivirus Default Configuration

```
edit security utm
set default-configuration anti-virus type avira-engine
```

Antivirus Feature Profile Configuration

```
edit security utm feature-profile anti-virus
set profile AV-Profile-1 fallback-options default permit
set profile AV-Profile-1 notification-options fallback-block type message no-notify-mail-sender
set profile AV-Profile-1 notification-options fallback-non-block notify-mail-receipient custom-message "Virus Detected"
set profile AV-Profile-1 notification-options virus-detection type message notify-mail-sender custom-message "VIRUS WARNING"
```

Antivirus UTM Policy

```
edit security utm
set utm-policy AV-Policy-1 anti-virus http-profile AV-Profile-1 pop3-profile AV-Profile-1
```

Security Policy Configuration

```
edit security policies
set from-zone Trust to-zone Untrust policy AV-Policy then permit application-services utm-policy AV-Policy-1
```

Monitoring Antivirus

```
show security utm anti-virus status
show security utm anti-virus statistics
```

## UTM: Content Filtering

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/content-filtering-features.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/content-filtering-supported-protocols.png)

Content Filtering Configuration: Custom Objects

```
edit security utm default-configuration
set content-filtering block-extension junos-default-extension
```

Content Filtering Configuration: Default Options

```
set content-filtering block-extension junos-default-extension block-mime list junos-default-bypass-mime
set content-filtering block-content-type zip exe java-applet http-cookie
set content-filtering notification-options type message custom-message "***BLOCKED***"
```

Content Filtering Configuration: Filter Profile

```
edit security utm feature-profile
set content-filtering profile Content-Filter-1
```

Content Filtering Configuration: UTM Policy

```
edit security utm
set feature-profile content-filtering profile Content-Filter-1
```

Content Filtering Configuration: Security Policy Configuration

```
edit security policies
set from-zone Trust to-zone Untrust policy Policy-1 then permit application-services utm-policy Content-Filter-1
```

Content Filtering Verification

```
show security utm content-filtering statistics
```

## UTM: Web Filtering

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/web-filtering-solutions.png)

Web Filtering Configuration: Custom Objects

```
edit security utm
set custom-objects url-pattern Bypass-WF value *.juniper.net
```

Web Filtering Configuration: Default Options

```
edit security utm
set default-configuration url-whitelist WF-Whitelist
```

Web Filtering Configuration: Filter Profile

```
edit security utm
set feature-profile web-filtering juniper-enhanced profile WF-Profile-1 default block custom-block-message "Do not visit this website please"
set feature-profile web-filtering juniper-enhanced profile WF-Profile-1 fallback-settings default log-and-permit
set feature-profile web-filtering juniper-enhanced profile WF-Profile-1 site-reputation-action harmful block
set feature-profile web-filtering juniper-enhanced profile WF-Profile-1 category Enhanced_Gambling action block
```

Web Filtering Configuration: UTM Policy

```
edit security utm
set utm-policy WF-Policy-1 web-filtering http-profile WF-Profile-1
```

Web Filtering Configuration: Security Policy

```
edit security policies
set from-zone Trust to-zone Untrust policy WF-Policy then permit application-services utm-policy WF-Policy-1
```

Web Filtering Verification

```
show security utm web-filtering statistics
show security utm web-filtering status
```

## Working with UTM Anti-Virus (LAB)

```
edit security utm
set default-configuration anti-virus type sophos-engine

top
edit security utm feature-profile anti-virus
set profile av-profile fallback-options engine-not-ready block
set profile av-profile notification-options fallback-block  type protocol-only no-notify-mail-sender custom-message "AV Fallback Block"
set profile av-profile notification-options virus-detection type protocol-only no-notify-mail-sender custom-message "VIRUS WARNING"

top
edit security utm
set utm-policy MyUTMPolicy anti-virus ftp download-profile av-profile
set utm-policy MyUTMPolicy anti-virus ?

top
edit security policies from-zone Untrust to-zone Server
set policy Untrust-Server then permit application-services utm-policy MyUTMPolicy
```

```
show security utm anti-virus status
show security utm anti-virus statistics
```

## Working with UTM Web Filtering (LAB)

```
edit security utm custom-objects
set url-pattern whitelist value http://utmserver.utm.juniper.net/good.html
set url-pattern blacklist value http://utmserver.utm.juniper.net/bad.html
set custom-url-category bad value blacklist
set custom-url-category good value whitelist

top
edit security utm default-configuration web-filtering
set url-whitelist good
set url-blacklist bad
set type juniper-local
set juniper-local custom-block-message "This site is not allowed!"

top
edit security utm feature-profile web-filtering juniper-local
set profile WebFilterPolicy default permit custom-block-message "This site is not allowed!"

top
edit security utm utm-policy MyUTMPolicy
set web-filtering http-profile WebFilterPolicy
```

```
show security utm web-filtering statistics
```

## Source NAT

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-1.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-2.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-3.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-4.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-5.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-6.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-7.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-8.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-9.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-10.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-11.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-12.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-13.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-14.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-15.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-16.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-17.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-18.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Source-NAT-19.png)

## Destination NAT

Main use case is allow a device on the Internet to connect to an internal device behind NAT.

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Destination-NAT-1.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Destination-NAT-2.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Destination-NAT-3.png)

Destination NAT example:

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Destination-NAT-4.png)

```
set security nat destination pool WEB-SERVER-POOL address 10.10.102.10/32
set security nat destination rule-set WEB-SERVER from zone unstrust
set security nat destination rule-set WEB-SERVER rule WEB-SERVER-ACCESS match source-address 0.0.0.0/0
set security nat destination rule-set WEB-SERVER rule WEB-SERVER-ACCESS match destination-address-name EXP-IP
set security nat destination rule-set WEB-SERVER rule WEB-SERVER-ACCESS then destination-nat pool WEB-SERVER-POOL
```

Proxy ARP must be configured to enable the device to respond to ARP queries for the destination NAT IP address.

{% include note.html content="Only required when both the interface IP network and NAT IP are on the same subnet" %}

```
set security nat proxy-arp interface ge-0/0/0 address 172.18.1.1
```

To verify Destination NAT

```
show security flow session
```

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Destination-NAT-5.png)

## Static NAT

Static NAT mapping includes **destination IP address translation** in one direction and **source IP address** translation in the reverse direction.
Static NAT is **bidirectional**.

{% include note.html content="Static NAT always takes precedence over Source or Destination NAT rules" %}

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Static-NAT-1.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Static-NAT-2.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Static-NAT-3.png)

Static NAT example:

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Static-NAT-4.png)

```
set security nat static rule-set RULE-SET1 from zone untrust
set security nat static rule-set RULE-SET1 rule R1 match destination-address 203.0.113.15/32
set security nat static rule-set RULE-SET1 rule R1 then static-nat prefix 10.10.102.10/32
```

Proxy ARP must be configured to enable the device to respond to ARP queries for the destination NAT IP Address.

{% include note.html content="Only required when both the interface IP network and NAT IP are on the same subnet" %}

```
set security nat proxy-arp interface ge-0/0/0.0 address 203.0.113.15
```

To verify Static NAT

```
show security flow session
```

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Static-NAT-5.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Static-NAT-6.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Static-NAT-7.png)

## Troublehsoot NAT

```
show security nat source rule all
show security nat destination rule all
show security nat static rule all
show security flow session protocol tcp
```

## IPsec VPN

Steps to configure IPSec VPN

1. Configuration of Internet Key Exchange (IKE) Phase 1
    - Proposal
    - Policy
    - Gateway
2. Configuration of IKE Phase 2 (IPSec)
    - Proposal
    - Policy
    - VPN tunnel
3. Configuration of the required VPN Routing Parameters
    - Configuring the st0 interface and bind to VPN
    - Configure routes to remote networks with st0 interface as next hop

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IKE-Phase-1-Configuration-Options.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IKE-Phase-2-Configuration-Options-1.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IKE-Phase-2-Configuration-Options-2.png)

IPSec VPN config example

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IPSec-VPN-cofig-example-1.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IPSec-VPN-cofig-example-2.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IPSec-VPN-cofig-example-3.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IPSec-VPN-cofig-example-4.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IPSec-VPN-cofig-example-5.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IPSec-VPN-cofig-example-6.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/IPSec-VPN-cofig-example-7.png)

Configuring Proxy ID (optional)

```
set security ipsec vpn BRANCH-TO-HQ ike proxy-identity local 10.10.101.0/24 remote 10.10.201.0/24
```

Traffic Selector (optional)

```
set security ipsec vpn BRANCH-TO-HQ traffic-selector local-ip 10.10.101.0/24 remote-ip 10.10.201.0/24
```

DPD (Dead Peer Detection) (optional)

```
set security ike gateway HQ dead-peer-detection interval 10 threshold 5
```

VPN Monitoring (optional) - Juniper proprietary method to monitor the VPN's health

```
set security ipsec vpn BRANCH-TO-HQ vpn-monitor destination-ip 10.10.201.10 optimized source-interface st0.0
```

### Troubleshooting IPSec VPN

Tunnel setup

```
show security ike security-associations
show security ipsec security-associations
```

Data Plane traffic statistics

```
show security ipsec statistics
```

Verify the `st` interface

```
show security st0 extensive
show security ipsec traffic-selector interface-name st0.0 detail
```

## Packet Captures

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Packet-Captures-1.png)

```
set forwarding-options packet-capture file filename PACKET-CAPTURE
set forwarding-options packet-capture maximum-capture-size 1500   (optional)

set firewall family inet filter PACKET-CAP term INIT from source-address 172.25.99.1/32
set firewall family inet filter PACKET-CAP term INIT from destination-address 198.51.100.8/32
set firewall family inet filter PACKET-CAP term INIT then sample   (important)
set firewall family inet filter PACKET-CAP term INIT then accept
set firewall family inet filter PACKET-CAP term RETURN from source-address 198.51.100.8/32
set firewall family inet filter PACKET-CAP term RETURN from destination-address 172.25.99.1/32
set firewall family inet filter PACKET-CAP term RETURN then sample   (important)
set firewall family inet filter PACKET-CAP term RETURN then accept
set firewall family inet filter PACKET-CAP term ALLOW-ALL then accept

set interfaces ge-0/0/0 unit 0 family inet filter input PACKET-CAP
set interfaces ge-0/0/0 unit 0 family inet filter output PACKET-CAP
set interfaces ge-0/0/0 unit 0 family inet address 172.25.99.254/24
```

The capture is stored in `/var/tmp/`

```
file list /var/tmp/ | match PACKET-CAPTURE
```

## Packet Captures on High-End SRX

Configure datapath debugging.
Drop to the shell to remove unnecessary fields.

{% include warning.html content="The datapath-debug should not be used in IPv6 environments when using hash-based session distribution" %}

{% include important.html content="Works only on:

SRX1400
SRX3400, SRX3600
SRX5400, SRX5600, SRX5800" %}

```
set security datapath-debug capture-file MY-CAPTURE format pcap size 1m files 5
set security datapath-debug maximum-capture-size 1500

set security datapath-debug action-profile perform-capture event np-egress packet-dump
set security datapath-debug action-profile perform-capture event np-ingress packet-dump

set security datapath-debug packet-filter cap-filter action-profile perform-capture
set security datapath-debug packet-filter cap-filter source-prefix 172.25.99.1/32
set security datapath-debug packet-filter cap-filter destination-prefix 198.51.100.8/32
```

Start the packet capture and stop it when finished

```
request security datapath-debug capture [start | stop]
```

View the packet capture locally

```
show security datapath-debug capture
```

View the packet capture offline. We need to remove some fields:

```
start shell
su

cd /var/log
e2einfo -Ccapture -Snormalize -I MY-CAPTURE -F MY-CAPTURE.pcap
```

Collecting Control Plane traffic

```
start shell
su root

tcpdump -i ge-0/0/0 -s 1500 -w /var/tmp/TO-HOST.pcap -c 5000

ls /var/tmp/ | grep TO-HOST.pcap
```

## Traceoptions

Always stored in `/var/log`

Enabling Tracefiles for Application Identification

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Traceoptions-1.png)

Enabling Tracefiles for IDP

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Traceoptions-2.png)

Enabling Tracefiles for Content Security

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Traceoptions-3.png)

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Traceoptions-4.png)

## Investigating False Positives/Negatives

### Antivirus

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Antivirus-tshoot-1.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Antivirus-tshoot-2.png)

```
show security utm anti-virus statistics
```

### Antispam

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Antispam-tshoot-1.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Antispam-tshoot-2.png)

```
show security utm anti-spam statistics
```

```
test security utm anti-spam ip-check 10.1.1.1
```

### Web Filtering

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/WebFiltering-tshoot-1.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/WebFiltering-tshoot-2.png)

```
show security utm web-filtering statistics
```

```
test security utm enhanced-web-filtering url-check juniper.net

test security utm web-filtering profile WF-pro juniper.net
test security utm web-filtering profile WF-pro example.com
```

### Content Filtering

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/ContentFiltering-tshoot-1.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/ContentFiltering-tshoot-2.png)

```
show security utm content-filtering statistics
```

### Examining Syslog Logging

```
show log message | match RT_UTM
```

## Basic Monitoring

- Resource Utilization

```
show system uptime
show chassis routing-enginge
show chassis hardware
show chassis forwarding
show system storage
show system virtual-memory
file list recursive
```

- Traffic Patterns

```
monitor interface traffic
```

- Chassis components

```
show chassis status
show fan status   (doesn't apply to vSRX)
show chassis fan
show chassis routing-engive
show chassis hardware
show chassis forwarding
```

- Interface status

```
show interfaces terse
show interfaces statistics
show interfaces ge-0/0/0.0
```

- Layer 2 information

```
show ethernet-switching table
```

- Route Table

```
show route
```

- Displaying Junos OS Release

```
show system uptime
show version
show system information
show system firmware
```

- Junos OS Naming Convention

![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Junos-OS-Maintenance-1.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Junos-OS-Maintenance-2.png)
![]({{ site.baseurl }}/images/2025/06-01-Starting-With-Juniper-SRX-Firewalls/Junos-OS-Maintenance-3.png)

- Upgrading Junos OS

```
request system software add /var/tmp/junos-srxsme-20.1r1 no-copy reboot
```

{% include tip.html content="Add the reboot option rather than executing an additional command to reboot the SRX" %}

- Rollback to Previoos Junos OS version

```
request system software rollback reboot
```

- Logs

```
show log messages | match Events
show log interactive-commands
```