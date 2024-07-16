# Name Server Setting
<hr/>

[1. Nginx Installation](#nginx-installation)

[2. Nginx Setting](#nginx-setting)

[3. Certbot - SSL certificates](#certbot-ssl-certificates)

[4. Renew Certificate](#renew-certificate)

* Reference
	* Nginx <https://www.nginx.com>
	* Certbot <https://certbot.eff.org>

<hr/>

## Nginx Installation

* Install Nginx
	* `sudo apt install nginx`

* Service start
	* `sudo systemctl start nginx`
	* `sudo systemctl enable nginx`	

## Nginx Setting

* `cd /etc/nginx/sites-available`

	```
	server {
		server_name ymcloud.yemilab.kr;
		root /var/www/html;
		index index.html;

		access_log /var/log/nginx/ymcloud-access.log;
		error_log  /var/log/nginx/ymcloud-error.log;
		client_max_body_size 16G;

		location / {
			proxy_pass http://localhost:8080/;
		}
	}
	```

* Make a link to enable
	* `cd /etc/nginx/sites-enabled`
	* `sudo ln -s ../sites-available/owncloud`

## Certbot - SSL certificates

* Reference <https://certbot.eff.org>

* Certbot : 
	* Connection with port &rarr; without port
	* HTTP &rarr; HTTPS

* Install core : `sudo snap install core; sudo snap refresh core`
* Install Certbot : `sudo snap install --classic certbot`
* Prepare command : `sudo ln -s /snap/bin/certbot /usr/bin/certbot`

* Get certificate & update Nginx setting : `sudo certbot --nginx`

## Renew Certificate

* The certificate is a short-term certificate for 90 days

* So we must renew certificate every 90 days

* Check expired date : `sudo certbot certificates`

<img src = "./images/3_dns/dns-01.png" width = "80%"></img>

* Renew Test : `sudo certbot renew --dry-run`
	* If there are no problems with the renewal process, you can achieve "success"
	
<img src = "./images/3_dns/dns-02.png" width = "80%"></img>

* Renew : `sudo certbot renew`

### Renew certificate automatically by Crontab

* Crontab rule

<img src = "./images/3_dns/dns-03.png" width = "80%"></img>

* Make executable file (`chmod +x file_name`) at `/bin/`

```
!/bin/bash

date >> /var/log/nginx/renew.log
/usr/bin/certbot renew --post-hook "systemctl restart nginx" >> /var/log/nginx/renew.log
```
* Crontab scheduler setting

`sudo crontab -e`

* Renew certificate 0:00, 1st day, every month 

`0 0 1 * * file_name`

* Log : `/var/log/nginx/renew.log`