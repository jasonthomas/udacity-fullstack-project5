# P5 - Linux Server Configuration

## IP Address and SSH Port
52.35.141.102 Port 2200

## Hosted Catalog Application
http://52.35.141.102/

## Configuration Changes and Software Installation

### Add user

useradd -d /home/grader -s /bin/bash grader

### Give grader user sudo

Add the following to /etc/sudoers.d/local
```
grader    ALL=(ALL)    NOPASSWD: ALL
```

### Change SSH Port to 2200

Edit /etc/ssh/sshd_config and change 'Port' Port 2200
Restart ssh service
```
service ssh restart
```
use `netstat -lnp` to see if ssh is running on specified port


### Enable and configure UFW
UFW has a default DENY policy, so you should make sure that you have the correct rules in place.
```
ufw allow 2200/tcp
ufw allow 80/tcp
ufw allow 123/tcp
ufw allow 123/udp /etc/services
ufw enable
```

### Set time to UTC
clock is already configured to UTC, but I did the following anyway

```
unlink /etc/localtime
ln -s /usr/share/zoneinfo/UTC /etc/localtime
```

### Install packages for catalog app
apt-get install apache2 libapache2-mod-wsgi
apt-get install postgresql-9.3
apt-get install python-flask python-psycopg2 python-sqlalchemy python-oauth2client

### Configure postgres to listen on localhost only
Edit /etc/postgresql/9.3/main/postgresql.conf and make listen_addresses = 'localhost', although this is the default configuration

### Add catalog user to postgres

```
su - postgres
createuser -W catalog
psql < <(echo 'create database catalogapp;')
psql < <(echo "GRANT ALL PRIVILEGES ON DATABASE catalogapp to catalog;")

```

### Create catalog directory, and catalog user. Extract catalog content
Download catalog app, and extract into /data/catalog

```
mkdir /data/catalog
unzip -d /data/catalog catalog.app.zip


### Create a catalog user for usw with apache and mod_wsgi
```
useradd -d /data/catalog -s /bin/bash catalog
```

### Configure Apache HTTPD with mod_wsgi
Normally I wouldn't use _default_, but I want this to be default VirtualHost.

Create file /etc/apache2/sites-available/catalog.conf

```
<VirtualHost _default_:*>

    WSGIDaemonProcess catalog user=catalog group=catalog threads=5
    WSGIScriptAlias / /data/catalog/app.wsgi

    <Directory /data/catalog>
        WSGIProcessGroup catalog
        WSGIApplicationGroup %{GLOBAL}
    Options All
    AllowOverride All
    Require all granted
    </Directory>
</VirtualHost>
```
Symlink /etc/apache2/sites-available/catalog.conf to /etc/apache2/sites-enabled/catalog.conf
```
ln -s /etc/apache2/sites-available/catalog.conf /etc/apache2/sites-enabled/catalog.conf
```

Restart Apache HTTPD
```
service apache2 restart
```
