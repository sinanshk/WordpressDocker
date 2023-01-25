# WordpressDocker
# Host a Wordpress application in Docker using Nginx,PHP-FPM,and MySQL
### Introduction
WordPress is now used by over half of the top one million websites on the internet (Content Management System). WordPress is extremely user-friendly, especially for non-technical users, making it a popular CMS choice. Installing a LAMP (Linux, Apache, MySQL, and PHP) or LEMP (Linux, Nginx, MySQL, and PHP) stack to run WordPress is often time-consuming. You may ease the process of setting up your preferred stack and installing WordPress by using tools like Docker and Docker Compose.
You'll learn how to set up a multi-container WordPress installation in this article. A MySQL database, a Nginx web server, and WordPress itself will be included in your containers. You can additionally safeguard your installation by using Let's Encrypt to get TLS/SSL certificates for the domain you want to use with your site.
![](https://user-images.githubusercontent.com/65948438/162495574-7652e457-f7a7-43fc-bd7d-4e9921e60664.png)
## To install Docker on an Amazon EC2 instance
Launch an instance with the Amazon Linux 2 AMI.see Launching an instance in the Amazon EC2 User Guide for Linux Instances.
Connect to your instance using SSH. For more information, see Connect to your Linux instance using SSH in the Amazon EC2 User Guide for Linux Instances.
Update the installed packages and package cache on your instance.
```
sudo yum update –y
```
Install the most recent Docker Engine package
```
sudo amazon-linux-extras install docker
```
Start and enable the Docker service
```
sudo service docker start
sudo systemctl enable docker
```
### Install Docker-Compose
Docker-compose is available in the PyPI python repository because it is a python script.python pip can be used to install it.we must first install Python and Python Pip on our system.
Firstly, Install python and python-pip by using these commands:
```
sudo yum install -y python python-pip
```
using the pip command, install docker-compose
```
pip install docker-compose
```
use the docker-compose command to verify the installation
```
docker-compose –v
```
### Setup WordPress
The system now has docker and docker-compose installed.in this step,we’ll create and configure a Docker-compose environment for our WordPress project.The ‘WordPress’ PHP application will be deployed as docker containers managed by docker-compose,with Nginx as the web server and MariaDB for the MySQL database.Each application will run in its own container,
##### -nginx: We use the ‘nginx: latest’ official docker image
##### -WordPress: On docker-hub, WordPress provides some docker images. We’ll use latest wordpress with latest PHP-FPM on it
##### -MySQL : use the most recent version of MariaDB’s official container
we require three docker images. We won’t run docker as root;we’ll use a regular Linux user.
So simply to create a new user
```
useradd sinan
passwd sinan
```
Add the user to the ‘docker’ group so that he or he can use the docker command.Then restart the docker service:
```
usermod -a -G docker sinan systemctl restart docker
```
Log in as the ‘sinan’ user for the WordPress project, make new directory:
```
su - sinan mkdir -p wordpress-compose cd wordpress-compose/
```
create a new directory for the project and a new file called ‘docker-compose.yml’.
```
touch docker-compose.yml
mkdir -p nginx/
mkdir -p db-data/
mkdir -p logs/nginx/
mkdir -p wordpress/
```






