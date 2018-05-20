
# DRAFT


# Overview

Tor's [ContactInfo](https://www.torproject.org/docs/tor-manual.html.en#ContactInfo) 
[descriptor field](https://gitweb.torproject.org/torspec.git/tree/dir-spec.txt#n608) was primarily
intended to contain an email address and PGP key fingerprint but since this field accepts an arbitrary string
it has been used for multiple other purposes (website urls, donation information, bitcoin addresses, ...).
Making use of provided information in an automated way is hard since there is no specification on how 
this string should look like. This is an specification to formalize the ContactInfo string.


# Motivation

- increase the information sharing among tor relay operators
- make it easier for current and future relay operators to find (rarely used) hosters for tor relays
by increasing the information sharing between relay operators. 	-> help the tor network grow
- (indirectly) improve geo- and autonomous system diversity on the tor network (more diverse is better)
- collect additional (self-reported) relay metrics (for things like [Relay Search](https://metrics.torproject.org/rs.html) and [OrNetStats](https://nusenu.github.io/OrNetStats))
- examples: How many use tor's Sandbox/OfflineMasterMode/KIST feature?
- This data could provide tor developers with information on how well tested/how much used new features (like Sandboxes) are before changing defaults. 
- improve the ability to contact relay operators (automatically)
- make provided information machine readable 
- provide the foundation for an automated contactInfo verification bot. 
- Mutually verified email addresses (and other contact options) could be displayed differently on [Relay Search](https://metrics.torproject.org/rs.html)
- make tor t-shirt delivery easier since contact information is potentially already verified
- increase the ability to detect undeclared relay groups / make hiding relay groups harder
- make hosting costs visible
- potentially detect relay operators impersonating other operators by using their email address


# Considerations

-  increases exposure of relay operators 

The machine readable information could be used by spammers and to target
relay operators. 
Operators concerned about this can omit contact information
but it should not be hard to create a new email address for the purpose .

- more information makes targeted exploitation easier / more silent

Attackers could use the additional information provided in these fields to specifically
target only vulnerable systems. Operators concerned about sharing configuration information 
should omit this type of information, but can share other information.

- increased descriptor size and directory traffic

The contactinfo field size could potentially grow because of this specification.
This is expected to be mitigated by directory data compression and diffs available since tor 0.3.1.

- ContactInfo size constraints

According to the [manual](https://www.torproject.org/docs/tor-manual.html.en#ContactInfo) there is no explicit 
ContactInfo size limit but there is a descriptor size limit. 
The [max. descriptor size](https://gitweb.torproject.org/torspec.git/tree/dir-spec.txt#n364) is 20000 bytes. 
The family size (number of listed fingerprints) and exit policy are two other relevant inputs.

# Defined Fields 

The fields specified in this document coexists with other arbibrary strings located in 
the relay's ContactInfo descriptor field. Defined fields may appear at any position within 
the contactInfo string. A field identifier (key) MUST only be used once, if it appears multiple times
in the ContactInfo string only the first occurance is considered. **Non-ASCII characters are not supported.**

Information is provided in key-value pairs:

```key ":" value WS key ":" value ...```

keys and values MUST not contain spaces.
WS is a single or multiple whitespace characters (space, tab, ..).
The order of keys is not mandatory but SHOULD follow the order in which they appear in this specification.
Specifically the email field SHOULD be the first field.

## example
An example contactInfo string as defined by this document could look like this:

```foo bar email:user[]example-operator.com hoster:www.example-hoster.com uplinkbw:100 trafficacct:unmetered cost:10.00USD virtualization:xen```


## contact information
### email
email address where the operator can be reached. 
The given address SHOULD be the same for all relays from a given operator.
The value is an addr-spec as defined in [RFC5322](https://tools.ietf.org/html/rfc5322#section-3.4.1) but
the "@" sign SHOULD be replaced with "[]". 
We are aware that this is trivially defeated anti-spam "protection" but 
not all email address scrappers are aware of this specification
(not targeted for tor contact info data).
International non-ASCII email addresses are NOT supported.

example value:

```user[]example.com```
  
### operatorurl
The website of the operator. This value SHOULD be consistent across all relays of an operator.
This can be a clearnet or .onion url.

length: < 254 characters

valid characters: [a-zA-Z0-9.-]

example value:

```
www.torservers.net
52g5y5karruvc7bz.onion
```

### pgp
40 characters pgp key fingerprint (long form) without leading "0x" and without spaces.
Case in-sensitive.

length: MUST be exactly 40 characters long

valid characters: [a-fA-F0-0]

example value:

```EF6E286DDA85EA2A4BA7DE684E2C6E8793298290```

### keybase
The keybase username identifier. This identifier MUST be usable
to create a valid keybase.io profile url.

length: < 50 characters

valid characters: [a-fA-F0-0]

example value:

```nusenu```

### twitter
twitter identifier without the leading "@". The identifier MUST be usable
to create a valid twitter profile url

length: MUST be 1-15 characters long

valid characters: [a-zA-Z0-9_]

 example value: 
 
```nusenu_```

### mastodon
url pointing to the operators mastodon profile.

length: < 254 characters

 example value:
 
 ```https://mastodon.social/@nusenu```

### xmpp
[XMPP](https://en.wikipedia.org/wiki/XMPP) handle of the operator. The "@" sign SHOULD be replaced with "[]"

length: < 254 characters

example value:

```user[]example.com```

### ricochet
[Ricochet](https://ricochet.im) handle of the operator.

valid characters: [a-z0-9]

length: MUST be 16 characters long

 example value:
 
 ```rs7ce36jsj24ogfw```
 
 ### bitmessage
 [Bitmessage](https://bitmessage.org/) handle of the operator.
 

## hoster information

### hoster
Commercial hoster domain where this server has been ordered. This is supposed to help 
other relay operators and future relay operators to find hosting providers. 
To normalize the provided domain:
- The domain MUST NOT include a protocol specifier (like "https://").
- The provided domain MUST NOT redirect to another (sub)domain
- If the hoster has multiple domains (using different TLDs) use the international version.
- The domain MUST NOT include trailing slashes "/".

If you are your own ISP (and are not offering a commercial service for others) this field SHOULD be omitted.

length: < 254 characters

valid characters: [a-zA-Z0-9.-]

example:

```www.example-hoster.com```

### cost
Monthly hosting costs the hosting company is charging for the server. This does not include time spend to manage the relay. 
The amount MUST be provided with two digits after the decimal separator. The decimal separator MUST be a full stop (not a comma). 
The value MUST be followed by the currency in [ISO4217 format](https://en.wikipedia.org/wiki/ISO_4217#Active_codes).

On servers with multiple tor instances the server hosting costs given in this field **MUST** be divided through the number of tor relay instances running on that OS.

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
Logical network interface speed in MBit/s (1MBit/s = 1 000 000 Bit/s). For asymetrical uplinks specify the lower of up- and download bandwidth.

On a server with multiple tor instances the total available bandwidth of the server **MUST** be divided by the number of tor relay instances. This is an integer value.

length: < 7 characters

valid characters: [0-9]

 example: 
 
 ```100```

### trafficacct
States if this is an unmetered or metered offering. In case of metered bandwidth the monthly included outbound (TX) traffic in GB ([GibiByte](https://en.wikipedia.org/wiki/Gibibyte)) MUST be provided. If no bandwidth is included, this value MUST be set to 0. If the hoster meters in+outbound the hoster provided value must be divided by two. This is an integer value.

On a server with multiple tor instances the total available monthly traffic of the server **MUST** be divided by the number of tor relay instances running on it.

length: < 10 characters

valid characters: [unmetrd0-9]

example values:

```
unmetered
0
25600
```


### memory 
Non-persistent memory (RAM) available on this server - measured in MB ([Mebibytes](https://en.wikipedia.org/wiki/Mebibyte)). (this is the output of 'free -m' on most Unix-based systems.)

On a server with multiple tor instances the memory size **MUST** be divided by the number of tor relay instances.
This is an integer value.

length: < 10 characters

valid characters: [0-9]

example:

```4096```

### cpu

Only relevant for relays running on bare metal.
String without spaces describing the used CPU model.

example:

```
i5-8400
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

### bitcoin
bitcoin address where people are expected to send donations to support the operation of this tor relay.

length: 26-35 characters

valid characters: [A-Za-z0-9]


### zcash
zcash address where people are expected send donations to support the operation of this tor relay.

length: < 96 characters

valid characters: [A-Za-z0-9]

### donationurl
url pointing to a website that contains donation information to support this tor relay.
This SHOULD be an HTTPS url.

length: < 254 characters

example:
```
https://torservers.net/donate.html
```

## tor configuration

### offlinemasterkey
Single character stating whether the [OfflineMasterKey](https://www.torproject.org/docs/tor-manual.html.en#OfflineMasterKey) feature is enabled ("y") on this tor instance or not ("n").

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

### scheduler
Value as configured with the torrc [scheduler](https://www.torproject.org/docs/tor-manual-dev.html.en#Schedulers) option
or "default" for no explicit configuration. This field MUST be omitted on tor releases that do not support this feature (<0.3.2.1-alpha).

length: <50 character

valid characters: [A-Za-z,]

## OS Information

### os
String stating which OS distribution and version is used. Distribution and version is separated with a "/" sign.
On platforms where the file [/etc/os-release](https://www.freedesktop.org/software/systemd/man/os-release.html) is available os is created by taking the `ID` and `VERSION_ID` values. The version identifier is optional and may be omitted.
The string is case-insensitive.

length: < 21 character

valid characters: [A-Za-z0-9/.]

 examples:
``` 
OpenBSD/6.1
HardenedBSD/11
ubuntu/16.04
debian/9
centos/7
fedora/26
arch
```	

### dnslocation

This field is only relevant for exit relays, non-exit relays SHOULD NOT set this field.

String describing the location of the used DNS resolver in relation to the exit relay
This field is only relevant for exit relays.

* **local** means the resolver is running on the same host as the tor process.
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

This field is only relevant for exit relays, non-exit relays SHOULD NOT set this field.

Character stating whether this exit relay is performing DNS QNAME minimization ("y") or not ("n").
QNAME minimization is defined in [RFC7816](https://datatracker.ietf.org/doc/rfc7816/).

length: 1 character

valid characters: [yn]

### dnssec

This field is only relevant for exit relays, non-exit relays SHOULD NOT set this field.

Character stating whether this exit relay is performing DNSSEC validation ("y") or not ("n").

length: 1 character

valid characters: [yn]

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
Single character stating whether automatic (unnattended) updates are enabled ("y") or not ("n").

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
