# CTFd installation!

### Platform

##### Ubuntu 19.04.1 with VMWare Workstation 16 Pro 

### Steps

1, We ganna update some files using command `sudo apt-get update`.

2, Install Python3 pip using command `sudo apt-get install python3-pip`.

3, Install vim using `sudo apt-get install vim`.

4, Install Web Framework Flask using `sudo python3 -m pip install Flask`.

5, Clone CTFd from GitHub using `git clone https://github.com/CTFd/CTFd.git`.

6, Get into directory which we just created using `cd CTFd`.

7, Modify Prepare file using `sudo vim prepare.sh`.
```
#!/bin/sh
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y build-essential python-dev python-pip libffi-dev
pip3 install -r requirements.txt
```
8, Install it using `sudo ./prepare.sh`.

9, Install Requirement using following commands.
```
sudo apt install mysql-server
sudo apt install mariadb-server
sudo apt-get install libmariadbclient-dev  for MariaDB DataBases
sudo apt install libmysqlclient-dev  for MySQL DataBases
sudo python3 -m pip install pymysql
```
10, Modify MySQL config files using `sudo vim /etc/mysql/my.cnf` and add these following lines at the end.
```
[mysqld]
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_bin
```
11, Restart mySQL using `sudo systemctl restart mysql`

12, Enter MySQL command interface using `sudo mysql -u root -p` and enter your root password.

13, Create DataBase using `create database ctfd;`.

14, Create a new DataBase user and grand all permission using `grant all privileges on *.* to 'EnterYourUsername'@'%' identified by 'EnterYourPassword';` .

P.S Remember to change EnterYourUsername and EnterYourPassword to your own DataBase Username and Password.

15, We're using this command to check if our database are using UTF-8 format`SHOW VARIABLES LIKE 'character\_set\_%';`.
```
+--------------------------+--------+
| Variable_name            | Value  |
+--------------------------+--------+
| character_set_client     | utf8   |
| character_set_connection | utf8   |
| character_set_database   | utf8   |
| character_set_filesystem | binary |
| character_set_results    | utf8   |
| character_set_server     | utf8   |
| character_set_system     | utf8   |
+--------------------------+--------+
7 rows in set (0.00 sec)
```
16, Modify MySQL Address and Port using `sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`.
```
bind-address            = 0.0.0.0
port            = 3305
```

17, Modify CTFd Database info using `sudo vim CTFd/config.py`.


P.S Remember to change you File Location according to your system setup and also make sure your in CTFd Directory.
```
DATABASE_URL = 'mysql+pymysql://EnterYourUsername:EnterYourPassword@localhost:3305/ctfd'
SQLALCHEMY_POOL_RECYCLE = 5
SQLALCHEMY_POOL_SIZE = 1000
SQLALCHEMY_MAX_OVERFLOW = 1000
SQLALCHEMY_POOL_TIMEOUT = 10
```
18, Set upper limit allowed for a listen() backlog to 4096 using `sudo vim /etc/sysctl.conf` and add this following line.
```
net.core.somaxconn=4096
```
19, Open Limits.conf using `sudo vim /etc/security/limits.conf` and add these following line
```
* soft nofile 65536
* hard nofile 65536
root hard nofile 65536
root soft nofile 65536
```	
20, Add these following config in mysql settings using `sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`
```
max_connections         = 50000
max_connect_errors      = 50000
```
21, Modify mysql service file using `sudo vim /lib/systemd/system/mysql.service`
```
LimitNOFILE=65535
```
22, Reboot the System using `sudo reboot`

23, Install Nginx and uWSGI using these following commands
```
sudo apt install nginx
sudo pip3 install uwsgi
sudo apt install uwsgi
```
24, Copy a Backup for the file we're going to modify later using `sudo cp wsgi.py wsgi_old.py`

