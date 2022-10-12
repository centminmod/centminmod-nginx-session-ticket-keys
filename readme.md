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

New, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: CF6721C0D4B08214B5845AA182C89983D3450233C920473A25A3F3D70071B216
    Master-Key: 196BB506A364130B446A1A8D8AA2C291D0114111F53D10CAA84DB402A28FFDBA38D8A07B44056DD4B618D6D4F27E2B9A
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: CF6721C0D4B08214B5845AA182C89983D3450233C920473A25A3F3D70071B216
    Master-Key: 196BB506A364130B446A1A8D8AA2C291D0114111F53D10CAA84DB402A28FFDBA38D8A07B44056DD4B618D6D4F27E2B9A
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: CF6721C0D4B08214B5845AA182C89983D3450233C920473A25A3F3D70071B216
    Master-Key: 196BB506A364130B446A1A8D8AA2C291D0114111F53D10CAA84DB402A28FFDBA38D8A07B44056DD4B618D6D4F27E2B9A
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: CF6721C0D4B08214B5845AA182C89983D3450233C920473A25A3F3D70071B216
    Master-Key: 196BB506A364130B446A1A8D8AA2C291D0114111F53D10CAA84DB402A28FFDBA38D8A07B44056DD4B618D6D4F27E2B9A
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: CF6721C0D4B08214B5845AA182C89983D3450233C920473A25A3F3D70071B216
    Master-Key: 196BB506A364130B446A1A8D8AA2C291D0114111F53D10CAA84DB402A28FFDBA38D8A07B44056DD4B618D6D4F27E2B9A
Reused, TLSv1/SSLv3, Cipher is ECDHE-ECDSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: CF6721C0D4B08214B5845AA182C89983D3450233C920473A25A3F3D70071B216
    Master-Key: 196BB506A364130B446A1A8D8AA2C291D0114111F53D10CAA84DB402A28FFDBA38D8A07B44056DD4B618D6D4F27E2B9A
```