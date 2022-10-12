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
/usr/local/bin/manage-session-keys check-domain domain.com
------------------------------------------
New-TLSv1-SSLv3-Cipher: ECDHE-RSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 88EFBC7609076696D65C7ED3D7EE8658461928BA4D6FB95EB137DCD2D4595728
    Master-Key: CA12DB554A6000A15D0632810EB606F0E5D0EE62CA40BCB1EC24ACBBA29FE2B00B03DA9282AC314C40AD1C6E40187B30
    TLS session ticket lifetime hint: 600 (seconds)
    0000: 3d 86 62 77 bd 61 a9 ac-f3 e5 09 a0 81 0d cf 2a   =.bw.a.........*
    Start Time: 1665605339
------------------------------------------
Reused-TLSv1-SSLv3-Cipher: ECDHE-RSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 88EFBC7609076696D65C7ED3D7EE8658461928BA4D6FB95EB137DCD2D4595728
    Master-Key: CA12DB554A6000A15D0632810EB606F0E5D0EE62CA40BCB1EC24ACBBA29FE2B00B03DA9282AC314C40AD1C6E40187B30
    TLS session ticket lifetime hint: 600 (seconds)
    0000: 3d 86 62 77 bd 61 a9 ac-f3 e5 09 a0 81 0d cf 2a   =.bw.a.........*
    Start Time: 1665605339
------------------------------------------
    nginx process ids: 12375 12360 12357 12340 12291 12290 12289 12288 12287 12286
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
    Session-ID: 88EFBC7609076696D65C7ED3D7EE8658461928BA4D6FB95EB137DCD2D4595728
    Master-Key: CA12DB554A6000A15D0632810EB606F0E5D0EE62CA40BCB1EC24ACBBA29FE2B00B03DA9282AC314C40AD1C6E40187B30
    TLS session ticket lifetime hint: 600 (seconds)
    0000: 3d 86 62 77 bd 61 a9 ac-f3 e5 09 a0 81 0d cf 2a   =.bw.a.........*
    Start Time: 1665605339
------------------------------------------
Reused-TLSv1-SSLv3-Cipher: ECDHE-RSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 88EFBC7609076696D65C7ED3D7EE8658461928BA4D6FB95EB137DCD2D4595728
    Master-Key: CA12DB554A6000A15D0632810EB606F0E5D0EE62CA40BCB1EC24ACBBA29FE2B00B03DA9282AC314C40AD1C6E40187B30
    TLS session ticket lifetime hint: 600 (seconds)
    0000: 3d 86 62 77 bd 61 a9 ac-f3 e5 09 a0 81 0d cf 2a   =.bw.a.........*
    Start Time: 1665605339
------------------------------------------
    nginx process ids: 12375 12360 12357 12340 12291 12290 12289 12288 12287 12286
```
```
# rotate session ticket keys
systemctl restart nginx-rotate-session-ticket-keys.timer

# recheck
/usr/local/bin/manage-session-keys check-domain domain.com
------------------------------------------
New-TLSv1-SSLv3-Cipher: ECDHE-RSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 046839BE610F884D09869838CF17EC18B3B906E2FBE597CEB36F213F3AACAB2A
    Master-Key: 1CB501C785333C599D9B4E8920D249E5DCB780A50471E46405F01AD0E84A726DC60890D1D24AEC6DB2A2139F420E7A73
    TLS session ticket lifetime hint: 600 (seconds)
    0000: c6 aa 2c 0e 14 89 9b 57-34 a3 20 d3 0d 0f 82 33   ..,....W4. ....3
    Start Time: 1665608624
------------------------------------------
Reused-TLSv1-SSLv3-Cipher: ECDHE-RSA-AES128-GCM-SHA256
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 046839BE610F884D09869838CF17EC18B3B906E2FBE597CEB36F213F3AACAB2A
    Master-Key: 1CB501C785333C599D9B4E8920D249E5DCB780A50471E46405F01AD0E84A726DC60890D1D24AEC6DB2A2139F420E7A73
    TLS session ticket lifetime hint: 600 (seconds)
    0000: c6 aa 2c 0e 14 89 9b 57-34 a3 20 d3 0d 0f 82 33   ..,....W4. ....3
    Start Time: 1665608624
------------------------------------------
    nginx process ids: 35751 35724 35717 35700 35667 35650 35649 35648 35647 18030
```