25, Replace with following in file using `sudo vim wsgi.py`
P.S Remember to edit FilePath and also make sure your in CTFd Directory!!!
```
import os
import sys
#Detect if we're running via `flask run` and don't monkey patch
#if not os.getenv("FLASK_RUN_FROM_CLI"):
#from gevent import monkey

#monkey.patch_all()
sys.path.insert(0,'/FilePathHere/CTFd')

from CTFd import create_app

app = create_app()
```

26, We ganna create and edit Web Config which is locate in /etc/uwsgi/apps-available
```
cd /etc/uwsgi/apps-available
```
27, Create and edit uwsgi.ini using `sudo vim uwsgi.ini`

P.S: Remember to change CTF Directroy according to your config
```
[uwsgi]
# Where you've put CTFD
#http=127.0.0.1:65535
chdir = /FilePathHere/CTFd/

mount = /="CTFd:create_app()"

plugins-dir = /usr/lib/uwsgi/
plugin = /usr/lib/uwsgi/plugins/python3_plugin.so
socket= /tmp/uwsgi.sock
chmod-socket = 666
master=true
processes = 10
threads = 20
max-requests = 100
enable-threads = true
vacuum = true
mod-socket = 666
manage-script-name = true
wsgi-file = wsgi.py
callable = app

limit-as = 6144
limit-post = 104857600
reload-on-as = 1024
reload-on-rss = 1024
#evil-reload-on-as = 2048
#evil-reload-on-rss = 1024
#max-requests = 1000
#max-worker-lifetime = 600
#reload-on-rss = 6144
#worker-reload-mercy = 600

harakiri = 60

#die-on-term = true


#socket=/tmp/uwsgi.sock
uid = www-data
gid = www-data
daemonize=/var/log/uwsgi/ctfd.log
```
28, Create Symbolic Link to app-enable using `sudo ln -s /etc/uwsgi/apps-available/uwsgi.ini /etc/uwsgi/apps-enabled/uwsgi.ini`

29, Install uwsgi Python Plugins using `sudo apt-get install -y uwsgi-plugin-python3`

30, Start our CTFd using `sudo uwsgi -d --ini /etc/uwsgi/apps-enabled/uwsgi.ini`

31, Change the owner of our CTFd file to www-data so Nginx can access it using `sudo chown -R www-data:www-data CTFd/`

32, Replace all stuff in Nginx WebPage Settings using `sudo vim /etc/nginx/sites-available/default`

P.S Also remember to change file location
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;
	root /yourfileLocation/CTFd;
	index index.html index.htm index.nginx-debian.html;

        server_name 0.0.0.0;

	location / {
             #try_files $uri $uri/ =404;
             root /yourfileLocation/CTFd;
             include uwsgi_params;
             uwsgi_pass unix:/tmp/uwsgi.sock;
             uwsgi_read_timeout 300s;
        }
	location /static {
               root /yourfileLocation/CTFd/CTFd/themes/core/static/;
        }
	client_max_body_size 1000m;
	}
```
33, Restart Nginx using `sudo systemctl restart nginx`

34, Restart or CTFd using following command
```
sudo pkill uwsgi -9
sudo uwsgi -d --ini /etc/uwsgi/apps-enabled/uwsgi.ini
```
35, You should able to access CTFd on Browser using localhost:80 now 

36, Success~~~~

![alt text](https://github.com/LunarstarPony/Lunarstar-Project/blob/main/CTFd%20Installation/1.png?raw=true)

P.S You can Create a Scripts to start and stop CTFd

##### Script for starting CTFd
```
sudo uwsgi -d --ini /etc/uwsgi/apps-enabled/uwsgi.ini
```
##### Script for stoping CTFd
```
sudo pkill uwsgi -9
```
##### Script for restarting CTFd
```
sudo pkill uwsgi -9
sudo uwsgi -d --ini /etc/uwsgi/apps-enabled/uwsgi.ini
```




    
### Some magical stuff!

harakiri =60: If 60 seconds idle then kick
