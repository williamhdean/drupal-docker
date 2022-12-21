# Deansal powered Drupal 9 With Composer Installation

This is a sample Drupal 9 with Composer installation pre-configured for use with Deansal.

Features:

- Drupal 9 Composer Project
- PHP
- MySQL

## Setup instructions

### Step #1: Project setup

1. Clone this repo into your Projects directory

    ```
    git clone https://github.com/rtdean93/drupal-docker.git
    cd drupal-docker
    ```

2. Initialize the environment

    This will initialize local settings and install the site via drush

    ```
    docker compose up
    ```

3. Initialize the site

    This will initialize local settings and install the site via drush

    ```
    cd app
    composer install
    ```

4. Point your browser to

    ```
    http://localhost:8080/
    ```

When the automated install is complete the command line output will display the admin username and password.


## Additional notes

For default DB configuration

- host: db
- user: root
- password: ParadiseCity93
- database: drupal


## Original configruation 

We used a great blog post at https://circleci.com/blog/continuous-drupal-p1-maintaining-with-docker-git-composer/ for the guidance here. Below is an abbreviated installation process:

### Setting Up The Project Directory

To start, we need to create a root directory to hold our entire project. This directory will be versioned with Git.

    ```
    mkdir -p ~/Projects/my-drupal
    cd ~/Projects/my-drupal
    ```

### Create The Dockerfile

Now there’s a few things we need to create before we can start installing Drupal. The first is a Dockerfile that will contain our actual Drupal website. You can copy & paste the following Dockerfile into ./Dockerfile, which is the root of our project.

    ```
    FROM drupal:9.4.9-php8.1-apache

    RUN apt-get update && apt-get install -y \
        curl \
        git \
        mariadb-client \
        vim \
        wget

    RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
        php composer-setup.php && \
        mv composer.phar /usr/local/bin/composer && \
        php -r "unlink('composer-setup.php');"

    RUN wget -O drush.phar https://github.com/drush-ops/drush-launcher/releases/download/0.4.2/drush.phar && \
        chmod +x drush.phar && \
        mv drush.phar /usr/local/bin/drush

    RUN rm -rf /var/www/html/*

    COPY apache-drupal.conf /etc/apache2/sites-enabled/000-default.conf

    WORKDIR /app
    ```


### Create Apache VHost Config

That Apache VirtualHost config file we just mentioned, let’s create it. This file should be located at ./apache-drupal.conf.

    ```
    <VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /app/web

        <Directory /app/web>
            AllowOverride All
            Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

    </VirtualHost>
    # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
    ```

### Create The Docker Compose File

Our Drupal site is going to be composed of two Docker images. The main image which holds the Drupal specific stuff, which we designated with the Dockerfile we made, and a database image. Docker Compose will allow use to create containers from these two images and have them connected to each other so that they “just work”.

The Docker Compose file should also be created at the root of our project, ./docker-compose.yml.

    ```
    version: '3'
    services:
      db:
        image: mariadb:10.9
        environment:
          MYSQL_DATABASE: drupal
          MYSQL_ROOT_PASSWORD: drupal
        volumes:
          - db_data:/var/lib/mysql
        restart: always
      drupal:
        depends_on:
          - db
        build: .
        ports:
          - "8080:80"
        volumes:
          - ./app:/app
        restart: always
    volumes:
      db_data:
    ```

### Save Our Progress With Git

    ```
    git init .
    git add .
    git commit -m "Initial commit."
    ```

### Building The Drupal Website

With the initial scaffolding out of the way, we can go ahead and get our actual site built. We start with bringing up our containers with Docker Compose.

    ```
    docker-compose up -d --build
    ```

--build tells Docker Compose to build our Dockerfile fresh when we run this command. We need to login to our main container to run Composer, but we need the container name to do so. We can find it by running docker-compose ps. The container connected to port 80 is the correct one. Here’s an example of the output:

$ docker-compose ps

         Name                       Command               State                  Ports
------------------------------------------------------------------------------------------------------
drupal-docker_db_1       docker-entrypoint.sh mariadbd    Up      3306/tcp
drupal-docker_drupal_1   docker-php-entrypoint apac ...   Up      0.0.0.0:8080->80/tcp,:::8080->80/tcp

The correct container name here is drupal-docker_drupal_1. We log into it using the docker exec command.

docker exec -it drupal-docker_drupal_1 bash

This will drop you into the container at the /app directory. Now we can use composer to install Drupal.

    ```
    /app #  composer create-project drupal/recommended-project /app --stability dev --no-interaction
    /app #  mkdir -p /app/config/sync
    /app #  chown -R www-data:www-data /app/web
    ```

### Instal Drupal 

Use the command line below to install drupal or visit http://localhost:8080 in your browser and go through the Drupal installer.

    ```
    drush si --db-url=mysql://root:drupal@db/drupal
    ```

#### See additional steps from Ricardo N Feliciano at https://circleci.com/blog/continuous-drupal-p1-maintaining-with-docker-git-composer/ 