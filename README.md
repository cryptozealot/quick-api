# Forked from https://github.com/andresharpe/quick-api
# Upgraded php-crud-api to latest version
# Upgraded php-apache image
# Enabled basic authenitication - user:password

# Installation
1. clone the repo
2. "cd quick-api"
3. Edit passwords "nano src/.htpasswd"
4. "docker-compose up -d"
5. open localhost:8081 and input credentials mysql, root, wookie, api
6. create table mytable
7. select an Auto Incrementable column, otherwise it will not work!!!
8. go to localhost/api/records/mytable

# Example request for testing

curl -X POST -H "Content-Type: application/json" -d @server.json -u admin:admin http://192.168.100.2/api/records/mytable



# Thanks to andresharpe!!
# Original readme from forked repo below:

# quick-api
A how-to guide for spinning up a fully functional REST API in 30 minutes (assuming fast connectivity) and with no coding!

Build a REST API supporting GET/POST/DELETE/PUT and basic Swagger documentation using free tools (docker, apache, php, mysql and the very cool mevdschee/php-crud-api script developed by Maurits van der Schee).

## Introduction
So you have a small team. You have a tight deadline to develop a Minimum Viable Product (MVP) or demo - complete with REST API, database server, possibly a responsive web portal and/or a mobile application.

You want to get everyone on the team going as quickly as possible and have a good idea what the models will look like in your freshly minted Model-View-Controller (MVC) design.

Getting the test REST API out there ASAP will get everybody going in parallel and help refine requirements.

