# Nginx Reverse Proxy

## Overview

Reverse proxy configuration for exposing Ruby on Rails puma application server to the web and applying HTTPS certificates from LetsEncrypt certbot service.


## How to install

### HTTP only
Simple HTTP reverse proxy

Linux:
- Update packages ```sudo apt-get update```
- Upgrade packages ```sudo apt-get upgrade```
- Install nginx ```sudo apt-get install nginx```
- Modify ```puma-server-http.conf``` <app-name>, <username>, <servernames> and directory to relate to configured rails application
- Copy contents to ```puma-server.conf``` into ```/etc/nginx/sites-available/default``` (or site config file for your server)
- Restart nginx server ```sudo service nginx restart```
- Ensure NEW tcp requests on port 80 are forwarded from the router and allowed  in the firewall of the server
- Rails server should now be accessible through ip of server on standard HTTP port


### HTTPS Redirect
HTTPS server with LetsEncrypt certificate and HTTP redirect to HTTPS

Linux:
- Update packages ```sudo apt-get update```
- Upgrade packages ```sudo apt-get upgrade```
- Install nginx ```sudo apt-get install nginx```
- Install certbot ```sudo apt-get install certbot python-certbot-nginx```
- Stop any currently running puma server processes ```pumactl stop```
- Modify ```puma-server-http.conf``` <app-name>, <username>, <servernames> and directory to relate to configured rails application
- Copy contents to ```puma-server.conf``` into ```/etc/nginx/sites-available/default``` (or site config file for your server)
- Restart nginx server ```sudo service nginx restart```
- Create SSL certificates by running ```sudo certbot --nginx``` and following the GUI
- Check any changes to the nginx conf file are acceptable (or copy ```puma-server-https.conf``` and modify to relate to configured rails application)
- Restart nginx server ```sudo service nginx restart```
- Ensure NEW tcp requests on port 80 and 443 are forwarded from the router and allowed  in the firewall of the server
- Rails server should now be accessible through ip of server on HTTPS
- Test HTTPS with [SSL Server Test](https://www.ssllabs.com/ssltest/)

Next its time to disable TLS 1.0/1.1 to remove weaker ciphers and vulnerabilities in their implementation and enable TLS 1.2/1.3 and HTTP/2:
- Backup the following files:
  - ```sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup-tls```
  - ```sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.backup-tls```
  - ```sudo cp /etc/letsencrypt/options-ssl-nginx.conf /etc/letsencrypt/options-ssl-nginx.conf.backup-tls```
- Open the nginx config in your editor ```sudo vim /etc/nginx/nginx.conf``` then edit the ```ssl_protocols``` line in the ```http {}``` block, replacing ```TLSv1 TLSv1.1``` with ```TLSv1.2 TLSv1.3```
- Open the LetsEncrypt config file in your editor ```sudo vim /etc/letsencrypt/options-ssl-nginx.conf``` then edit the ```ssl_protocols``` line, replacing ```TLSv1 TLSv1.1``` with ```TLSv1.2 TLSv1.3```
- Open the nginx site config file in your editor ```sudo vim /etc/nginx/sites-available/default``` then edit the ```listen 443 ssl default_server;``` line in the ```server``` block, and insert ```http2``` inbetween ```ssl``` and ```defualt_server```, if you copied https the config file from this repository then this should already be done
- Check nginx config is ok using ```sudo nginx -t``` then reload using ```sudo service nginx restart```
- Check certbot renewal is ok using ```sudo certbot renew --dry-run```
- Check TLSv1.0/1.1 are disabled by using [SSL Server Test](https://www.ssllabs.com/ssltest/) again

To increase security it is possible to create a stronger DH safe prime and change the exchange parameters to include a HTHS header:
- Make new directory for keys ```sudo mkdir /etc/pki/nginx```
- Generate new keys ```sudo openssl dhparam -out /etc/pki/nginx/dhparams.pem 2048```
- Enter nginx config file ```/etc/nginx/sites-available/default``` and modify 'ssl_dhparam' to match the filepath for the newly generated keys ```ssl dhparam "/etc/pki/nginx/dhparams.pem";```
- While in the nginx config file also add the HTHS header to the TLS server config ```add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;```

LetsEncrypt allows for auto-renewal of SSL certificates:
- Test auto renewal using ```sudo /usr/bin/certbot renew --dry-run```
- Open cron.service ```crontab -e``` and add ```45 00 * * * /usr/bin/certbot renew --quiet --no-self-upgrade --renew-hook "service nginx restart"``` to run the update at 45mins past midnight each day

OPTIONAL: Further security increases on the nginx server:

Improve SSL session cache, disable SSL session tickets:
- Open the LetsEncrypt config file in your editor ```sudo vim /etc/letsencrypt/options-ssl-nginx.conf``` then replace the ssl cache settings with: 
```
ssl_session_cache shared:le_nginx_SSL:40m; # holds approx 40 x 4000 sessions
ssl_session_timeout 2h;
ssl_session_tickets off;
```

Improve the TLSv1.2/1.3 cipher suite:
- Check the cipher list for your grade of secuity on Mozilla's website [here](https://wiki.mozilla.org/Security/Server_Side_TLS)
- Open the LetsEncrypt ssl configuration file ```sudo vim /etc/letsencrypt/options-ssl-nginx.conf```
- Replace the list of ```ssl_ciphers``` with the recommended set from Mozilla, for example:
```
ssl_ciphers "TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
```
- Restart the nginx server using ```sudo service nginx restart```

X-Content-Type-Settings:
- Open the nginx site config in your editor ```sudo vim /etc/nginx/sites-available/default``` and add the following lines to the server block (already added if you copied the https config file):
```
add_header X-Frame-Options SAMEORIGIN;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```
- Restart the nginx server using ```sudo service nginx restart```

Work through the rest of the Mozilla web [secuirty cheatsheet](https://infosec.mozilla.org/guidelines/web_security) to match your sites needs 