## Creating and running containers

### Prerequisites: Install Docker

1. Follow the [official documentation](https://docs.docker.com/install/) to install docker on your platform.

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
    Here we are mapping `/tmp/html` folder on the host machine to the `/var/www/html` folder inside the container.

1. Save the following file as `index.html` inside `/tmp/html` folder.
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

### Exercise 4 (Optional): Docker netwroking.

1. List all network interfaces while nginx container is running in `bridge` mode (use `ifconfig` or `ip addr show`). Find `docker0` network and check its CIDR range. Go inside running container (`docker exec -it <container-id> bash`) and find its IP. Try to access port 80  in the container from host machine by using container internal IP address.
1. Adopt previous example to use [host network](https://docs.docker.com/network/network-tutorial-host/). 
1. Make sure you understand the difference between [host](https://docs.docker.com/network/host/) and default [bridge](https://docs.docker.com/network/bridge/) network.
1. Once running in network mode access the site via port 80 from host. What ip does the container have?


