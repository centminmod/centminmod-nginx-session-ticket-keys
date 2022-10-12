Setup rotating Nginx TLS session tickey keys on Centmin Mod Nginx based servers via [`ssl_session_ticket_key`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_ticket_key) instead of using Nginx's built-in key.

# Initial Setup

On Centmin Mod based Nginx server

```
wget -O /usr/local/bin/manage-session-keys https://github.com/centminmod/centminmod-nginx-session-ticket-keys/raw/master/manage-session-keys
chmod +x /usr/local/bin/manage-session-keys
/usr/local/bin/manage-session-keys setup

systemctl status nginx-create-session-ticket-keys.service
systemctl status nginx-rotate-session-ticket-keys.service
systemctl status nginx-rotate-session-ticket-keys.timer
systemctl status nginx-session_ticket_keys.mount
```
```
Usage:

/usr/local/bin/manage-session-keys setup
/usr/local/bin/manage-session-keys create
/usr/local/bin/manage-session-keys rotate
/usr/local/bin/manage-session-keys uninstall
/usr/local/bin/manage-session-keys status
/usr/local/bin/manage-session-keys check-domain yourdomain.com
```

Add to each Centmin Mod Nginx Vhosts with HTTPS enabled i.e. `/usr/local/nginx/conf/conf.d/domain.com.ssl.conf` the following:

```
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/1.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/2.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/3.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/4.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/5.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/6.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/7.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/8.key;
```

Example excerpt

```
server {
  listen 443 ssl http2;
  server_name domain.com www.domain.com;

  include /usr/local/nginx/conf/ssl/domain.com/domain.com.crt.key.conf;
  include /usr/local/nginx/conf/ssl_include.conf;
  ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/1.key;
  ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/2.key;
  ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/3.key;
  ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/4.key;
  ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/5.key;
  ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/6.key;
  ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/7.key;
  ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/8.key;
```

Or instead of within each Nginx vhost, add globally to existing include file `/usr/local/nginx/conf/ssl_include.conf` via a new include file `/usr/local/nginx/conf/ssl-session-ticket-keys.conf`.

Existing `/usr/local/nginx/conf/ssl_include.conf`include file with added `/usr/local/nginx/conf/ssl-session-ticket-keys.conf` include file:
```
ssl_session_cache      shared:SSL:10m;
ssl_session_timeout    60m;
ssl_protocols  TLSv1.2 TLSv1.3;
include /usr/local/nginx/conf/ssl-session-ticket-keys.conf;
```

In `/usr/local/nginx/conf/ssl-session-ticket-keys.conf`

```
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/1.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/2.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/3.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/4.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/5.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/6.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/7.key;
ssl_session_ticket_key /usr/local/nginx/conf/session_ticket_keys/8.key;
```

**Notes**

