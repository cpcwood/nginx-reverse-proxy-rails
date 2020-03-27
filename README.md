# Nginx Reverse Proxy

## Overview

Reverse proxy configuration for exposing Ruby on Rails puma application server to the web.


## How to install

Linux:
- Update packages ```sudo apt-get update```
- Install nginx ```sudo apt-get install nginx```
- Modify ```puma-server.conf``` <app-name>, <username>, <servername> and directory to relate to configured rails application
- Copy contents to ```puma-server.conf``` into ```/etc/nginx/sites-available/default``` (or site config file for your server)
- Restart nginx server ```sudo service nginx restart```
- Rails server should now be accessible through ip of server on standard HTTP port