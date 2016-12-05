# AWS Linux Webserver

## Initial Set-Up
- [To Do] Add instructions to remove root remote access
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove
sudo dpkg-reconfigure tzdata
```
- User Prompt
	- Scroll to the bottom of Continents list
	- Select `Etc`; in the second list
	- Select `UTC`

## Add `grader`
```
sudo adduser grader --disabled-password
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

### Provide sudo permissions to connect externally
```
sudo touch /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader
```
- [ ] Paste `grader ALL=(ALL) NOPASSWD:ALL`, Save, Exit

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


## Apache - Install
```
sudo apt-get install apache2
```

- [ ] Confirm page works - default apache page

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
