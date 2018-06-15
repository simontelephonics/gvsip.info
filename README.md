Google Voice SIP Information
============================

On the Google Voice Product Forum, Google representatives [announced that they "will finish migrating the last of \[their\] XMPP interop capabilities for Google Voice to the new \[SIP-based\] Voice platform"](https://productforums.google.com/forum/#!topic/voice/NYRy5U31o98) starting on June 18, 2018. The confusing wording of the statement makes it unclear exactly what will happen on June 18, but one possibility is that the XMPP interop will be shut off at that time.

This site documents how to connect to Google Voice using SIP.

Community Resources
-------------------
* [Alternatives to XMPP (Google Voice Product Forum)](https://productforums.google.com/forum/?utm_medium=email&utm_source=footer#!topic/voice/Psj4Zd-hRpM;context-place=forum/voice)
* [Google Voice XMPP support will go away in June (DSL Reports forum)](https://www.dslreports.com/forum/r31938501-Google-Voice-XMPP-support-will-go-away-in-June) - Findings and speculation on Google Voice's SIP implementation
* [Google Voice Gateway beta test for SIP interop (DSL Reports forum)](https://www.dslreports.com/forum/r31966059-Google-Voice-Gateway-beta-test-for-SIP-interop) - Simon Telephonics updating the Google Voice Gateway to use the new SIP protocol: beta testing and results
* [ObiTalk forum](https://obitalk.com/forum) - Polycom/Obihai users sharing their experience and some useful information about how Obi devices connect to Google Voice using SIP

Technical Details
-----------------
### SIP RFCs
Google Voice uses these standards extending what is commonly implemented in a SIP UAC.

* [RFC 5626](https://tools.ietf.org/html/rfc5626) (SIP Outbound) - the single registrar/UA model, CR/LF keepalives 
* [RFC 3608](https://tools.ietf.org/html/rfc3608) (Service Routes)
* [RFC 3325](https://tools.ietf.org/html/rfc3325), the section on P-Preferred-Identity header, which you get from the P-Associated-URI headers in the REGISTER reply. (The URI here is assigned by Google and is a Base 32-encoded concatenation of the username you register and a 20-digit ID associated with your account. You don't need to encode or decode this; just use it in the P-Preferred-Identity header.)
* [draft-ietf-sipcore-sip-authn-02](https://tools.ietf.org/html/draft-ietf-sipcore-sip-authn-02), Third-Party Authentication for Session Initiation Protocol, section 3: Authentication using the Resource Owner Password Credentials flow

### Media requirements
#### ICE
On the media side, GV requires the client to present a full ICE implementation. 

#### RTCP
Google Voice implements rtcp-mux ([RFC 5761](https://tools.ietf.org/html/rfc5761)).

#### Codecs
* G.711µ
* Opus

### Endpoints

| Proxy                            | Registrar             | Used by          | Notes                  |
|----------------------------------|-----------------------|------------------|------------------------|
| voice.telephony.goog:5061 (TLS)  | voice.sip.google.com  | GV Android app   |                        |
| alt#.voice.telephony.goog:5061   |                       |                  | # = 1..?               |
| obihai.telephony.goog:5061 (TLS) | obihai.sip.google.com | Obihai devices   | Registration generates an "ObiTalk Device" entry on the GV settings page |
| alt#.obihai.telephony.goog:5061  |                       |                  | # = 1..?               |

Example Flow
------------

### Registration & establishing the TLS socket
#### Request
```sip
REGISTER sip:obihai.sip.google.com SIP/2.0
Contact: <sip:me@example.com;transport=tls>;obn=identifier
Expires: 3600
To: <sip:me@obihai.sip.google.com>
Call-ID: ...
Via: SIP/2.0/TLS 10.10.10.10:38250;alias;rport;branch=...
From: <sip:me@obihai.sip.google.com>;tag=...
CSeq: 1 REGISTER
Max-Forwards: 70
Allow: ACK, INVITE, BYE, CANCEL, REGISTER, REFER, OPTIONS, PRACK, INFO
Supported: outbound, path
Authorization: Bearer username="me",realm="obihai.sip.google.com",token="an access token"
Content-Length: 0
```
#### Success Response
```sip
SIP/2.0 200 OK
Via: SIP/2.0/TLS 10.10.10.10:38250;rport=38250;branch=...;received=...;alias
Service-Route: <sip:ENCODED-ROUTE:5060;uri-econt=ENCODED-ROUTE-PART-2;lr>
Service-Route: <sip:SECOND-ENCODED-ROUTE:5060;transport=udp;lr;uri-econt=PART-2>
Require: outbound
Contact: <sip:me@10.10.10.10:38250;transport=tls>;obn=identifier;expires=...
To: <sip:me@obihai.sip.google.com>;tag=...
From: <sip:me@obihai.sip.google.com>;tag=...
Call-ID: ...
CSeq: 1 REGISTER
Allow: ACK, BYE, CANCEL, INFO, INVITE, NOTIFY, OPTIONS, PRACK, REGISTER, SUBSCRIBE, UPDATE
P-Associated-URI: <sip:BASE-32-ENCODED-URI@obihai.sip.google.com>
P-Associated-URI: <sip:me@obihai.sip.google.com>
Content-Length: 0
```

Store the Service-Routes and the encoded P-Associated-URI. These are used later when establishing a dialog. Service-Routes become `Route:` headers (maintain the same order) and the encoded P-Associated-URI is used for the `P-Preferred-Identity:` header.

Outbound calls are placed over the established socket.

### INVITE
#### Request
```sip
INVITE sip:18005551212@obihai.sip.google.com SIP/2.0
Route: <sip:ENCODED-ROUTE:5060;uri-econt=ENCODED-ROUTE-PART-2;lr>
Route: <sip:SECOND-ENCODED-ROUTE:5060;transport=udp;lr;uri-econt=PART-2>
P-Preferred-Identity: <sip:BASE-32-ENCODED-URI@obihai.sip.google.com>
Max-Forwards: 19
Via: SIP/2.0/TLS 10.10.10.10:38250;alias;rport;branch=...
From: <sip:me@example.com>;tag=...
To: <sip:18005551212@obihai.sip.google.com>
Call-ID: ...
CSeq: 2 INVITE
Contact: <sip:me@10.10.10.10:38250;transport=TLS>
Allow: ACK, INVITE, BYE, CANCEL, REGISTER, REFER, OPTIONS, PRACK, INFO
Supported: outbound, path, replaces, 100rel
Content-Type: application/sdp
Content-Length: ...

(sdp)
```

#### Responses
```sip
SIP/2.0 100 Trying
...
```

```sip
SIP/2.0 183 Session Progress
...
(sdp)
```

The 183 response will include an SDP and Google Voice will start sending early media.

There may be a `180 Ringing` response after this. If so, the endpoint should locally play ringback tone to the caller (differs from common practice defined in [RFC 3960](https://tools.ietf.org/html/rfc3960)).

```sip
SIP/2.0 200 OK
...
(sdp)
```

The provisional responses and the 200 OK will contain `Record-Route:` headers which are used for in-dialog routing.

### Incoming call (INVITE)
#### Request
```sip
INVITE sip:me@10.10.10.10:38250;transport=tls SIP/2.0
Via: SIP/2.0/TLS 64.9.243.172:5061;branch=z9hG4bK...;rport
Via: SIP/2.0/UDP ENCODED-ROUTE:5060;branch=z9hG4bK...;econt=PART-2
Via: SIP/2.0/UDP SECOND-ENCODED-ROUTE:5060;branch=z9hG4bK...;econt=PART-2
Max-Forwards: 68
Record-Route: <sip:64.9.243.172:5061;lr;transport=tls>
Record-Route: <sip:ENCODED-ROUTE:5060;lr;transport=udp;uri-econt=PART-2>
Contact: <sip:+18005551212@ENCODED-DOMAIN:5060;transport=udp;uri-econt=PART-2>
To: <sip:BASE-32-ENCODED-URI@obihai.sip.google.com>
From: "caller" <sip:+18005551212@216.239.32.1:5060>;tag=...
Call-ID: ...
CSeq: 18524 INVITE
Allow: ACK, BYE, CANCEL, INVITE, UPDATE
Content-Type: application/sdp
Supported: 100rel
Privacy: none
P-Asserted-Identity: "caller" <sip:+18005551212@216.239.32.1>
P-Called-Party-ID: <sip:BASE-32-ENCODED-URI@obihai.sip.google.com>
Content-Length: 553

v=0
o=- 2125604025 1529079021317 IN IP4 74.125.39.26
s=SIP Call
c=IN IP4 74.125.39.26
t=0 0
a=ice-lite
a=ice-pwd:...
a=ice-ufrag:...
a=group:BUNDLE audio
a=fingerprint:sha-256 16:61:CE:09:B3:82:D2:81:DE:77:DB:B6:62:1C:CB:7E:D0:1B:F3:0B:D4:F7:D2:89:F1:74:35:45:2E:C3:FE:6E
a=setup:actpass
m=audio 19305 RTP/AVP 0 101
a=rtpmap:0 PCMU/8000
a=rtpmap:101 telephone-event/8000
a=rtcp-mux
a=candidate:1 1 UDP 1 74.125.39.26 19305 typ host
a=candidate:2 1 UDP 2 2001:4860:4864:2::26 19305 typ host
a=sendrecv
```

#### Responses
```sip
SIP/2.0 100 Trying
Via: SIP/2.0/TLS 64.9.243.172:5061;branch=z9hG4bK...;rport
Via: SIP/2.0/UDP ENCODED-ROUTE:5060;branch=z9hG4bK...;econt=PART-2
Via: SIP/2.0/UDP SECOND-ENCODED-ROUTE:5060;branch=z9hG4bK...;econt=PART-2
Record-Route: <sip:64.9.243.172:5061;lr;transport=tls>
Record-Route: <sip:ENCODED-ROUTE:5060;lr;transport=udp;uri-econt=PART-2>
To: <sip:BASE-32-ENCODED-URI@obihai.sip.google.com>
From: "caller" <sip:+18005551212@216.239.32.1:5060>;tag=...
Call-ID: ...
CSeq: 18524 INVITE
Server: ...
Content-Length: 0
```

```sip
SIP/2.0 180 Ringing
Via: SIP/2.0/TLS 64.9.243.172:5061;branch=z9hG4bK...;rport
Via: SIP/2.0/UDP ENCODED-ROUTE:5060;branch=z9hG4bK...;econt=PART-2
Via: SIP/2.0/UDP SECOND-ENCODED-ROUTE:5060;branch=z9hG4bK...;econt=PART-2
Record-Route: <sip:64.9.243.172:5061;lr;transport=tls>
Record-Route: <sip:ENCODED-ROUTE:5060;lr;transport=udp;uri-econt=PART-2>
To: <sip:BASE-32-ENCODED-URI@obihai.sip.google.com>
From: "caller" <sip:+18005551212@216.239.32.1:5060>;tag=...
Call-ID: ...
CSeq: 18524 INVITE
Server: ...
Contact: <sip:me@10.10.10.10:38250;transport=TLS>
Allow: ACK, INVITE, BYE, CANCEL, REGISTER, REFER, OPTIONS, PRACK, INFO
Content-Length: 0
```

```sip
SIP/2.0 200 OK
Via: SIP/2.0/TLS 64.9.243.172:5061;branch=z9hG4bK...;rport
Via: SIP/2.0/UDP ENCODED-ROUTE:5060;branch=z9hG4bK...;econt=PART-2
Via: SIP/2.0/UDP SECOND-ENCODED-ROUTE:5060;branch=z9hG4bK...;econt=PART-2
Record-Route: <sip:64.9.243.172:5061;lr;transport=tls>
Record-Route: <sip:ENCODED-ROUTE:5060;lr;transport=udp;uri-econt=PART-2>
To: <sip:BASE-32-ENCODED-URI@obihai.sip.google.com>
From: "caller" <sip:+18005551212@216.239.32.1:5060>;tag=...
Call-ID: ...
CSeq: 18524 INVITE
Server: ...
Contact: <sip:me@10.10.10.10:38250;transport=TLS>
Allow: ACK, INVITE, BYE, CANCEL, REGISTER, REFER, OPTIONS, PRACK, INFO
Content-Length: 424

v=0
o=- 11370442 1 IN IP4 10.10.10.10
s=-
c=IN IP4 10.10.10.10
t=0 0
m=audio 24624 RTP/AVP 0 101
a=rtpmap:0 PCMU/8000
a=rtpmap:101 telephone-event/8000
a=fmtp:101 0-15
a=sendrecv
a=rtcp:24624
a=rtcp-mux
a=ptime:20
a=ice-ufrag:...
a=ice-pwd:...
a=candidate: (private IP)
a=candidate:... 1 UDP 2130706431 PUB.LIC.IP.ADDR 24624 typ host
```

#### Request
```sip
ACK sip:me@10.10.10.10:38250;transport=tls SIP/2.0
...
```

Known Implementations
---------------------
* Google Voice app for Android
* Obihai 200/202/212 (ATA), 1022/1032/1062/2062/2162/2182 (IP Phone)
* [Simon Telephonics Google Voice Gateway](https://simonics.com/gw) - currently in [beta testing for SIP interop](https://www.dslreports.com/forum/r31966059-Google-Voice-Gateway-beta-test-for-SIP-interop)

Contributions/Corrections
-------------------------
[Submit an issue](https://github.com/simontelephonics/gvsip.info/issues/new) for contributions or corrections. 
