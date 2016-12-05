# AWS Linux Webserver

## Initial Set-Up
- [To Do] Add instructions to remove root remote access
```
sudo apt-get update
sudo apt-get dist-upgrade
sudo apt-get autoremove
sudo apt-get autoclean
sudo dpkg-reconfigure tzdata
```
- User Prompt
	- Scroll to the bottom of Continents list
	- Select `Etc`; in the second list
	- Select `UTC`

## Add `grader` and give sudo permissions
```
sudo adduser grader sudo --disabled-password
```

- [ ] Answer questions and enter password

### Provide permissions to connect externally
```
sudo su - grader
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
nano ~/.ssh/authorized_keys
```

- [To Do] Add instructions for authorized keys

### Provide sudo permissions - back up plan
```
sudo usermod -a -G sudo grader
```


## SSH Port change
```
sudo nano /etc/ssh/sshd_config
```
- [ ] Find `#port 22` and change to `port 2200`
	- Note removal of `#` from the line

## Firewall
```
sudo ufw status
```

- [ ] Confirm status - initial state should be disabled

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow ntp
sudo ufw allow www
sudo ufw enable
sudo ufw status
```
- [ ] Confirm status, should now be enabled with details that align with configuration

## LAMP

### Apache
#### Install
```
sudo apt-get update
sudo apt-get install apache2
```

- [ ] Confirm page works - default apache page
	- Navigate to servers IP address, if not sure use command below
	```
	ifconfig eth0 | grep inet | awk '{ print $2 }'
	```

### MySQL

#### Install and Activate
```
sudo apt-get install mysql-server libapache2-mod-auth-mysql php5-mysql
sudo mysql_install_db

```

#### Set-Up
```
sudo /usr/bin/mysql_secure_installation
```
- Expected prompts with desired answers:
```
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

### PHP
#### Install
```
sudo apt-get install php5 libapache2-mod-php5 php5-mcrypt
```

#### Configuration
```
sudo nano /etc/apache2/mods-enabled/dir.conf
```
- [ ] Add `index.php` to beginning of index files list.

#### PHP Module installation
```
sudo apt-get install php5-mysql
```


### Post install verification
```
sudo nano /var/www/info.php
```
- [ ] Paste in
```
<?php
phpinfo();
?>
```
- [ ] Save and Exit

- [ ] Restart Apache using
```
sudo service apache2 restart
```

- [ ] Navigate to `http://[your IP here]/info.php`
	- If everything works you should see the default PHP page
	- Otherwise, review install instructions and begin debugging


## mod_wsgi - Install and Configure
### Install 
```
sudo apt-get install libapache2-mod-wsgi
```

### Configure
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```

- Add 
```
WSGIScriptAlias / /var/www/html/myapp.wsgi
```
to end of `<VirtualHost>` block

- Restart Apache
```
sudo apache2ctl restart
```
- [ ] Confirm page works - no errors

## PostGreSQL - Install
### Install
```
sudo apt-get install postgresql
```
- [ ] Confirm page works - no errors

## Python modules
### SQLAlchemy
```
sudo pip install SQLAlchemy
```

### Flask
```
sudo pip install Flask
```

### oauth2client (google)
```
sudo apt-get update
sudo apt-get install python-oauth2client
```
