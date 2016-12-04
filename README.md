vagrant up
vagrant ssh

-------------------
## Initial Set-Up
` Add instructions to remove root remote access
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove

sudo apt-get install finger```

## Add `grader`
```
sudo adduser grader```

- Answer questions and enter password

```
finger grader
sudo su - grader
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
nano ~/.ssh/authorized_keys


sudo cp /etc/sudoers.d/vagrant /etc/sudoers.d/grader
sudo nano /etc/sudoers/d/grader
```

- Modify reference to user

## Firewall
- Add steps to shut down extraneous access


## Install Apache
```
sudo apt-get install apache2```

- Confirm page works - default apache page

## mod_wsgi
### Install 
```
sudo apt-get install libapache2-mod-wsgi```

### Configure
```
sudo nano /etc/apache2/sites-enabled/000-default.conf```

Add 
```
WSGIScriptAlias / /var/www/html/myapp.wsgi```
to end of `<VirtualHost>` block

---- Restart Apache ----
```
sudo apache2ctl restart```


## PostGreSQL
### Install
```
sudo apt-get install postgresql```


## Python modules
- SQLAlchemy
```
sudo pip install SQLAlchemy```

- Flask
```
sudo pip install Flask```

- oauth2client (google)
```
sudo apt-get update
sudo apt-get install python-oauth2client```# AWS_Linux_Webserver
