# First Docker Image Get!

### Platform

##### Ubuntu 19.04.1 with VMWare Workstation 16 Pro 

### Steps

1, Install Docker using `sudo apt-get install docker.io`

2, Install Docker Compose using `sudo apt-get install docker-compose`

3, Create a New Directory using `mkdir DockerStuff`

4, Enter that new Directory we just created using `cd DockerStuff`

5,Create a Docker File using `sudo vim Dockerfile`
```
FROM ubuntu:14.04


RUN apt-get update && apt-get install -y apache2 libapache2-mod-php5 zip

COPY www/ /var/www/html/

EXPOSE 80

ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

6, Create a New Directory in the Folder called www using `mkdir www`.

7, Create and edit a html file using `vim index.html`.
```
<!DOCTYPE html>
<html lang="en">
  <head>
  </head>
  <body>
	123213123123123123123123123
	yee123213123123123123123123123
	123213123123123123123123123
  </body>
</html>
```
8, Build the Docker Image using `docker build -t dockerimagename`.

P.S Remember to change dockerimagename to any name you like!

9,  Run the Docker you just Create using `docker run -p port:80 -d dockerimagename`

P.S. Remember to change dockerimagename to the name you previous entered and change port to port you'd like Docker to run on.

10, Head into browser and enter `localhost:port` to see if it's working!

![alt text](https://github.com/LunarstarPony/Lunarstar-Project/blob/main/First%20Docker%20Image/1.png?raw=true)

11, Success!!!!
