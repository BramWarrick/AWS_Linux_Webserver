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


## Apache
### Install
```
sudo apt-get update
sudo apt-get install apache2
```

- [ ] Confirm page works - default apache page
	- Navigate to servers IP address, if not sure use command below
	```
	ifconfig eth0 | grep inet | awk '{ print $2 }'
	```


## mod_wsgi - Install and Configure
### Install 
```
sudo apt-get install libapache2-mod-wsgi python-dev
```

### Configure
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```

### Enable
```
sudo a2enmod wsgi
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

## Flask
### Create Flask App
```
cd /var/www
sudo mkdir [Application name]
cd [Application name]
sudo mkdir [Application name]
cd [Application name]
sudo mkdir static templates
```

### Create __init__.py file
```
sudo nano __init__.py
```
- [ ] Add code below to file, save and exit
```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, World!!!"
if __name__ == "__main__":
    app.run()
```

### Install Flask
- [ ] Ensure pip is installed locally
```
sudo apt-get install python-pip

```

- [ ] Ensure virtualenv is installed locally
```
sudo pip install virtualenv
```

- [ ] Create virtual environment
```
sudo virtualenv [Virtual Environment Name]
```

- [ ] Install Flask on virtual environment
```
source [Virtual Environment Name]/bin/activate
sudo pip install Flask
```

- [ ] Confirm install
 - ```
 sudo python __init__.py
 ```
 - Should show “Running on http://localhost:5000/” or "Running on http://127.0.0.1:5000/"

### Virtual Host
#### Create Config File
```
sudo nano /etc/apache2/sites-available/[Application name].conf
```
- [ ] Modify lines of code below to reflect
	- Server Name - change to server's IP Address
	- Server Admin
	- Application name
```
&lt;VirtualHost *:80&gt;  
&nbsp;&nbsp;ServerName [Server Name]  
&nbsp;&nbsp;ServerAdmin [Server Admin]  
&nbsp;&nbsp;WSGIScriptAlias / /var/www/[Application name]/[Application name].wsgi  
&nbsp;&nbsp;&lt;Directory /var/www/[Application name]/[Application name]/&gt;  
&nbsp;&nbsp;&nbsp;Order allow,deny  
&nbsp;&nbsp;&nbsp;Allow from all  
&nbsp;&nbsp;&lt;/Directory&gt;  
&nbsp;&nbsp;Alias /static /var/www/[Application name]/[Application name]/static  
&nbsp;&nbsp;&lt;Directory /var/www/[Application name]/[Application name]/static/&gt;  
&nbsp;&nbsp;&nbsp;Order allow,deny  
&nbsp;&nbsp;&nbsp;Allow from all  
&nbsp;&nbsp;&lt;/Directory&gt;  
&nbsp;&nbsp;ErrorLog ${APACHE_LOG_DIR}/error.log  
&nbsp;&nbsp;LogLevel warn  
&nbsp;&nbsp;CustomLog ${APACHE_LOG_DIR}/access.log combined  
&lt;/VirtualHost&gt;  
```
- [ ] Add modified lines of code to file, save and exit


- [ ] Enable virtual host
```
sudo a2ensite [Application name]
```

### Create .wsgi File
```
cd /var/www/[Application name]  
sudo nano [Application name].wsgi  
```

- [ ] Modify lines below to reflect Application Name
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/[Application name]/")

from [Application name] import app as application
application.secret_key = 'Add your secret key'
```
- [ ] Copy into nano window, save and exit

- [ ] Restart Apache
```
sudo service apache2 restart
```


## Python modules
### SQLAlchemy
```
source [Virtual Environment Name]/bin/activate
sudo pip install SQLAlchemy
```
### Flask
- Installed earlier in process

### oauth2client (google)
```
source [Virtual Environment Name]/bin/activate
sudo apt-get update
sudo apt-get install python-oauth2client
```
