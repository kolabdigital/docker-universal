# Universal docker-compose development environment setup for running miscellaneous PHP projects


## Prerequisites

You need to have docker with docker-compose installed on your system. You can do it with installer available in here <a href="https://www.docker.com/products/docker-toolbox" target="_blank">https://www.docker.com/products/docker-toolbox</a>. However I do not recommend to do it this way because it mounts shared volumes using vboxsf mode which is very slow. As an alternative you can use great Vagrant box available in here <a href="https://github.com/blinkreaction/boot2docker-vagrant" target="_blank">>https://github.com/blinkreaction/boot2docker-vagrant</a>

## Installation

Clone this repository:

```bash
$ git clone git@github.com:kolabdigital/docker-universal.git
```

Then enter project folder and run:

```bash
$ docker-compose up
```

If you use vagrant box visit this ip 192.168.10.10

If you are using docker-machine check machine ip with command
```bash
$ docker-machine ip default
```
If everything is set up correctly you should see phpinfo() output


## What containers are included

### nginx
Nginx us used as load balancer/reverse proxy server. Whole www directory is mounted to it.

You can access Nginx logs in the following directories on your host machine:
* `logs/nginx`

In order to run multiple websites you need to add individual vhost for each one of them inside hosts file. For example on Mac edit `/etc/hosts` and add.

```
192.168.10.10  symfony.dev
192.168.10.10  drupal7.dev
```

Each time you want to run new website you need to add new server block inside `sites-enabled` directory. Sample configuration files are provided inside `site-examples` directory. Things to modify inside nginx server block configuration

```
server_name  drupal7.dev; # One of vhost added to /etc/hosts file. Needs to be unique for each project
root /var/www/drupal7; # Path to website inside www directory
```

If you are using custom php-fpm container you also need change fastcgi_pass for example this is default one
```
fastcgi_pass php:9000;
```

If your PHP container name is `my-php` fastcgi_pass changes to this

```
fastcgi_pass my-php:9000;
```

### php
Default PHP-FPM container. If you don't need custom PHP version/settings you can use this container to run all your apps.

If for some projects you want custom php settings clone this container, php.ini is available in `conf` folder. If you need additional Linux packages install them inside Dockerfile.

After you cloned php container you can modify docker-compose.yml for example

```bash
myphp:
    build: my-php-fpm
    ports:
        - 9002:9000 # Remember to have unique host port number
    links:
      - mysql:mysql
    volumes:
        - ./www:/var/www

nginx:
    build: nginx
    ports:
        - 80:80
    links:
        - php
        - symfony
        - myphp # After new container is created you need to link it to nginx
    volumes_from:
        - symfony
    volumes:
        - ./www:/var/www
        - ./sites-enabled:/etc/nginx/sites-enabled
        - ./logs/nginx/:/var/log/nginx
```

### mysql

MySQL database container. Mysql container has fallowing shared volumes:
 * `dbstorage` : Prevent from data loose when destroying container,
 * `db` : You can place databases on your host and then import them from inside container using this command

```bash
$ docker exec -it dockeruniversal_mysql_1 /bin/bash
$ mysql -u root -p password database_name < database_file.sql
```

### symfony
You can use this container to run symfony2 or symfony3 apps.
Used as example and can be disabled.

### phpmyadmin
Used to access mysql data with phpmyadmin on port `8181` for example 192.168.10.10:8181
You can disable it if you want.

