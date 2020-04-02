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

To increase security it is possible to create a stronger DH safe prime and change the exchange parameters to include a HTHS header:
- Make new directory for keys ```sudo mkdir /etc/pki/nginx```
- Generate new keys ```sudo openssl dhparam -out /etc/pki/nginx/dhparams.pem 2048```
- Enter nginx config file ```/etc/nginx/sites-available/default``` and modify 'ssl_dhparam' to match the filepath for the newly generated keys ```ssl dhparam "/etc/pki/nginx/dhparams.pem";```
- While in the nginx config file also add the HTHS header to the TLS server config ```add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;```

LetsEncrypt allows for auto-renewal of SSL certificates:
- Test auto renewal using ```sudo /usr/bin/certbot renew --dry-run```
- Open cron.service ```crontab -e``` and add ```45 00 * * * /usr/bin/certbot renew --quiet --no-self-upgrade --renew-hook "service nginx restart"``` to run the update at 45mins past midnight each day