## Solution Overview
The solution comprises of
- [Docker](https://docker.com) to help you run a isolated instances/environments of the four main solution components (web server, database and scripting language, documentation server)
- [php-crud-api](https://github.com/mevdschee/php-crud-api) developed by Maurits van der Schee, a really clever, lean and very feature rich single file PHP script that exposes a database as an API
- [PHP/Apache](https://hub.docker.com/_/php/) docker container to host and execute the script
- A [MySQL](https://hub.docker.com/_/mysql/) docker container to define our entities and store them in
- The single file [Adminer](https://hub.docker.com/_/adminer/) script that will allow us to view/modify our MySQL data, running in a Docker container
- A [Swagger](https://hub.docker.com/r/swaggerapi/swagger-ui/) docker container to display the API documentation and test the API that is emitted automatically by the *php-crud-api* script


## The Step-by-step Guide

For this project I am using an Ubuntu Server (16.04.2 LTS with Xubuntu Desktop), but other than the [initial installation of Docker](https://docs.docker.com/engine/installation/), this tutorial should work for you.

#### Step 1: Get Docker
For Debian and Ubuntu based systems you can simply install directly from the Docker website.

```bash
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker andre
```
#### Step 2: Get Docker Compose
I initially tried getting the docker environment setup using only the Docker command line and Dockerfiles, but it is far easier with the composer, so on Ubuntu just type..

```bash
sudo apt-get install docker-compose -y
```
and you're done.
#### Step 3: Create a folder for your project
```
cd ~/projects
```
```
mkdir quick-api
cd quick-api
```
Create a folder to host our website source code (our php/apache docker will point here)
```
mkdir src
```
We will also need to customise the default docker image for php/apache (essentially create a new image deriving from it) and will do so in this folder..
```
mkdir php-mysqli
```
#### Step 4: Create Docker Composer definition
Create a file called *docker-compose.yml* in this folder. It will contain the intructions for Docker Compose to create the three images (mysql, adminer to manage the database and php/apache for hosting the php-crud-api script)
```yaml
# docker-compose.yml

version: '2'

services:

  mysql:
    image: mysql:8.0.2
    environment:
      MYSQL_ROOT_PASSWORD: "wookie"
      MYSQL_DATABASE: "api"
    volumes:
      - mysql_data:/var/lib/mysql

  adminer:
    image: adminer
    ports:
      - 8080:8080
    depends_on:
      - mysql

  php:
    build: ./php-mysqli
    ports:
      - 80:80
    volumes:
      - ./src:/var/www/html
    depends_on:
      - mysql

volumes:
  mysql_data:
```
The *volumes* directive in the mysql section makes sure that your database changes survives an image (or PC) reboot (as Docker containers are [ephemeral](http://www.dictionary.com/browse/ephemeral) and you'll lose all file system  changes inside a Docker container when it is shut down).

In the *php* section the *volumes* directive also creates a map to a subfolder outside the php Docker image (*./src* in our project folder).

#### Step 5: Add mysqli to php-apache
There are quite a few official [php images for Docker](https://hub.docker.com/_/php/), but none of them seem to contain the [*mysqli*](https://en.wikipedia.org/wiki/MySQLi) extension enabled, which the *php-crud-api* script depends on.

Create a file called *Dockerfile* in the *phpmysql* subfolder. Let's add both the inclusion of *mysqli* with PHP and activate the *rewrite* module of Apache whilst we at it (we will need this too).

```
#php-mysqli/Dockerfile

FROM php:7.1.7-apache

RUN apt-get update \
      && apt-get install -y mysql-client libmysqlclient-dev \
      && docker-php-ext-install mysqli \
      && a2enmod rewrite
```

#### Step 6: Build the containers
Just before we get the api script installed, let's test that our containers all work.

(Make sure you are on a fast connection and feel like having a coffee. If not you can offer to make one for your spouse or parents.)

```
docker-compose up -d
```
Docker will launch into an epic and magical excursion to pull the referenced images, cache them, build the php image from our Dockerfile and start up three containers in the background (*-d*).
```
...
Creating quickapi_mysql_1
Creating quickapi_php_1
Creating quickapi_adminer_1

andre@devbox:~/projects/quick-api$
```
To check if your containers are running succesfully type *docker ps*..
```
andre@devbox:~/projects/quick-api$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
d864a9aa2542        adminer             "entrypoint.sh doc..."   6 minutes ago       Up 6 minutes        0.0.0.0:8080->8080/tcp   quickapi_adminer_1
d7b5b6d1dda9        quickapi_php        "docker-php-entryp..."   6 minutes ago       Up 6 minutes        0.0.0.0:80->80/tcp       quickapi_php_1
46f413739138        mysql:8.0.2         "docker-entrypoint..."   6 minutes ago       Up 6 minutes        3306/tcp                 quickapi_mysql_1
andre@devbox:~/projects/quick-api$
```

#### Step 7: Test the stack
A simple way to make sure that most of the new stack is working, is to create a new file in the *src* folder called *index.php* with the following content:
```php
<?php
  // src/index.php
  phpinfo();
?>
```
Then simply open the browser on your development server and browse to http://localhost

You should see - and I quote directly from the [PHP manual](http://php.net/manual/en/function.phpinfo.php) - "a large amount of information about the current state of PHP. This includes information about PHP compilation options and extensions, the PHP version, server information and environment (if compiled as a module), the PHP environment, OS version information, paths, master and local values of configuration options, HTTP headers, and the PHP License."

![phpinfo result](https://user-images.githubusercontent.com/7415999/28838867-fd2dc734-76f1-11e7-9992-77ae491a8e1f.png "phpinfo Results")

#### Step 8: Create a table in MySQL
Next, point your browser to *http://localhost/8080*. You should see the following page.

![screenshot_2017-08-01_19-49-31](https://user-images.githubusercontent.com/7415999/28839088-c8ee461e-76f2-11e7-910c-211870d7565f.png)

As per the screenshot above, set the **System** to *MySQL*, **Server** field to *mysql*, **Username** to *root*, **Password** to *wookie* and **Database** to *api*. Click on *Create table* and create a table called **movies** with the following four fields:

- id, unsigned integer, Autonumber
- name, varchar(128)
- year, unsigned smallint
- director, varchar(64)

The screen should look similar to this:

![screenshot_2017-08-01_19-57-45](https://user-images.githubusercontent.com/7415999/28839509-2e7b0e26-76f4-11e7-8c04-530a8ea0f080.png)

Click on the *Save* button and then on *New item* to create the first record. Leave *id* blank, and enter a movie name, year and director like so:

![screenshot_2017-08-01_20-08-06](https://user-images.githubusercontent.com/7415999/28839863-4829ea30-76f5-11e7-8d28-9ed110a31ec8.png)

#### Step 9: Create the API

The final step is to install the single file php-api-api script and to configure it.

Create an *api* sub folder in *./src*
```
mkdir src/api
cd src/api
```
Then download the *api.php*  [script](https://raw.githubusercontent.com/mevdschee/php-crud-api/master/api.php)
```
wget https://raw.githubusercontent.com/mevdschee/php-crud-api/master/api.php
```
...edit the file and uncomment and change the following few lines (circa line 2711) to point the script to the correct database.

```php
<?php
...
// uncomment the lines below when running in stand-alone mode:

//$api = new PHP_CRUD_API(array(
// 	'dbengine'=>'MySQL',
// 	'hostname'=>'localhost',
// 	'username'=>'ist-test',
//	'password'=>'HowF72yIv0UxPZyS',
// 	'database'=>'ist-data-test02',
// 	'charset'=>'utf8'
//));
//$api->executeCommand();


$api = new PHP_CRUD_API(array(
	'dbengine'=>'MySQL',
	'hostname'=>'mysql',
	'username'=>'root',
	'password'=>'wookie',
	'database'=>'api',
 	'charset'=>'utf8'
));
$api->executeCommand();
```
Save the file and then create a file called *.htaccess* in the same folder (this will allow the apache server inside the php container to 'rewrite' the url in the browser to make **http://localhost/api/api.php/movies** equivalent to **http://localhost/api/movies** which is far cooler)

Have it contain:
```
#src/api/.htaccess

RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ api.php/$1 [L,QSA]
```

#### Step 10: Done! Test your API

Open a browser and try **http://localhost/api/movies/1**. You should see :
```json
{"id":1,"name":"Star Wars: Episode IV - A New Hope","year":1977,"director":"George Lucas"}
```

#### Step 11: Add Swagger documentation and test your API

The php-crud-api script automatically generates [Swagger](https://swagger.io/) documentation, and Swagger-UI can easily be added to the Docker Compose script. Add the *swagger* service just below the *php* service as follows.

```yaml
# docker-compose.yml

version: '2'

services:

...

  swagger:
    image: swaggerapi/swagger-ui
    environment:
      API_URL: "http://localhost/api/api.php"
    ports:
      - 8081:8080
    depends_on:
      - mysql

volumes:
  ...
```
Run the following command to shut down and restart the docker containers.

```
docker-compose down
```
```
docker-compose up -d
```
Finally browse to *localhost:8081* to view full Swagger documentation for your new API.

![screenshot_2017-08-01_22-36-32](https://user-images.githubusercontent.com/7415999/28845969-06986bea-770a-11e7-850c-7bd058e95d7e.png)

You can use Swagger to test creating, deleting, updating and listing entities via you newly minted REST API. You can also add as many tables via *adminer* as you like and the API will automatically be extended with new entities.

Enjoy!
