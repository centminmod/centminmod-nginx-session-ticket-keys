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
------------------------------------------
New-TLSv1-SSLv3-Cipher ECDHE-ECDSA-AES128-GCM-SHA256
Protocol TLSv1.2
Cipher ECDHE-ECDSA-AES128-GCM-SHA256
Session-ID 26631C59D27EC12A36B9BB3495EC1DB46AC4F8F474128C24CD4D6582E044DB3C
Master-Key 9EB568AEF21FC0B2A6D209A5ABECDD81E9891CD97064ADB3BAB50693DECD3E57399D90814EA02CC227A67933390E7C58
------------------------------------------
Reused-TLSv1-SSLv3-Cipher ECDHE-ECDSA-AES128-GCM-SHA256
Protocol TLSv1.2
Cipher ECDHE-ECDSA-AES128-GCM-SHA256
Session-ID 26631C59D27EC12A36B9BB3495EC1DB46AC4F8F474128C24CD4D6582E044DB3C
Master-Key 9EB568AEF21FC0B2A6D209A5ABECDD81E9891CD97064ADB3BAB50693DECD3E57399D90814EA02CC227A67933390E7C58
```

# Check Nginx TLS Session Key Rotation

Check Centmin Mod Nginx's vhost session ticket key rotation

```
# check
/usr/local/bin/manage-session-keys check-domain domain.com
------------------------------------------
New-TLSv1-SSLv3-Cipher ECDHE-RSA-AES128-GCM-SHA256
Protocol TLSv1.2
Cipher ECDHE-RSA-AES128-GCM-SHA256
Session-ID 20F8D5519F33D61162C9D33F4B2F2BE79F5BF0D70B316224130958821EFEB347
Master-Key 05F8696C5F53CB75AEFFDE833FAD263507D4EDF474FD67EE41235C2A223F7CAD77242454F5DDF9E5F100EB872846D048
```
```
# rotate session ticket keys
/usr/local/bin/manage-session-keys rotate

# recheck
/usr/local/bin/manage-session-keys check-domain domain.com
------------------------------------------
New-TLSv1-SSLv3-Cipher ECDHE-RSA-AES128-GCM-SHA256
Protocol TLSv1.2
Cipher ECDHE-RSA-AES128-GCM-SHA256
Session-ID E2C17A078D8C4091A02C476159EF1239E331ECDB87EE12A48A6EFA3AC4FFE1D1
Master-Key 0F9D89A5E3805F283629BE4D949364A002E43DAB1613652D0BE34DF4D1D26853782CBAB6422AE22BC116B70656BAD120
------------------------------------------
Reused-TLSv1-SSLv3-Cipher ECDHE-RSA-AES128-GCM-SHA256
Protocol TLSv1.2
Cipher ECDHE-RSA-AES128-GCM-SHA256
Session-ID E2C17A078D8C4091A02C476159EF1239E331ECDB87EE12A48A6EFA3AC4FFE1D1
Master-Key 0F9D89A5E3805F283629BE4D949364A002E43DAB1613652D0BE34DF4D1D26853782CBAB6422AE22BC116B70656BAD120
```