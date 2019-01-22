## Creating and running containers

### Prerequisites: Install Docker

1. Follow the [official documentation](https://docs.docker.com/install/) to install docker on your platform.

Note: Windows 10 Pro is required, Docker will not work on Windows 10 Home due to Hyper-V requirements.

### Exercise 1: Creating a custom image.

1. Start container using `ubuntu` image and attach to it.
    ```
    docker run -it ubuntu bash
    ```
    This command runs `bash` inside the container.

1. Install `nginx` inside the container.
    ```
    apt-get update
    apt-get install -y nginx
    ```

1. In a separate terminal window list all running containers. Copy `CONTAINER ID` field.
    ```
    docker ps
    ```

1. Commit your changes to a new image. (Replace <conainer-id> with actual container id)
    ```
    docker commit <container-id> my-image
    ```

1. List all images and make sure that `my-image` is on the list.
    ```
    docker images
    ```

1. Exit from the running container.
    ```
    exit
    ```

### Exercise 2: Exposing ports.

1. Run previously created image.
    ```
    docker run -it -p 8080:80 my-image nginx -g 'daemon off;'
    ```
    The arguments of this command have the following meaning:
    * `-it` - attach to the container.
    * `-p 8080:80` - map port `80` in the container to port `8080` on the host system.
    * `my-image` - run image `my-image`
    * `nginx -g 'daemon off;'` - start nginx in foreground mode. Without `daemon off` parameter nginx will start in a background process, and the command finishes immediately. After start command finishes, container will be killed.

1. Open `http://localhost:8080` in your web browser and make sure that nginx is available.

### Exercise 3: Mapping volumes.

1. Run the following command.
    ```
    docker run -it -p 8080:80 -v /tmp/html:/var/www/html my-image nginx -g 'daemon off;'
    ```
    Windows:
    ```
    docker run -it -p 8080:80 -v c:/tmp/html:/var/www/html my-image nginx -g "daemon off;"
    ```

    Here we are mapping `/tmp/html` folder on the host machine to the `/var/www/html` folder inside the container.

1. Save the following file as `index.html` inside `/tmp/html` folder on your local machine.
    ```
    <!DOCTYPE html>
    <html>
    <body>

    <h1>My First Heading</h1>

    <p>My first paragraph.</p>

    </body>
    </html>
    ```

1. Open `http://localhost:8080/` and make sure that the content of the previously created file is displayed.

### Exercise 4 (Optional): Dockerfiles.

Docker can build images automatically by reading instructions from a script. It is generally best practice to use Dockerfiles to easily update, maintain, modify and recreate containers.

In this exercise, we will create a Docker image with Nginx and PHP-FPM 7 using an Ubuntu 16.04 docker image. Additionally, we need Supervisord, so we can start Nginx and PHP-FPM 7 both in one command.

1. Save the following file as `Dockerfile` inside `/tmp/docker-exercise4`
    ```
    #Download base image ubuntu 16.04
    FROM ubuntu:16.04

    # Set the author
    MAINTAINER Firstname Lastname <firstname.lastname@company.com>

    # Set a label
    LABEL com.example.version="1.0.0"

    # Update Software repository
    RUN apt-get update

    # Install nginx, php-fpm and supervisord from ubuntu repository
    RUN apt-get install -y nginx php7.0-fpm supervisor && \
        rm -rf /var/lib/apt/lists/*

    # Define the ENV variable
    ENV nginx_vhost /etc/nginx/sites-available/default
    ENV php_conf /etc/php/7.0/fpm/php.ini
    ENV nginx_conf /etc/nginx/nginx.conf
    ENV supervisor_conf /etc/supervisor/supervisord.conf

    # Enable php-fpm on nginx virtualhost configuration
    COPY default ${nginx_vhost}
    RUN sed -i -e 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' ${php_conf} && \
        echo "\ndaemon off;" >> ${nginx_conf}

    # Copy supervisor configuration
    COPY supervisord.conf ${supervisor_conf}

    RUN mkdir -p /run/php && \
        chown -R www-data:www-data /var/www/html && \
        chown -R www-data:www-data /run/php

    # Volume configuration
    VOLUME ["/etc/nginx/sites-enabled", "/etc/nginx/certs", "/etc/nginx/conf.d", "/var/log/nginx", "/var/www/html"]

    # Configure Services and Port
    COPY start.sh /start.sh
    CMD ["./start.sh"]

    EXPOSE 80 443
    ```

1. Save the following file as `default` inside `/tmp/docker-exercise4`, this is the nginx virtual host file
    ```
    server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        }
    }
    ```

1. Save the following file as `supervisord.conf` inside `/tmp/docker-exercise4`, this is the supervisord configuration file
    ```
    [unix_http_server]
    file=/dev/shm/supervisor.sock   ; (the path to the socket file)

    [supervisord]
    logfile=/var/log/supervisord.log ; (main log file;default $CWD/supervisord.log)
    logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
    logfile_backups=10           ; (num of main logfile rotation backups;default 10)
    loglevel=info                ; (log level;default info; others: debug,warn,trace)
    pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
    nodaemon=false               ; (start in foreground if true;default false)
    minfds=1024                  ; (min. avail startup file descriptors;default 1024)
    minprocs=200                 ; (min. avail process descriptors;default 200)
    user=root             ;

    [rpcinterface:supervisor]
    supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

    [supervisorctl]
    serverurl=unix:///dev/shm/supervisor.sock ; use a unix:// URL  for a unix socket

    [include]
    files = /etc/supervisor/conf.d/*.conf


    [program:php-fpm7.0]
    command=/usr/sbin/php-fpm7.0 -F
    numprocs=1
    autostart=true
    autorestart=true

    [program:nginx]
    command=/usr/sbin/nginx
    numprocs=1
    autostart=true
    autorestart=true
    ```

1. Save the following file as `start.sh` inside `/tmp/docker-exercise4`, this is the script that is run when the container is created from the image
    ```
    #!/bin/sh
    /usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf
    ```

1. Make the startup script executable
    ```
    chmod +x /tmp/docker-exercise4/start.sh
    ```

1. Build and verify the custom docker image was created
    ```
    cd /tmp/docker-exercise4/
    docker build -t custom_nginx_image .
    docker images
    ```

1. Save the following file as `info.php` inside `/tmp/html` folder on your local machine.
    ```
    echo '<?php phpinfo(); ?>' > /webroot/info.php
    ```

1. Run the docker image
    ```
    docker run -it -p 3000:80 -v /tmp/html:/var/www/html --name test custom_nginx_image
    ```
    Windows:
    ```
    docker run -it -p 3000:80 -v c:/tmp/html:/var/www/html --name test custom_nginx_image
    ```

1. Open `http://localhost:3000/info.php` in your web browser and make verify that nginx and php is working.

When creating Dockerfiles there are [several best practices](https://docs.docker.com/v17.09/engine/userguide/eng-image/dockerfile_best-practices/) which should always be followed.

### Exercise 5 (Optional): Docker networking.

1. Find the docker command to list the networks (Hint: Use `docker -h`)
1. Inspect the bridge and host networks of docker
1. Create a new isolated bridge network called `isolated_bridge_network`
1. Inspect both bridge networks, what is the difference between the two?
1. Adopt the previous nginx example to run use the new isolated bridge network
1. Go inside running container (`docker exec -it <container-id> bash`) and find its IP (`ip addr show`)
1. Inspect the docker container (`docker inspect <container-id>`) and find its IP
1. Make sure you understand the difference between [host](https://docs.docker.com/network/host/) and default [bridge](https://docs.docker.com/network/bridge/) networks.

