
# DRAFT
please send your first feedback until 2017-10-27

# Motivation

- make it easier for current and future relay operators to find (rarely used) hosters for tor relays
by increasing the information sharing between relay operators. 	-> help the tor network grow
- improve geo- and autonomous system diversity on the tor network (more diverse is better)
- collect additional (self-reported) relay metrics (for things like [atlas](https://atlas.torproject.org) and [OrNetStats](https://nusenu.github.io/OrNetStats))
- exmples: How many use tor's Sandbox/OfflineMasterMode/KIST feature?
- This data could provide tor developers with information on how well tested/how much used new features (like Sandboxes) are before changing defaults. 
- improve the ability to contact relay operators (automatically)
- make provided information machine readable 
- provide the foundation for an automated contactInfo verification bot. 
- Mutually verified email addresses (and other contact options) could be displayed differently on [atlas](https://atlas.torproject.org)
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
This should be mitigated with the use of directory data compression and diffs available since tor 0.3.1.

- ContactInfo size constraints
According to the [manual](https://www.torproject.org/docs/tor-manual.html.en#ContactInfo) there is no explicit 
ContactInfo size limit but there is a descriptor size limit. 
The [max. descriptor size](https://gitweb.torproject.org/torspec.git/tree/dir-spec.txt#n364) is 20000 bytes. 
The family size (number of listed fingerprints) and exit policy are two other relevant inputs.

# Defined Fields 

The fields specified in this document coexists with other arbibrary strings located in 
the relay's ContactInfo descriptor field. Defined fields may appear at any position within 
the contactInfo string. A field identifier (key) MUST only be used once, if it appears multiple times
in the ContactInfo string only the first occurance is considered.

Information is provided in key-value pairs:

```key ":" value SP key ":" value ...```

keys and values MUST not contain spaces.
The order of keys is not mandatory but SHOULD follow the order in which they appear in this specification.
Specifically the email field SHOULD be the first field.

## example
An example contactInfo string as defined by this document could look like this:

```foo bar email:user[]example-operator.com hosterurl:https://example-hoster.com uplinkbw:100 trafficacct:unmetered cost:10 virtualization:xen```


## contact information
### email
email address where the operator can be reached.
The value is an addr-spec as defined in [RFC5322](https://tools.ietf.org/html/rfc5322#section-3.4.1) but
the "@" sign MUST be replaced with "[]". 
We are aware that this is trivially defeated anti-spam "protection" but 
not all email address scrappers are aware of this specification
(not targeted for tor contact info data).
International non-ASCII email addresses are NOT supported.

example value:

```user[]example.com```
  
### operatorurl
The website of the operator

example value:

```https://www.torservers.net```

### pgp
40 characters pgp key fingerprint (long form) without leading "0x" and without spaces.
Case in-sensitive.

example value:

```EF6E286DDA85EA2A4BA7DE684E2C6E8793298290```

### keybase
The keybase username identifier. This identifier MUST be usable
to create a valid keybase.io profile url.

example value:

```nusenu```

### twitter
twitter identifier without the leading "@". The identifier MUST be usable
to create a valid twitter profile url

 example value: 
 
```nusenu_```

### mastodon
url pointing to the operators mastodon profile.

 example value:
 
 ```https://mastodon.social/@nusenu```

### xmpp
xmpp/jabber handle of the operator.

### ricochet
ricochet handle of the operator.

 example value:
 
 ```rs7ce36jsj24ogfw```

## hoster information

### hosterurl
hoster URL where this server has been ordered. This should help 
other relay operators and future relay operators to find hosting providers. The URL MUST NOT include
a protocol specifier (like "https://").
If you are your own ISP (and are not offering a commercial service for others) this field is obsolete.

example:

```www.example-hoster.com```

### cost
Monthly hosting costs the hosting company is charging for the server. This does not include time spend to manage the relay. 
The amount MUST be provided with two digits after the decimal separator. The decimal separator MUST be a full stop (not a comma). 
The value MUST be followed by the currency in [ISO4217 format](https://en.wikipedia.org/wiki/ISO_4217#Active_codes).
On servers with multiple tor instances divide the server hosting costs through the number of tor relay instances running on that OS.

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
On a server with multiple tor instances the total available bandwidth of the server MUST be divided by the number of tor relay instances. This is an integer value.

 example: 
 
 ```100```

### trafficacct
States if this is an unmetered or metered offering. In case of metered bandwidth the monthly included outbound (TX) traffic in TB (**TODO**) is provided. If no bandwidth is included this value MUST be set to 0. If the hoster meters in+outbound the hoster provided value must be divided by two.

example values:

```
unmetered
0
25
```


### memory 
Non-persistent memory (RAM) available on this server - measured in MB ([Mebibytes](https://en.wikipedia.org/wiki/Mebibyte)). (this is the output of 'free -m' on most Unix-based systems.)
On a server with multiple tor instances the memory size MUST be divided by the number of tor relay instances.
This is an integer value.

example:

```4096```

### virtualization
States the underlying virtualization technology used on which the OS is running. 
Use "baremetal" for bare-metal servers (not virtualized).

example values:

```
xen
kvm
bhyve
virtualbox
vmware
hyper-v
vmm
```


## donation information

### bitcoin
bitcoin address where people should send donations to support the operation of this tor relay.

### zcash
zcash address where people should send donations to support the operation of this tor relay.

### donationurl
URL pointing to a website that contains donation information to support this tor relay.
This SHOULD be an https URL.

example:
```
https://torservers.net/donate.html
```

## tor configuration

### offlinemasterkey
Single character stating whether the [OfflineMasterKey](https://www.torproject.org/docs/tor-manual.html.en#OfflineMasterKey) feature is enabled ("y") on this tor instance or not ("n").

### signingkeylifetime
Integer stating the [signing key renewal interval](https://www.torproject.org/docs/tor-manual.html.en#SigningKeyLifetime) in days.

### sandbox
Single character stating whether this instance runs with [Sandbox](https://www.torproject.org/docs/tor-manual.html.en#Sandbox) enabled ("y") or not ("n").


### scheduler
Value as configured with the torrc [scheduler](https://www.torproject.org/docs/tor-manual-dev.html.en#Schedulers) option
or "default" for no explicit configuration.

## OS Information

### os
String stating which OS distribution and version is used. Distribution and version is separated with a "/" sign.
On platforms where the file [/etc/os-release](https://www.freedesktop.org/software/systemd/man/os-release.html) is available os is created by taking the `ID` and `VERSION_ID` values. The version identifier is optional and may be omitted.
The string is case-insensitive.

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

### tls
String stating which tls library is used. 

example values: 

```
openssl
libressl
```
		
### aesni
Character stating stating whether AES-NI is available and used ("y") or not available/not used ("n").
 
### autoupdate
Single character stating whether automatic (unnattended) updates are enabled ("y") or not ("n").

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
