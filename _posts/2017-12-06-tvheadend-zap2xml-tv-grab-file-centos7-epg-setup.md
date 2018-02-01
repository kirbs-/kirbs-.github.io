Tvheadend, Apache reverse proxy, tv_grab_file, zap2it EPG setup on Centos 7

1. Download tv_grab_file ans save to /usr/local/bin
2. sudo chmod +x /usr/local/bin/tv_grab_file
3. Download zap2xml.pl from http://zap2xml.awardspace.info/
4. install perl dependencies
'''
sudo yum install perl-CPAN perl-Compress-Zlib perl-HTML-Parser perl-HTTP-Cookies perl-LWP-Protocol-https perl-JSON gcc cpan
cpan JSON::XS
cpan Bundle::CPAN
cpan Sub::Util
'''
5. Create zap2it account and set to locale.
6. Set cron 
'''
0 3 * * * /usr/local/bin/zap2xml.pl -u <login> -p <password>
'''

7. zap2xml.pl -u <login> -p <password> # to pull initial EPG
8. run find_tv_grabbers
9. create tvheadend.service in /etc/systemd/system/
'''

'''
10. start tvheadend
`sudo systemctl start tvheadend`
11. set to load on boot
`sudo systemctl enable tvheadend`
11. setup reverse proxy
13. Open firewall ports
```
sudo firewall-cmd --add-port=9982/tcp --permanent --zone=public
sudo firewall-cmd --add-port=9981/tcp --permanent --zone=public
sudo firewall-cmd --reload
```