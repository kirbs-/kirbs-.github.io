---
layout: page
title: Nextcloud setup with Centos 7 and Apache
order: 1
---
1. `mkdir /var/www/html/nextcloud`
2. `cd /var/www/html/nextcloud`
3. `git clone nextcloud server`
4. `mkdir /home/data`
5. `chown apache:apache -R nextcloud`
6. `chown apache:apache /home/data`
7. install PHP 5.6
8. Install ius-release rpm
9. Install epel-release rpm
11. `cd /var/www/nextcloud`
11. `git submodule update --init`
12. add virtual host
```
<VirtualHost *:80>
	ServerName <url>
	DocumentRoot /var/www/html/nextcloud/server
</Virtualhost>
```
13. (Optional) Enable HTTPS
```		
<VirtualHost *:443>
	ServerName example.com
	SSLEngine On
	SSLCertificateFile /path/to/certificate
	SSLCertificateKeyFile /path/to/keyfie
	SSLCACertificateFile /path/to/ca/certificate
	DocumentRoot /var/www/html/nextcloud/server
	<Directory /var/www/html/nextcloud/server>
		AllowOverride all
		Options -MultiViews
	</Directory>
</Virtualhost>
```
14. restart httpd `sudo systemctl restart httpd`
15. Create nextcloud database user and database
16. Visit http://<address>/nextcloud and complete setup
