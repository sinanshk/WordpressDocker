
# Host a Wordpress application in Docker using Nginx,PHP-FPM,and MySQL

### Introduction

WordPress is now used by over half of the top one million websites on the internet(Content Management System).WordPress is extremely user-friendly,especially for non-technical users,making it a popular CMS choice.Installing a LAMP (Linux,Apache,MySQL,and PHP)orLEMP (Linux,Nginx,MySQL,andPHP)stack to run WordPress is often time-consuming.You may ease the process of setting up your preferred stack and installing WordPress by using tools like Docker and Docker Compose.A MySQL database,a Nginx web server,and WordPress itself will be included in your containers.

![](https://user-images.githubusercontent.com/123317740/214765856-f1c88183-9f6a-4bb3-bba5-05599e448b45.png)


## To install Docker on an Amazon EC2 instance

Launch an instance with the Amazon Linux 2 AMI.Connect to your instance using SSH.

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
![](https://user-images.githubusercontent.com/123317740/214648235-52991019-c093-4722-b7a0-8d9e5fc1f505.jpg)

#### docker-compose.yml: It’s the configuration file,which we must create when starting a new project with Docker.

#### nginx: This directory contains our additional nginx configuration,such as the virtual host,and so on.

#### db-data: The data directory for mysql. ‘/var/lib/mysql’ data is mounted in the db-data directory.

#### logs: Application log,nginx,mariadb,and php-fpm are all stored in this directory.

#### wordpress: That directory will contain all WordPress files.

create a new nginx configuration file for our wordpress virtual host in the ‘nginx’ directory.
create a new wordpress.conf file:
```
vim nginx/wordpress.conf
```
```
server {
       listen 80;
       server_name wp-hakase.co;
       root /var/www/html;
       index index.php;
       access_log /var/log/nginx/hakase-access.log;
       error_log /var/log/nginx/hakase-error.log;
       location / {
       try_files $uri $uri/ /index.php?$args;
       }
       location ~ \.php$ {
       try_files $uri =404;
       fastcgi_split_path_info ^(.+\.php)(/.+)$;
       fastcgi_pass wordpress:9000;
       fastcgi_index index.php;
       include fastcgi_params;
       fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
       fastcgi_param PATH_INFO $fastcgi_path_info;
       }
}
```

![](https://user-images.githubusercontent.com/123317740/214650003-351c4509-ffaf-4f75-a300-9b45ee6d0455.png)

#### Configure Docker-Compose

first create the docker-compose.yml file
Vim,edit docker-compose.yml:
define our services,starting with Nginx on the first line.using the latest version of the Nginx official docker image.We’ve set up port mapping from port 80 on the container to port 80 on the host.Then,configure the docker volumes for our Nginx virtual host configuration, Nginx log files volume, and the web root directory volume ‘/var/www/html’ next. The WordPress container is linked to the Nginx container.

```
vim docker-compose.yml
```

```
nginx:
    image: nginx:latest
    ports:
        - '80:80'
    volumes:
        - ./nginx:/etc/nginx/conf.d
        - ./logs/nginx:/var/log/nginx
        - ./wordpress:/var/www/html
    links:
        - wordpress
    restart: always
mysql:
    image: mariadb
    ports:
        - '3306:3306'
    volumes:
        - ./db-data:/var/lib/mysql
    environment:
        - MYSQL_ROOT_PASSWORD=user_passwd
    restart: always
wordpress:
    image: wordpress:4.7.1-php7.0-fpm
    ports:
        - '9000:9000'
    volumes:
        - ./wordpress:/var/www/html
    environment:
        - WORDPRESS_DB_NAME=wpdb
        - WORDPRESS_TABLE_PREFIX=wp_
        - WORDPRESS_DB_HOST=mysql
        - WORDPRESS_DB_PASSWORD=user_passwd
    links:
        - mysql
    restart: always
```

![](https://user-images.githubusercontent.com/123317740/214650870-61ca1fc5-b782-4a89-a30d-8494718d0699.png)

(we’ll need to specify the MySQL server.using the most recent MariaDB image.Configure port 3306 for the container,and use the environment variable ‘MYSQL ROOT PASSWORD’ to set the MySQL root password.set up the MySQL data directory’s container volume.Using the WordPress 4.7 docker image with PHP-FPM 7.0 installed,we’ll set up the WordPress service.Set the PHP-fpm port to 9000.Then enable the docker volume for the web directory ‘/var/www/html’ to the host directory ‘wordpress,’.configure the database using the WordPress environment variable, and then link the WordPress service to mysql)

docker-compose configuration is complete.

#### Run Docker-compose

using Docker compose to create new containers.Start new containers based on our compose file in the wordpress-compose directory.

```
cd ~/wordpress-compose/ docker-compose up –d
```

![](https://user-images.githubusercontent.com/123317740/214651885-d6dada45-d221-4033-8636-46497cc9c4b3.jpg)

```
docker-compose ps
```

![](https://user-images.githubusercontent.com/123317740/214652119-7e03863a-0f43-4774-b2cb-bcaab93c8d78.jpg)


#### Install WordPress

check the system’s available ports/open ports before moving on to the next step. Make sure we have three ports open: 80, 3306, and 9000


![](https://user-images.githubusercontent.com/123317740/214653002-1f0171d6-fd0a-4e6f-b384-c53273ca66ef.png)

Now open a web browser and type the server’s URL or IP address into the address bar
Load your domain in web browser and complete the wordpress installation. I'm attaching my completed wordpress site setup.


![](https://user-images.githubusercontent.com/123317740/214654047-ecba442b-27ff-4171-ba29-bd1030e54b0f.jpg)

The WordPress installation page can be viewed.Select the preferred language and click ‘Continue.’

![](https://user-images.githubusercontent.com/123317740/214654157-270e8c31-3d9c-4ce0-9a03-d6db9c269b3c.jpg)