* `/usr/local/nginx/conf/ssl-session-ticket-keys.conf` is automatically created and populated if you run `/usr/local/bin/manage-session-keys setup` from [initial setup](#initial-setup) steps.

# Check Domain's Session Resumption

```
/usr/local/bin/manage-session-keys check-domain google.com

CONNECTED(00000003)
depth=3 C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA
verify return:1
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R1
verify return:1
depth=1 C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
verify return:1
depth=0 CN = *.google.com
verify return:1
---
Certificate chain
 0 s:/CN=*.google.com
   i:/C=US/O=Google Trust Services LLC/CN=GTS CA 1C3
 1 s:/C=US/O=Google Trust Services LLC/CN=GTS CA 1C3
   i:/C=US/O=Google Trust Services LLC/CN=GTS Root R1
 2 s:/C=US/O=Google Trust Services LLC/CN=GTS Root R1
   i:/C=BE/O=GlobalSign nv-sa/OU=Root CA/CN=GlobalSign Root CA
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIN7TCCDNWgAwIBAgIQJj7Rgvf7Iw4K092uo999iDANBgkqhkiG9w0BAQsFADBG
MQswCQYDVQQGEwJVUzEiMCAGA1UEChMZR29vZ2xlIFRydXN0IFNlcnZpY2VzIExM
QzETMBEGA1UEAxMKR1RTIENBIDFDMzAeFw0yMjA5MTIwODE3MDBaFw0yMjEyMDUw
ODE2NTlaMBcxFTATBgNVBAMMDCouZ29vZ2xlLmNvbTBZMBMGByqGSM49AgEGCCqG
SM49AwEHA0IABOCYGChq2xG/2slS+0nGt2csiDKNE8fTbn1BZAkKIrO+3wDauTKn
WynvUvnHxztlNt86h7YXagRdCnCysMNlt9KjggvPMIILyzAOBgNVHQ8BAf8EBAMC
B4AwEwYDVR0lBAwwCgYIKwYBBQUHAwEwDAYDVR0TAQH/BAIwADAdBgNVHQ4EFgQU
XoTxICEfJtHnENW2boGeagAOmEowHwYDVR0jBBgwFoAUinR/r4XN7pXNPZzQ4kYU
83E1HScwagYIKwYBBQUHAQEEXjBcMCcGCCsGAQUFBzABhhtodHRwOi8vb2NzcC5w
a2kuZ29vZy9ndHMxYzMwMQYIKwYBBQUHMAKGJWh0dHA6Ly9wa2kuZ29vZy9yZXBv
L2NlcnRzL2d0czFjMy5kZXIwggl/BgNVHREEggl2MIIJcoIMKi5nb29nbGUuY29t
ghYqLmFwcGVuZ2luZS5nb29nbGUuY29tggkqLmJkbi5kZXaCFSoub3JpZ2luLXRl
c3QuYmRuLmRldoISKi5jbG91ZC5nb29nbGUuY29tghgqLmNyb3dkc291cmNlLmdv
b2dsZS5jb22CGCouZGF0YWNvbXB1dGUuZ29vZ2xlLmNvbYILKi5nb29nbGUuY2GC
CyouZ29vZ2xlLmNsgg4qLmdvb2dsZS5jby5pboIOKi5nb29nbGUuY28uanCCDiou
Z29vZ2xlLmNvLnVrgg8qLmdvb2dsZS5jb20uYXKCDyouZ29vZ2xlLmNvbS5hdYIP
Ki5nb29nbGUuY29tLmJygg8qLmdvb2dsZS5jb20uY2+CDyouZ29vZ2xlLmNvbS5t
eIIPKi5nb29nbGUuY29tLnRygg8qLmdvb2dsZS5jb20udm6CCyouZ29vZ2xlLmRl
ggsqLmdvb2dsZS5lc4ILKi5nb29nbGUuZnKCCyouZ29vZ2xlLmh1ggsqLmdvb2ds
ZS5pdIILKi5nb29nbGUubmyCCyouZ29vZ2xlLnBsggsqLmdvb2dsZS5wdIISKi5n
b29nbGVhZGFwaXMuY29tgg8qLmdvb2dsZWFwaXMuY26CESouZ29vZ2xldmlkZW8u
Y29tggwqLmdzdGF0aWMuY26CECouZ3N0YXRpYy1jbi5jb22CD2dvb2dsZWNuYXBw
cy5jboIRKi5nb29nbGVjbmFwcHMuY26CEWdvb2dsZWFwcHMtY24uY29tghMqLmdv
b2dsZWFwcHMtY24uY29tggxna2VjbmFwcHMuY26CDiouZ2tlY25hcHBzLmNughJn
b29nbGVkb3dubG9hZHMuY26CFCouZ29vZ2xlZG93bmxvYWRzLmNughByZWNhcHRj
aGEubmV0LmNughIqLnJlY2FwdGNoYS5uZXQuY26CEHJlY2FwdGNoYS1jbi5uZXSC
EioucmVjYXB0Y2hhLWNuLm5ldIILd2lkZXZpbmUuY26CDSoud2lkZXZpbmUuY26C
EWFtcHByb2plY3Qub3JnLmNughMqLmFtcHByb2plY3Qub3JnLmNughFhbXBwcm9q
ZWN0Lm5ldC5jboITKi5hbXBwcm9qZWN0Lm5ldC5jboIXZ29vZ2xlLWFuYWx5dGlj
cy1jbi5jb22CGSouZ29vZ2xlLWFuYWx5dGljcy1jbi5jb22CF2dvb2dsZWFkc2Vy
dmljZXMtY24uY29tghkqLmdvb2dsZWFkc2VydmljZXMtY24uY29tghFnb29nbGV2
YWRzLWNuLmNvbYITKi5nb29nbGV2YWRzLWNuLmNvbYIRZ29vZ2xlYXBpcy1jbi5j
b22CEyouZ29vZ2xlYXBpcy1jbi5jb22CFWdvb2dsZW9wdGltaXplLWNuLmNvbYIX
Ki5nb29nbGVvcHRpbWl6ZS1jbi5jb22CEmRvdWJsZWNsaWNrLWNuLm5ldIIUKi5k
b3VibGVjbGljay1jbi5uZXSCGCouZmxzLmRvdWJsZWNsaWNrLWNuLm5ldIIWKi5n
LmRvdWJsZWNsaWNrLWNuLm5ldIIOZG91YmxlY2xpY2suY26CECouZG91YmxlY2xp
Y2suY26CFCouZmxzLmRvdWJsZWNsaWNrLmNughIqLmcuZG91YmxlY2xpY2suY26C
EWRhcnRzZWFyY2gtY24ubmV0ghMqLmRhcnRzZWFyY2gtY24ubmV0gh1nb29nbGV0
cmF2ZWxhZHNlcnZpY2VzLWNuLmNvbYIfKi5nb29nbGV0cmF2ZWxhZHNlcnZpY2Vz
LWNuLmNvbYIYZ29vZ2xldGFnc2VydmljZXMtY24uY29tghoqLmdvb2dsZXRhZ3Nl
cnZpY2VzLWNuLmNvbYIXZ29vZ2xldGFnbWFuYWdlci1jbi5jb22CGSouZ29vZ2xl
dGFnbWFuYWdlci1jbi5jb22CGGdvb2dsZXN5bmRpY2F0aW9uLWNuLmNvbYIaKi5n
b29nbGVzeW5kaWNhdGlvbi1jbi5jb22CJCouc2FmZWZyYW1lLmdvb2dsZXN5bmRp
Y2F0aW9uLWNuLmNvbYIWYXBwLW1lYXN1cmVtZW50LWNuLmNvbYIYKi5hcHAtbWVh
c3VyZW1lbnQtY24uY29tggtndnQxLWNuLmNvbYINKi5ndnQxLWNuLmNvbYILZ3Z0
Mi1jbi5jb22CDSouZ3Z0Mi1jbi5jb22CCzJtZG4tY24ubmV0gg0qLjJtZG4tY24u
bmV0ghRnb29nbGVmbGlnaHRzLWNuLm5ldIIWKi5nb29nbGVmbGlnaHRzLWNuLm5l
dIIMYWRtb2ItY24uY29tgg4qLmFkbW9iLWNuLmNvbYINKi5nc3RhdGljLmNvbYIU
Ki5tZXRyaWMuZ3N0YXRpYy5jb22CCiouZ3Z0MS5jb22CESouZ2NwY2RuLmd2dDEu
Y29tggoqLmd2dDIuY29tgg4qLmdjcC5ndnQyLmNvbYIQKi51cmwuZ29vZ2xlLmNv
bYIWKi55b3V0dWJlLW5vY29va2llLmNvbYILKi55dGltZy5jb22CC2FuZHJvaWQu
Y29tgg0qLmFuZHJvaWQuY29tghMqLmZsYXNoLmFuZHJvaWQuY29tggRnLmNuggYq
LmcuY26CBGcuY2+CBiouZy5jb4IGZ29vLmdsggp3d3cuZ29vLmdsghRnb29nbGUt
YW5hbHl0aWNzLmNvbYIWKi5nb29nbGUtYW5hbHl0aWNzLmNvbYIKZ29vZ2xlLmNv
bYISZ29vZ2xlY29tbWVyY2UuY29tghQqLmdvb2dsZWNvbW1lcmNlLmNvbYIIZ2dw
aHQuY26CCiouZ2dwaHQuY26CCnVyY2hpbi5jb22CDCoudXJjaGluLmNvbYIIeW91
dHUuYmWCC3lvdXR1YmUuY29tgg0qLnlvdXR1YmUuY29tghR5b3V0dWJlZWR1Y2F0
aW9uLmNvbYIWKi55b3V0dWJlZWR1Y2F0aW9uLmNvbYIPeW91dHViZWtpZHMuY29t
ghEqLnlvdXR1YmVraWRzLmNvbYIFeXQuYmWCByoueXQuYmWCGmFuZHJvaWQuY2xp
ZW50cy5nb29nbGUuY29tghtkZXZlbG9wZXIuYW5kcm9pZC5nb29nbGUuY26CHGRl
dmVsb3BlcnMuYW5kcm9pZC5nb29nbGUuY26CGHNvdXJjZS5hbmRyb2lkLmdvb2ds
ZS5jbjAhBgNVHSAEGjAYMAgGBmeBDAECATAMBgorBgEEAdZ5AgUDMDwGA1UdHwQ1
MDMwMaAvoC2GK2h0dHA6Ly9jcmxzLnBraS5nb29nL2d0czFjMy9RT3ZKME4xc1Qy
QS5jcmwwggEEBgorBgEEAdZ5AgQCBIH1BIHyAPAAdgBRo7D1/QF5nFZtuDd4jwyk
eswbJ8v3nohCmg3+1IsF5QAAAYMw/O3FAAAEAwBHMEUCIEQk2EUAX3M5/z1Q28Ji
uu2OPoUsKBugb++/BfvvwyZqAiEAri32JynGOZ3mehit+ypuFsqSAL/lS1VJKN7v
BOHV9YUAdgBGpVXrdfqRIDC1oolp9PN9ESxBdL79SbiFq/L8cP5tRwAAAYMw/O3Y
AAAEAwBHMEUCIQCKeeZZqKMbL0M0daBbhisQlttAo3sF4FJ0wMtWPCWg2AIgI7cf
q6XI3GWrxRy6A2nMGw4MYwPiieUBrxaLbOTA7J8wDQYJKoZIhvcNAQELBQADggEB
AEdWIZMhoQKC7DxMgoG7IIO8QeMFNsEndd7Y34k+B0XIxmSniGQWeDW42Af8yLUW
ejJCh11CgpEsBSUFR47SFkW2JDAjGUc98iQM3q0+Zsw4g2LBWtEGgIqfaAGVHNsu
OSSFNdH28mi4oo7Kw6c2NK7JkyKVt3nhZOCyduwdvegAjG5hOM+Ydwz2IEjZMxbb
4hHSQVrzPZErLsrIIXRAwnouLDrPfSJKZmZvBbwYLUcRUaOrRHaaRI7HBBxbUC+b
oDqQ3OBA2PGdABMBs/7ZbfBD2w0PX+Zqief4mvvldfmJJyWzSAj6rIA4qMkfMv76
0csBvOhYj2iDFJCsyJWTUhc=
-----END CERTIFICATE-----
subject=/CN=*.google.com
issuer=/C=US/O=Google Trust Services LLC/CN=GTS CA 1C3
---
No client certificate CA names sent
Peer signing digest: SHA256
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 6712 bytes and written 430 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
Server public key is 256 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 92BEA34D6878903C0B80B792A0DB4E57E0DC7EE3D3EDEB2431BB887E81DE3874
    Session-ID-ctx: 
    Master-Key: 11944A9652CC80361A953121403B9108EF8FEE3747A45BABDFEE07ACA269970CB53BFE9ED776F3BB15DEAE0C36A25DFC
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1665589712
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
drop connection and then reconnect
CONNECTED(00000003)
---
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 92BEA34D6878903C0B80B792A0DB4E57E0DC7EE3D3EDEB2431BB887E81DE3874
    Session-ID-ctx: 
    Master-Key: 11944A9652CC80361A953121403B9108EF8FEE3747A45BABDFEE07ACA269970CB53BFE9ED776F3BB15DEAE0C36A25DFC
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1665589712
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
drop connection and then reconnect
CONNECTED(00000003)
---
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 92BEA34D6878903C0B80B792A0DB4E57E0DC7EE3D3EDEB2431BB887E81DE3874
    Session-ID-ctx: 
    Master-Key: 11944A9652CC80361A953121403B9108EF8FEE3747A45BABDFEE07ACA269970CB53BFE9ED776F3BB15DEAE0C36A25DFC
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1665589712
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
drop connection and then reconnect
CONNECTED(00000003)
---
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 92BEA34D6878903C0B80B792A0DB4E57E0DC7EE3D3EDEB2431BB887E81DE3874
    Session-ID-ctx: 
    Master-Key: 11944A9652CC80361A953121403B9108EF8FEE3747A45BABDFEE07ACA269970CB53BFE9ED776F3BB15DEAE0C36A25DFC
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1665589712
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
drop connection and then reconnect
CONNECTED(00000003)
---
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 92BEA34D6878903C0B80B792A0DB4E57E0DC7EE3D3EDEB2431BB887E81DE3874
    Session-ID-ctx: 
    Master-Key: 11944A9652CC80361A953121403B9108EF8FEE3747A45BABDFEE07ACA269970CB53BFE9ED776F3BB15DEAE0C36A25DFC
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1665589712
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
drop connection and then reconnect
CONNECTED(00000003)
---
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 92BEA34D6878903C0B80B792A0DB4E57E0DC7EE3D3EDEB2431BB887E81DE3874
    Session-ID-ctx: 
    Master-Key: 11944A9652CC80361A953121403B9108EF8FEE3747A45BABDFEE07ACA269970CB53BFE9ED776F3BB15DEAE0C36A25DFC
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    Start Time: 1665589712
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
DONE
```