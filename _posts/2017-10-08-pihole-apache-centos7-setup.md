Centos 7 Apache Pi-Hole Setup

clone repo
disable ip4andor6 line
edit setupVar.conf and add IP4 address
copy files to /var/www
create file /etc/dnsmask.d/00-pihole-custom.conf (this allows the apache server at the IP address to respond to subdomain requests).
'''
address=/.brains.lan/192.168.1.101
'''
Add viritual host setup

<VirtualHost *:80>
 ServerName 192.168.1.101
 DocumentRoot /var/www/html/admin
 <Directory /var/www/html/admin>
  AllowOverride all
  Options -MultiViews
  Require all granted
 </Directory>
</VirtualHost>

sudo systemctl restart httpd

sudo firewall-cmd --add-service=dns --permanent 
sudo firewall-cmd --reload

Modify log rotation to store data for 10 years
/etc/pihole/logrotate
'''
/var/log/pihole.log {
	su root root
	daily
	rotate 3650
	copytruncate
	compress
	delaycompress
	notifempty
	nomail
	missingok
	dateext
	dateformat -%Y%m%d
}
'''

test setup with
`sudo /usr/sbin/logrotate -d /etc/pihole/logrotate`