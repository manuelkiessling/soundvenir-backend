# Soundvenirs Website and Webservice API

[![Build Status](https://travis-ci.org/manuelkiessling/soundvenirs-backend.png?branch=master)](https://travis-ci.org/manuelkiessling/soundvenirs-backend)

## Setting up the production server environment

The following describes the steps you must take in order to create an environment which allows the production website and webservice API to be served from this environment.

### Requirements

* Ubuntu 12.04 64bit
* git
* sqlite3
* php5-fpm
* php5-gd
* php5-sqlite
* composer
* npm >= 1.3.11
* bower
* nginx

### Installing the requirements

    cd
    sudo apt-get install git nginx sqlite3 php5-cli php5-fpm php5-gd php5-sqlite
    wget http://nodejs.org/dist/v0.10.28/node-v0.10.28.tar.gz
    tar xvfz node-v0.10.28.tar.gz
    cd node-v0.10.28
    ./configure
    make
    sudo make install
    sudo npm install -g bower
    curl -sS https://getcomposer.org/installer | php
    sudo ln -s ~/composer.phar /usr/local/bin/composer

### Setting up the environment

Set `listen` to `127.0.0.1:9001` in `/etc/php5/fpm/pool.d/www.conf`.

Create `/etc/nginx/sites-available/soundvenirs.com` with the following content:

    server {
        server_name www.soundvenirs.com soundvenirs.com;
        listen 80;
        access_log /var/log/nginx/soundvenirs.com.access.log;
        error_log /var/log/nginx/soundvenirs.com.error.log;
        charset utf-8;
        client_max_body_size 20M;
    
        root /opt/soundvenirs.com/public;
        index index.php;
    
        location ~ \.php$ {
            fastcgi_pass 127.0.0.1:9001;
            fastcgi_index index.php;
            fastcgi_param PHP_VALUE upload_max_filesize=20M;
            include fastcgi_params;
        }
    
        location / {
            if (-f $request_filename) {
                expires max;
                break;
            }
            rewrite ^(.*) /index.php last;
        }
    }

Then run

    sudo ln -s /etc/nginx/sites-available/soundvenirs.com /etc/nginx/sites-enabled/
    cd /opt
    git clone git@github.com:manuelkiessling/soundvenirs-backend.git ./soundvenirs.com
    cd /opt/soundvenirs.com
    composer install
    bower install
    sudo service php5-fpm restart
    sudo service nginx restart

### Creating the database

Run

    sqlite3 /var/tmp/soundvenirs.production.sqlite

Then, inside the SQLite shell, run

    CREATE TABLE sounds(
       uuid CHAR(36) PRIMARY KEY NOT NULL,
       title TEXT NOT NULL,
       lat FLOAT NULL,
       long FLOAT NULL,
       mp3url TEXT
    );

Now, run

    sudo chown www-data:www-data /var/tmp/soundvenirs.production.sqlite

If you point the A record for *www.soundvenirs.com* at the IP address of the production server, you are now able to access the website at http://www.soundvenirs.com/.
