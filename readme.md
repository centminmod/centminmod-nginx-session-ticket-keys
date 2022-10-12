Setup rotating Nginx TLS session tickey keys on Centmin Mod Nginx based servers via [`ssl_session_ticket_key`](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_ticket_key) instead of using Nginx's built-in key.

# Initial Setup

On Centmin Mod based Nginx server

```
wget -O /usr/local/bin/manage-session-keys https://github.com/centminmod/centminmod-nginx-session-ticket-keys/raw/master/manage-session-keys
chmod +x /usr/local/bin/manage-session-keys
/usr/local/bin/manage-session-keys setup

systemctl status nginx-create-session-ticket-keys.service
systemctl status usr-local-nginx-conf-session_ticket_keys.mount
```

The session ticket keys are saved to a ramdisk mounted directory at `/usr/local/nginx/conf/session_ticket_keys`

```
df -hT /usr/local/nginx/conf/session_ticket_keys
Filesystem     Type   Size  Used Avail Use% Mounted on
tmpfs          tmpfs   16G   32K   16G   1% /usr/local/nginx/conf/session_ticket_keys
```

# Manual Usage:


`/usr/local/bin/manage-session-keys` usage:

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

Where the `0000: ` line is first part of the session ticket key's hex bytes

```
/usr/local/bin/manage-session-keys check-domain google.com
------------------------------------------
New-TLSv1-SSLv3-Cipher: ECDHE-ECDSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 4382F8DA5AAA25EFB7B5544E1804DE2AC3406072894BF6429855F8BB785542DB
    Master-Key: A84625DB58796D6EF1471DD3FD690E87EBF70EEF9FEB27A392F72EA19D1A38B1107E6A53FF170020C34FAA2462015DEF
    0000: 02 22 1d 79 9a b1 78 4e-63 4b 2f db 29 aa 15 4e   .".y..xNcK-.)..N
------------------------------------------
Reused-TLSv1-SSLv3-Cipher: ECDHE-ECDSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-AES128-GCM-SHA256
    Session-ID: 4382F8DA5AAA25EFB7B5544E1804DE2AC3406072894BF6429855F8BB785542DB
    Master-Key: A84625DB58796D6EF1471DD3FD690E87EBF70EEF9FEB27A392F72EA19D1A38B1107E6A53FF170020C34FAA2462015DEF
    0000: 02 22 1d 79 9a b1 78 4e-63 4b 2f db 29 aa 15 4e   .".y..xNcK-.)..N
```

# Check Nginx TLS Session Key Rotation

Check Centmin Mod Nginx's vhost session ticket key rotation where the `0000: ` line is first part of the session ticket key's hex bytes

```
# check
/usr/local/bin/manage-session-keys check-domain domain.com
------------------------------------------
New-TLSv1-SSLv3-Cipher: ECDHE-RSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: EAB0936BEC5E8C149D660220E94A68134E58D974DD81CF5D240DA7835E577F50
    Master-Key: A5CD7C853EE4B2C1EB47E0135D2BEDA2B9B7999D4DC29613E9752EE3599C92BE12BDB3C098B364B3E5487321932993D2
    0000: 1a 8c 33 6b 46 ef f1 55-73 c1 67 de 70 0f 65 63   ..3kF..Us.g.p.ec
------------------------------------------
Reused-TLSv1-SSLv3-Cipher: ECDHE-RSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: EAB0936BEC5E8C149D660220E94A68134E58D974DD81CF5D240DA7835E577F50
    Master-Key: A5CD7C853EE4B2C1EB47E0135D2BEDA2B9B7999D4DC29613E9752EE3599C92BE12BDB3C098B364B3E5487321932993D2
    0000: 1a 8c 33 6b 46 ef f1 55-73 c1 67 de 70 0f 65 63   ..3kF..Us.g.p.ec
```
```
# rotate session ticket keys
/usr/local/bin/manage-session-keys rotate
systemctl restart nginx

# recheck
/usr/local/bin/manage-session-keys check-domain domain.com
------------------------------------------
New-TLSv1-SSLv3-Cipher: ECDHE-RSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 7D56B663C4889D4B17F5C96905F0192747D5693E21F8BC891B340773294688EB
    Master-Key: 4FE563C339F0ED03CCDDA85222DB3607A08EC649C7F64DC803BC4B1BB2999B7D5F8E9AD470A1B81F10BB9091E990F6F5
    0000: 4a 51 ee 3b 38 4b 12 67-ae 84 68 4d 09 61 9e 3f   JQ.;8K.g..hM.a.?
------------------------------------------
Reused-TLSv1-SSLv3-Cipher: ECDHE-RSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 7D56B663C4889D4B17F5C96905F0192747D5693E21F8BC891B340773294688EB
    Master-Key: 4FE563C339F0ED03CCDDA85222DB3607A08EC649C7F64DC803BC4B1BB2999B7D5F8E9AD470A1B81F10BB9091E990F6F5
    0000: 4a 51 ee 3b 38 4b 12 67-ae 84 68 4d 09 61 9e 3f   JQ.;8K.g..hM.a.?
```