# AWS Linux Webserver
- IP: 35.165.146.88

### Software Installed
- Apache2
- wsgi client (libapache2-mod-wsgi)
- git
- python-pip
- virtualenv
- Flask
- SQLAlchemy
- Psycopg2
- Python-oauth2client
- Item Catalog (from previous work)

### Configuration changes
#### Security
- Created `grader`
	- Gave `sudo` permissions, password
	- Allowed remote connection
- `root` remote access removed (`rm authorized_keys`)
- Firewall (ufw)
	- Close all inbound traffic
	- Open ports: ssh = 2200/tcp, NTP = 123, web = 80/tcp

#### Maintenance
- Updated and Upgraded software
- Changed timezone to UTC

#### Application support
- Created file structure and symlinks for application and simple version control
	- Symbolic links bounce from /var/www/html/item-catalog to /var/www/item-catalog/live to /var/apps/item-catalog/version/0001
	- Provides greater ease if future versions occurred
- Created virtual environment - installed python libraries there
- `git clone` of repository
	- Included .wsgi file
- Modified Apache2 conf file (.../sites-enabled/0000-default.conf)
	- Added directory information
	- Added alias, mapped to main.wsgi file

### Third Party Resources
- Remove `root` access
	- http://www.tecmint.com/disable-or-enable-ssh-root-login-and-limit-ssh-access-in-linux/
- Create Flask app (with virtual environment)
	- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps  
	- http://www.datasciencebytes.com/bytes/2015/02/24/running-a-flask-app-on-aws-ec2/
- Secure PostGreSQL
	- https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps 