**Version 2**

# Overview

The Tor relay [ContactInfo](https://www.torproject.org/docs/tor-manual.html.en#ContactInfo) string was primarily
intended to contain an email address and PGP key fingerprint but since this field accepts an arbitrary string
it has been used for multiple other purposes (website urls, donation information, bitcoin addresses, ...).
Making use of provided information in an automated way is hard since there is no specification on how 
this string should look like. This is a specification to formalize the ContactInfo string.
This specification is optional (opt-in), operators can choose to implement it or not.

A simple to use ContactInfo generator for this specification can be found at [https://torcontactinfogenerator.netlify.app/](https://torcontactinfogenerator.netlify.app/)

# Example

An example ContactInfo string as defined by this specification could look like this:

```foo bar email:tor[]example.com url:https://example.com proof:uri-rsa uplinkbw:100 ciissversion:2```

In words this means:
* the technical contact for this relay can be reached at tor@example.com
* the entity responsible for this relay has a website at https://example.com
* the proof file to verify the `url` can be fetched from https://example.com/.well-known/tor-relay/rsa-fingerprint.txt
* this relay has an uplink bandwidth of 100 Mbit/s
* this ContactInfo string implements version 2 of this specification

# Motivation

- increase information sharing among Tor relay operators
- improve the ability to contact relay operators
- increase trust in contact information
  - give operators a standardized way to authenticate some fields
  - detect relay operators impersonating other operators or entities that do not run relays
- collect additional (self-reported) relay metrics
  - for Tor relay operators 
  - for Tor developers
- provide metadata for Tor network graphs
- make hosting costs visible
- make it easier for current and future relay operators to find (rarely used) hosters
  - (indirectly) improve geo- and autonomous system diversity on the Tor network (more diverse is better)
  
## Design Goals

- make provided information machine readable
- allow for automatic verification (limited)
- do not require changes in Tor
- low entry barrier

# Defined Fields 

The fields specified in this document coexists with other arbitrary strings located in 
the relay's ContactInfo descriptor field. Defined fields may appear at any position within 
the contactInfo string. A field identifier (key) MUST only be used once, if it appears multiple times
in the ContactInfo string only the first occurance is considered. 
UTF-8 is supported to the extend that Tor supports it ([proposal 285](https://gitweb.torproject.org/torspec.git/tree/proposals/285-utf-8.txt)).
Punycode encoding should be used for internationalized domain names.

Information is provided in key-value pairs:

```
key ":" value WS key ":" value ...
```

**keys and values MUST NOT contain any whitespace.**
WS is a single or multiple whitespace characters (space, tab, ..).
The order of keys is not mandatory but SHOULD follow the order in which they appear in this specification.
Specifically the email field SHOULD be the first field.

The version field (`ciissversion`) and at least one additional field (any) is mandatory.

## Overview of definied fields

  * **email**
  * **url**
  * **proof**
  * **ciissversion** (mandatory)
  * pgp
  * abuse
  * keybase
  * twitter
  * mastodon
  * matrix
  * xmpp
  * otr3
  * hoster
  * cost
  * uplinkbw
  * trafficacct
  * memory
  * cpu
  * virtualization
  * btc
  * zec
  * xmr
  * donationurl
  * offlinemasterkey
  * signingkeylifetime
  * sandbox
  * os
  * tls
  * aesni
  * autoupdate
  * confmgmt
  * dnslocation
  * dnsqname
  * dnssec
  * dnslocalrootzone 

## contact information

### email
This field contains the email address of the technical contact managing this Tor relay.
The given email address SHOULD be the same for all relays this entity manages.
The value is an addr-spec as defined in [RFC5322](https://tools.ietf.org/html/rfc5322#section-3.4.1) but
the "@" sign SHOULD be replaced with "[]". 
We are aware that this is trivially defeated anti-spam "protection" but 
not all email address scrappers are aware of this specification
(not targeted for Tor contact info data).

example value:

```
contact[]example.com
```

### url
This field contains an URL (or hostname) pointing to the website of the entity (organization or person) responsible for this Tor relay.
In most cases the responsible entity will be the same as the technical contact mentioned in the `email` field.
This field MUST be consistent across all relays where this entity is responsible.
It MUST point to a specific (non-shared) domain/hostname. Two organizations/persons can not have the same field content.
The `url` field is verified using the selected proof method described bellow (`proof` field). 

When displaying the `url` field content on websites and tools implementing this specification:
* The `url` SHOULD be ignored if verification does not succeed.
* End users MUST be able to tell verified from unverified URLs.

In cases where the responsible organization or person does not have a website, this field can be used to specify a DNS domain only. 
In that case "http(s)://" is omitted. 

length: < 400 characters

valid characters: [_%/:a-zA-Z0-9.-]

example value:

```
https://example.com
```

### proof

The `proof` field is only relevant when the `url` field is set. It is ignored when `url` is not set.
The `proof` field gives the operator the option to authenticate the `url` field.

Since the `url` can be set to an arbitrary value - without consent of the entity it points to -
the `proof` field tells interested parties how they can verify the `url` value.
A relay operator can choose one out of two options to establish a proof (proofs can not be combined), they are also the two possible field values:

* uri-rsa
* dns-rsa

The "uri-rsa" method is preferred over "dns-rsa" because it is easier to setup if a webserver
is available and faster when performing proof verfications. The DNS based option SHOULD only be used
when no webserver is available. All relays using a given `url` value MUST have the same consistent `proof` value.
You can not use multiple distinct `proof` values within a single group of relays using a certain `url` value.

Tools performing proof checks SHOULD re-verify the availability of the proof at least every 6 months.

#### uri-rsa

The "uri-rsa" proof method uses the well-known "tor-relay" URI to fetch the RSA SHA1 Tor relay fingerprints
from a fixed location on the `url` domain for verification.

Example: If the `url` points to "https://example.com", the verification process fetches the relay fingerprints from:

https://example.com/.well-known/tor-relay/rsa-fingerprint.txt

The text file contains the RSA SHA1 relay fingerprints from that entity - one per line. 

For bridges the file is named:

https://example.com/.well-known/tor-relay/hashed-bridge-rsa-fingerprint.txt

In case of bridges the file contains [hashed fingerprints](https://metrics.torproject.org/onionoo.html#details_bridge_hashed_fingerprint)
instead of fingerprints.

The path and filename is static and defined in 
[Tor proposal 326](https://gitlab.torproject.org/tpo/core/torspec/-/blob/main/proposals/326-tor-relay-well-known-uri-rfc8615.md).
It is not required that all listed relay fingerprints point to running relays, but all running relays contained in the file
MUST have the same `url` field value.

Note: This URI MUST be accessible via HTTPS regardless whether the `url` uses HTTPS or not. The URI MUST NOT redirect to another domain.

#### dns-rsa

The "dns-rsa" proof method uses DNS instead of HTTPS and places the RSA SHA1 relay fingerprint or the hashed fingerprint in
case of a bridge in DNS TXT records.
DNSSEC MUST be enabled on the domain located in the `url` field to ensure the integrity of DNS records.

When choosing this method (for example because no webserver is available) the operator creates a DNS TXT record for each relay/bridge
to proof it's `url` field.

These DNS TXT records look as follows (example: `url:example.com`):

*relay-fingerprint*.example.com
value:
"we-run-this-tor-relay"

*relay-fingerprint* is the 40 character RSA SHA1 fingerprint of the Tor relay.

For bridges:

*hashed-fingerprint*.example.com
value:
"we-run-this-tor-bridge"

*hashed-fingerprint* is the 40 character SHA1 hash of the bridge's fingerprint.

Each relay/bridge has its own DNS record, a single TXT record MUST be returned per relay/bridge only.

### pgp
40 characters PGP key fingerprint (long form) without leading "0x" and without spaces.
Case in-sensitive. This key relates to the email address given in the `email` field,
but providing the `pgp` field without an `email` field is also possible.

This key SHOULD be available on https://keys.openpgp.org

length: MUST be exactly 40 characters long

valid characters: [a-fA-F0-9]

example value:

```
EF6E286DDA85EA2A4BA7DE684E2C6E8793298290
```

### abuse
Email address of the abuse handling contact for this Tor relay.
This is primariy relevant for Tor exit relays but can also be used on non-exit relays.
The value is an addr-spec as defined in [RFC5322](https://tools.ietf.org/html/rfc5322#section-3.4.1) but
the "@" sign SHOULD be replaced with "[]". 
We are aware that this is trivially defeated anti-spam "protection" but 
not all email address scrappers are aware of this specification
(not targeted for Tor contact info data).

example value:

```
abuse[]example.com
```


### keybase
The technical contact's keybase username. This identifier MUST be usable
to create a valid keybase.io profile url.

length: < 50 characters

valid characters: [a-fA-F0-9]

example value:

```
nusenu
```

### twitter
The entity's twitter username without the leading "@" (non-technical contact). The user MUST be usable
to create a valid twitter profile url. If the responsible organization or person has no twitter account, the technical contact's twitter handle can be used
instead.

length: MUST be 1-15 characters long

valid characters: [a-zA-Z0-9_]

 example value: 
 
```
torproject
```

### mastodon
url pointing to the entity's mastodon profile (responsible organization/person).

length: < 254 characters

 example value:
 
 ```
 https://mastodon.social/@nusenu
 ```
 
### matrix

[Matrix](https://matrix.org/) [user identifier](https://matrix.org/docs/spec/appendices#user-identifiers)
for the technical contact for this Tor relay.

example value:

```
@user:example.com
```


### xmpp
[XMPP](https://en.wikipedia.org/wiki/XMPP) handle for the technical contact of this Tor relay. The "@" sign SHOULD be replaced with "[]".

length: < 254 characters

example value:

```
user[]example.com
```

### otr3
OTR version 3 key fingerprint without spaces.
Case in-sensitive. This key fingerprint relates to the xmpp address given in the `xmpp` field.

length: MUST be exactly 40 characters long

valid characters: [a-fA-F0-9]

example value:

```
EF6E286DDA85EA2A4BA7DE684E2C6E8793298290
```

## hoster information

### hoster
Commercial hoster domain where this server has been ordered. This is supposed to help 
other relay operators and future relay operators to find hosting providers. 
To normalize the provided domain:
- The domain MUST NOT include a protocol specifier (like "https://").
- The provided domain MUST NOT redirect to another (sub)domain
- If the hoster has multiple domains (using different TLDs) use the international version.
- The domain MUST NOT include trailing slashes "/".

If you are your own ISP (and are not offering a commercial service for others) this field MUST be omitted.

length: < 254 characters

valid characters: [a-zA-Z0-9.-]

example:

```
www.example-hoster.com
```

### cost
Monthly hosting costs the hosting company is charging for the server. This does not include time spend to manage the relay. 
The amount MUST be provided with two digits after the decimal separator. The decimal separator MUST be a full stop (not a comma). 
The value MUST be followed by the currency in [ISO4217 format](https://en.wikipedia.org/wiki/ISO_4217#Active_codes).

On servers with multiple Tor instances the server hosting costs given in this field **MUST** be divided through the number of Tor relay instances running on that OS.

length: < 13 characters

valid characters: [A-Z0-9.]

example:
 
```
10.70USD
10.00EUR
```
**invalid** examples:
```
1USD
1.1USD
1,10USD
```


### uplinkbw
Logical network interface speed in Mbit/s (1Mbit/s = 1 000 000 Bit/s) or the value of RelayBandwidthRate in your torrc setting (whatever is smaller). For asymetrical uplinks specify the lower of up- and download bandwidth.

On a server with multiple Tor instances the total available bandwidth of the server **MUST** be divided by the number of Tor relay instances running on it. This is an integer value.

length: < 7 characters

valid characters: [0-9]

 example: 
 
 ```
 100
 ```

### trafficacct
States if this is an unmetered or metered offering. In case of metered bandwidth the monthly included outbound (TX) traffic in GiB ([GibiByte](https://en.wikipedia.org/wiki/Gibibyte)) MUST be provided. If no traffic is included in the monthly costs, this value MUST be set to 0. If the hoster meters in+outbound the hoster provided value must be divided by two. This is an integer value.

On a server with multiple Tor instances the total available monthly traffic of the server **MUST** be divided by the number of Tor relay instances running on it.

length: < 10 characters

valid characters: [unmetrd0-9]

example values:

```
unmetered
0
25600
```


### memory 
Non-persistent memory (RAM) available on this server - measured in MB ([Mebibytes](https://en.wikipedia.org/wiki/Mebibyte)). This is the output of `free -m` on most Unix-based systems.

On a server with multiple Tor instances the memory size **MUST** be divided by the number of Tor relay instances.
This is an integer value.

length: < 10 characters

valid characters: [0-9]

example:

```
4096
```

### cpu

Only relevant for relays running on bare metal.
String without spaces describing the used CPU model.

example:

```
intel-i5-8400
```

### virtualization
States the underlying virtualization technology used on which the OS is running. 
Use "baremetal" for bare-metal servers (not virtualized).

length: < 15 characters

valid characters: [a-z-]

possible values:

```
kvm
qemu
bochs
xen
vmware
virtualbox
hyper-v
lxc
bhyve
openvz
parallels
vmm      #https://man.openbsd.org/vmm
zvm
```


## donation information

### donationurl
url pointing to a website that contains donation information to support this Tor relay.
This MUST be an HTTPS URL.

length: < 254 characters

example:
```
https://torservers.net/donate.html
```

### btc
Bitcoin or [OpenAlias](https://openalias.org/) address where people can send donations to support the operation of this Tor relay.

length: < 100 characters

valid characters: [A-Za-z0-9]


### zec
Zcash address where people can send donations to support the operation of this Tor relay.

length: < 96 characters

valid characters: [A-Za-z0-9]

### xmr
Monero or [OpenAlias](https://openalias.org/) address where people can send donations to support the operation of this Tor relay.

length: < 100 characters

## Tor configuration

### offlinemasterkey
Single character stating whether the [OfflineMasterKey](https://www.torproject.org/docs/tor-manual.html.en#OfflineMasterKey) feature is enabled ("y") on this Tor instance or not ("n").

length: 1 character

valid characters: [yn]

### signingkeylifetime
Integer stating the [signing key renewal interval](https://www.torproject.org/docs/tor-manual.html.en#SigningKeyLifetime) in days.

length: < 6 characters

valid characters: [0-9]


### sandbox
Single character stating whether this instance runs with [Sandbox](https://www.torproject.org/docs/tor-manual.html.en#Sandbox) enabled ("y") or not ("n").

length: 1 character

valid characters: [yn]

## OS Information

### os
String stating which OS distribution and version is used. Distribution and version is separated with a "/" sign.
On platforms where the file [/etc/os-release](https://www.freedesktop.org/software/systemd/man/os-release.html) is available os is created by taking the `ID` and `VERSION_ID` values. The version identifier is optional and may be omitted.
The string is case-insensitive.

length: < 21 character

valid characters: [A-Za-z0-9/.]

 examples:
``` 
OpenBSD/6.7
FreeBSD/13
ubuntu/20.04
debian/10
centos/8
arch
```	

### tls
String stating which tls library is used. 

length: < 15 character

valid characters: [a-z]

example values: 

```
openssl
libressl
```
		
### aesni
Character stating whether AES-NI is available and used ("y") or not available/not used ("n").
 
length: 1 character

valid characters: [yn]
 
### autoupdate
Single character stating whether automatic (unattended) updates are enabled ("y") or not ("n").

length: 1 character

valid characters: [yn]

### confmgmt
States what configuration managment system is used. 
Set to "manual" for no configuration management.


example values

```
ansible
chef
puppet
salt
```

length: < 16 character

valid characters: [a-z]

## DNS Configuration (Exits only)

### dnslocation

This field is only relevant for exit relays, it is ignored on relays that do not allow exiting.

String describing the location of the used DNS resolver in relation to the exit relay.

* **local** means the resolver is running on the same host as the Tor process.
* **sameas** means the resolver is running on the same autonomous system as the exit relay and queries to the resolver do not cross another AS before reaching the resolver.
* **remote** means the resolver is running on a system outside the exit relay's autonomous system

Multiple options may apply and MUST be separated using a comma. The order is relevant, the primary
resolver MUST appear first followed by fallback resolvers.

example values: 

```
local
sameas
remote
local,sameas
```

valid characters: [a-z,]

### dnsqname

This field is only relevant for exit relays, it is ignored on relays that do not allow exiting.

Character stating whether this exit relay is using a resolver that is performing DNS QNAME minimization ("y") or not ("n").
QNAME minimization is defined in [RFC7816](https://datatracker.ietf.org/doc/rfc7816/).

length: 1 character

valid characters: [yn]

### dnssec

This field is only relevant for exit relays, it is ignored on relays that do not allow exiting.

Character stating whether this exit relay is using a resolver that is performing DNSSEC validation ("y") or not ("n").

length: 1 character

valid characters: [yn]

### dnslocalrootzone

This field is only relevant for exit relays, it is ignored on relays that do not allow exiting.

Character stating whether this exit relay is using a resolver that is running the DNS root zone on loopback ("y") 
to avoid latency and information disclosure to DNS root servers or not ("n").
This is defined in [RFC7706](https://tools.ietf.org/html/rfc7706).

length: 1 character

valid characters: [yn]

## Specification Version Information

### ciissversion

This field is mandatory.

Version of the **C**ontact**I**nfo **I**nformation **S**haring **S**pecification (this document) used to generate the ContactInfo string.

format: integer counter

valid characters: [0-9]

length: <4 digits

example values:

```
1
2
```


# Considerations

-  increases exposure of relay operators 

The machine readable information could be used by spammers and to target
relay operators. The amount of spam observed has been limited.
Operators concerned about spam should create a dedicated email address for the purpose of using it
in the ContactInfo string. This email address can be rotated should there be any 
unacceptable amount of spam.

- more information makes targeted exploitation easier / more silent

Attackers could use the additional information provided in these fields to specifically
target only vulnerable systems. Operators concerned about sharing configuration information 
should omit this type of information, but can share other information.

- increased descriptor size and directory traffic

The contactinfo field size could potentially grow because of this specification.
This is mitigated by directory data compression and diffs available since Tor version 0.3.1.

- ContactInfo size constraints

According to the [manual](https://www.torproject.org/docs/tor-manual.html.en#ContactInfo) there is no explicit 
ContactInfo size limit but there is a descriptor size limit. 
The [max. descriptor size](https://gitweb.torproject.org/torspec.git/tree/dir-spec.txt#n364) is 20000 bytes. 
The family size (number of listed fingerprints) and exit policy are two other relevant inputs.


## HTTPS URLs and used certificate authority

The HTTPS endpoints located in field values MUST use certificates from a well known trusted certificate authority (for example [Let's Encrypt](https://letsencrypt.org/)).
