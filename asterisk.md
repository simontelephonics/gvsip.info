# Asterisk config example

## pjsip.conf
```
; if using chan_sip for non-gv incoming registrations:
[incoming-registrations]
type=transport
protocol=udp
bind=0.0.0.0:9999
 
; if using chan_pjsip for non-gv incoming registrations:
;[incoming-registrations-unused]
;type=transport
;protocol=udp
;bind=0.0.0.0
 
[transport_tls]
type=transport
protocol=tls
bind=0.0.0.0:5061
 
[gvsipN]
type=transport
protocol=flow
 
[gvsipN]
type=registration
outbound_auth=gvsipN
server_uri=sip:obihai.sip.google.com
outbound_proxy=sip:voice.telephony.goog:5061\;transport=tls\;lr\;hide
client_uri=sip:<your choice for unique user identifier>@obihai.sip.google.com
retry_interval=60
support_path=yes
support_outbound=yes
contact_header_params=obn=<name to appear on GV settings page>
line=yes
endpoint=gvsipN
transport=gvsipN
 
[gvsipN]
type=auth
auth_type=google_oauth
refresh_token=<your oauth refresh token>
oauth_clientid=<your oauth client id>
oauth_secret=<your oauth client secret>
username=<your choice for unique user identifier>
realm=obihai.sip.google.com
 
[gvsipN]
type=aor
contact=sip:obihai.sip.google.com
 
[gvsipN]
type=endpoint
context=from-external
disallow=all
allow=ulaw
allow=opus
outbound_auth=gvsipN
outbound_proxy=sip:voice.telephony.goog:5061\;transport=tls\;lr\;hide
aors=gvsipN
direct_media=no
ice_support=yes
rtcp_mux=yes
media_use_received_transport=yes
transport=gvsipN
```

## rtp.conf
```
[general]
stunaddr=stun.l.google.com:19302
```

## extensions.conf
```
[from-internal]
exten => _+1NXXNXXXXXX,1,Goto(${EXTEN:2},1)
exten => _NXXNXXXXXX,1,Goto(1${EXTEN},1)
exten => _1NXXNXXXXXX,1,Dial(PJSIP/${EXTEN}@gvsip1)
```
