
# DRAFT
please send your first feedback until 2017-10-27

# Motivation

- make it easier for current and future relay operators to find (rarely used) hosters for tor relays
by increasing the information sharing between relay operators. 	-> help the tor network grow
- improve geo- and autonomous system diversity on the tor network (more diverse is better)
- collect additional (self-reported) relay metrics (for things like atlas [1] and OrNetStats [2])
	exmples: How many use tor's Sandbox/OfflineMasterMode/KIST feature?
	This data could provide tor developers with information on how well tested/how much used new features (like Sandboxes) are before changing defaults. 
- improve the ability to contact relay operators (automatically)
- make provided information machine readable 
- provide the foundation for an automated contactInfo verification bot. Mutually verified email addresses (and other contact options) could be displayed differently on atlas.torproject.org
- make t-shirt delivery easier since contact information is potentially already verified
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
ContactInfo size limit but there is a descriptor size limit. The [max. descriptor size](ttps://gitweb.torproject.org/torspec.git/tree/dir-spec.txt#n364) is 20000 bytes [3]. The 
family size and exit policy are two other relevant inputs.


# Defined Fields 

The fields specified in this document coexists with other arbibrary strings located in 
the relay's ContactInfo descriptor field. Defined fields may appear at any position within 
the contactInfo string.

Information is provided in key-value pairs:

```key ":" value SP key ":" value ...```

keys and values MUST not contain spaces.
The order of keys is not mandatory but SHOULD follow the order in which they appear in this specification.
Specifically the email field SHOULD be the first field.

## examples


```foo bar email:user[]example-operator.com hoster:https://example-hoster.com uplinkbw:100 trafficacct:unmetered cost:10 virtualization:xen```


## contact information
### email
email address where the operator can be reached.
The value is an addr-spec as defined in [RFC5322](https://tools.ietf.org/html/rfc5322#section-3.4.1) but
the "@" sign MUST be replaced with "[]". 
We are aware that this is trivially defeated anti-spam "protection" but 
not all email scrappers are aware of this specification
(not targeted for tor contact info data).
International non-ASCII email addresses are NOT supported.
example value:
	user[]example.com
  
### website
The website of the operator
 example value:
	https://www.torservers.net

### pgp
40 characters pgp key fingerprint (long form) without leading "0x" and without spaces.
Case in-sensitive.
 example value:
	EF6E286DDA85EA2A4BA7DE684E2C6E8793298290

### keybase
The keybase username identifier. This identifier should be usable
to create a valid keybase.io url: https://keybase.io/<id>
 example value:
	nusenu

### twitter
twitter handle without the leading "@"

 example value: 
	nusenu_

### mastodon
mastodon url

 example value:
	https://mastodon.social/@nusenu

### xmpp
xmpp/jabber handle

### ricochet
ricochet handle

 example value:
	rs7ce36jsj24ogfw

## hoster information

### hoster
hoster URL where this server has been ordered. This should help 
other relay operators and future relay operators to find hosting providers.

### cost
Monthly hosting costs this server is causing measured in US dollar. On servers with
multiple tor instances devide the server hosting costs through the number of tor instances.

 example:
	10.7

### uplinkbw
Interface speed in MBit/s. For asymetrical uplinks specify the lower of up- and download bandwidth.

 example: 100   (for 100MBit/s)

### trafficacct
States if this is an unmetered offering or metered TODO

### virtualization
Possible values: 
xen, kvm, bhyve, openvz, vmware, hyper-v, vmm, "none" for bare-metal servers (not virtualized). TODO


## donation information

### bitcoin
bitcoin address where people should send donations to.

### zcash
zcash address where people should send donations to.

## tor configuration

### offlinemasterkey
Character stating whether the [OfflineMasterKey](https://www.torproject.org/docs/tor-manual.html.en#OfflineMasterKey) feature is enabled ("y") on this tor instance or not ("n").

### signingkeylifetime
Integer stating the [signing key renewal interval](https://www.torproject.org/docs/tor-manual.html.en#SigningKeyLifetime) in days.

### sandbox
Character stating whether this instance runs with [sandbox](https://www.torproject.org/docs/tor-manual.html.en#Sandbox) enabled ("y") or not ("n").


### scheduler
Value as configured with the torrc [scheduler](https://www.torproject.org/docs/tor-manual-dev.html.en#Schedulers) option
or "default" for no explicit configuration.

## OS Information

### os
String stating which OS and version is used. OS and version is separated with a "/" sign.

 example:
	OpenBSD/6.1

### ssl
String stating which ssl-library is used. Possible values: openssl, libressl, TODO
		
### aesni
Character stating stating whether AES-NI is available and used ("y") or not available/not used ("n").
 
### autoupdate
Single haracter stating whether automatic (unnattended) updates are enabled ("y") or not. Possible values "y" and "n".

### configurationmgmt
States what configuration managment system is used. Example values: ansible, chef, puppet, salt, manual (for no configuration management) TODO
