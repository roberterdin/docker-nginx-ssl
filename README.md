# docker-nginx-ssl
An updated version of the official [nginx container](https://hub.docker.com/_/nginx/) for working SSL. For detailed configuration possibilities refer to the original documentation.

## Example default.conf
This configuration should result in an A+ on the [Qualys SSL Server Test](https://www.ssllabs.com/ssltest/), given your certificate and keys are good.
```
server {
    listen 443 ssl;
    server_name example.io www.example.io;

    ssl_certificate /etc/hostkeys/fullchain.pem;
    ssl_certificate_key /etc/hostkeys/privkey.pem;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/hostkeys/dh.pem;
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:
DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECD
HE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS
-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!E
XPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_stapling on;
    ssl_stapling_verify on;
    add_header Strict-Transport-Security max-age=15768000;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

# redirect traffic on port 80 to port 443
server {
    listen 80;
    server_name example.io www.example.io;
    return 301 https://$host$request_uri;
}
```
## Keys
In addition to the full certificate chain (fullchain.pem) and the private key (privkey.pem) you need a DH keypair. This can be created as follows
```
sudo openssl dhparam -out <keys-dir>/dh.pem 2048
```

## run
```shell
docker run \
-v <web-dir>:/usr/share/nginx/html:ro \
-v <keys-dir>:/etc/hostkeys \
-v <config-dir>/default.conf:/etc/nginx/conf.d/default.conf \
-p 80:80 \
-p 443:443 \
roberterdin/docker-nginx-ssl:latest
```
