# dolibarr

* See https://www.microlinux.fr/dolibarr-centos-7

## Install epel

```bash
yum install epel-release yum-utils -y
```

## Generate the certificate for https://gestion.mydomain.com

* yum install openssl
* See https://github.com/davidboukari/ssl


## Install apache2

```bash
# /etc/httpd/conf.d/10-gestion.mydomain.com.conf

# http://gestion.mydomain.com -> https://gestion.mydomain.com
<VirtualHost *:80>
  ServerName gestion.mydomain.com
  Redirect / https://gestion.mydomain.com
</VirtualHost>

# https://gestion.mydomain.com
<VirtualHost _default_:443>
  Header always set Strict-Transport-Security \
    "max-age=63072000; includeSubDomains"
  ServerAdmin admin@gestion.mydomain.com
  DocumentRoot "/var/www/html/gestion.mydomain.com/htdocs"
  ServerName gestion.mydomain.com:443
  #SSLEngine on
  SSLCertificateFile /etc/ssl/certs/gestion.mydomain.com.crt
  SSLCertificateKeyFile /etc/ssl/private/gestion.mydomain.com.key
  #SSLCertificateChainFile /etc/letsencrypt/live/sd-100246.dedibox.fr/fullchain.pem
  BrowserMatch "MSIE [2-5]" \
    nokeepalive ssl-unclean-shutdown \
    downgrade-1.0 force-response-1.0
  ErrorLog logs/gestion.mydomain.com-error_log
  CustomLog logs/gestion.mydomain.com-access_log common
</VirtualHost>
```

## Install php

```bash
yum-config-manager --enable remi-php72
yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
yum-config-manager --enable remi-php74
yum install php php-common php-opcache php-mcrypt php-cli php-gd php-curl php-mysql php-zipstream php-zip
yum install unzip
```
`

## selinux

sudo setsebool -P httpd_can_sendmail on

## install mysql

```bash
yum install mariadb-server mariadb-client
systemctl start mariadb
mysql_secure_installation
```

### Install the database

```bash
mysql -u root -p
create database `dolibarr`;
grant all on `dolibarr`.* to root@localhost identified by 'xxxx';
flush privileges;
quit
```

## Install dolibar

```bash
yum install git
cd /var/www/htdocs
git clone https://github.com/Dolibarr/dolibarr.git gestion.mydomain.com
cd gestion.mydomain.com
mkdir documents
chown -R apache.apache documents
chmod 770 documents
touch documents/install.lock
chown -R microlinux:microlinux slackbox-dolibarr/
find slackbox-dolibarr/ -type d -exec chmod 0755 {} \;
find slackbox-dolibarr/ -type f -exec chmod 0644 {} \;
touch htdocs/conf/conf.php
chown microlinux:apache htdocs/conf/conf.php
chmod 0660 htdocs/conf/conf.php
```

To install just go to the url:

* https://gestion.mydomain.com/index.php

## Security

* https://wiki.dolibarr.org/index.php/Informations_s%C3%A9curit%C3%A9

### fail2ban

```
[INCLUDES]
before = common.conf

[Definition]
#2020-03-21 07:00:22 DEBUG   37.166.184.140  Bad password, connexion refused
failregex= ^.* <HOST> .*Authentication KO.*$
#datepattern = ^%%Y-%%m-%%d %%H:%%M:%%S
ignoreregex =
```

```bash
#Dolibarr
[dolibarr]
enabled  = true
filter   = dolibarr
port="http,https"
logpath  = /var/www/html/gestion.mydomain.com/documents/dolibarr.log
maxretry = 3
#findtime = 600
findtime = 32000000
```

To test:

```bash
fail2ban-regex --print-all-matched /var/www/html/gestion.mydomain.com/documents/dolibarr.log /etc/fail2ban/filter.d/dolibarr.conf
```
