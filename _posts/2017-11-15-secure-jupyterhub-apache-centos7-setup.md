---
layout: page
title: Secure Jupyterhub setup with Centos 7 and Apache
order: 1
---
1. mkvirtualenv jupyter
2. workon jupyter
3. pip install jupyterhub
4. npm install -g configurable-http-proxy
5. pip install notebook
6. jupyterhub --generate-config
7. mkdir /etc/jupyterhub
8. chown <user> /etc/jupyterhub
9. mv jupyterhub_config.py /etc/jupyterhub/juypterhub_config.py 
10. which juypterhub # copy this to ExecStart
11. create systemd jupyterhub.service file
```
[Unit]
Description=Jupyterhub
After=network-online.target

[Service]
#replace user with an existing system user. Should be the same user from step 8.
User=user
# first part of ExecStart should point to the jupyterhub executable in the jupyter virtualenv.
ExecStart=/usr/local/.virtualenvs/jupyter/bin/jupyterhub -f /etc/jupyterhub/jupyterhub_config.py
# append jupyterhub's parent folder to the end of Environment.
Environment="PATH=/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/.virtualenvs/jupyter/bin"

WorkingDirectory=/etc/jupyterhub

[Install]
WantedBy=multi-user.target
```
12. sudo systemctl enable juypterhub
13. sudo systemctl start juypterhub
14. Add Apache virtual host and configure SSL. This assumes you already have SSL certificates. Otherwise use OpenSSL or Let' Encrypt.
```
<VirtualHost *:80>
  ServerName jupyter.example.com
  Redirect / https://jupyter.example.com
</VirtualHost>
<VirtualHost *:443>
  ServerName jupyter.example.com
  SSLEngine On
  SSLCertificateFile /path/to/certificate_file.crt
  SSLCertificateKeyFile /path/to/certificate_file.key
  SSLCACertificateFile /path/to/ca-chain.crt
  ProxyPreserveHost On
  ProxyRequests Off
  ProxyPass / http://127.0.0.1:8000/
  ProxyPassReverse / http://127.0.0.1:8000/
  <Location ~ "/(user/[^/]*)/(api/kernels/[^/]+/channels|terminals/websocket)/?">
    ProxyPass ws://localhost:8000
    ProxyPassReverse ws://localhost:8000
  </Location>
</VirtualHost>
```
15. sudo systemctl restart httpd