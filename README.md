Google Voice SIP Information
============================

On the Google Voice Product Forum, Google representatives [announced that Google Voice's XMPP interop 
would be discontinued](https://productforums.google.com/forum/#!topic/voice/NYRy5U31o98) on June 18, 2018.

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

Known Implementations
---------------------
* Google Voice app for Android
* Obihai 200/202/212 (ATA), 1022/1032/1062 (IP Phone)
* [Simon Telephonics Google Voice Gateway](https://simonics.com/gw) - currently in [beta testing for SIP interop](https://www.dslreports.com/forum/r31966059-Google-Voice-Gateway-beta-test-for-SIP-interop)

Contributions/Corrections
-------------------------
[Submit an issue](https://github.com/simontelephonics/gvsip.info/issues/new) for contributions or corrections. 
