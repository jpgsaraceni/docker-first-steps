# First steps with docker

My study repository for docker.

Containers allow flexibility for development accross different machines with different OSs, versions, configurations etc.

An image contains all configuration related to the aplication (including dependencies, env variables, run scripts...), requiring no individual configuration on a host.

A container is an instance of an image running isolated on the host machine. Containers can run on local machines, VMs or deployed on a cloud.

The instructions here are for running docker on Linux. For other OSs check the [docker docs](https://docs.docker.com/).

## Requirements

* [Docker](https://docs.docker.com/engine/install/)

## Start docker

After installing, run:

```shell
sudo service docker start
```

The tutorial begins doing eveything separetely, and only after uses Docker Compose (which makes things *a lot* more simple).

Docker Compose allows you to have a single YAML file ate the root of your directory, with all the necessary configuration for the multiple containers your app may use.

## Building an example app

Contains code from the [docker docs Get Started tutorial](https://docs.docker.com/get-started/).

## 1. Run the docker/getting-started image

Run in your terminal:

```shell
sudo docker run -d -p 80:80 docker/getting-started
```

This is just an example image and this step is not actually necessary for the next ones.

### Flags `-d` and `-p`

`-d` means the container will run in detached mode (in the background).
`-p 80:80` means port 80 of the host will be mapped on port 80 of the container.

### Dockerfile

This file is what tells docker how to build a container image.

```dockerfile
# base image
FROM node:12-alpine
RUN apk add --no-cache python2 g++ make
WORKDIR /app
# tutorial says to use `COPY . .`, but when I ran docker
# it couldn't find the app directory in the container
COPY /app .
RUN yarn install --production
# default command to run when starting this image
CMD ["node", "src/index.js"]
```

## 2. Build container image in app directory

All following Docker Commands are to be run in the app directory.

### app dierectory

This app was not written by me, it was downloaded [following the docker docs tutorial](https://docs.docker.com/get-started/02_our_app/). To add it so you can test the docker commands described here, clone or download from its [repo](https://github.com/docker/getting-started/tree/master/app).

```shell
sudo docker build -t getting-started .
```

This command builds a container image according to Dockerfile.
The `-t` flag tags the container image named `getting-started`.
The `.` at the end of the command is the path to Dockerfile.

## 3. Run the app

```shell
sudo docker run -dp 3000:3000 getting-started
```

## 4. Updating the app

After making changes to app files, you must first stop the container. To do so, first you need the ID of the container:

```shell
sudo docker ps
```

Then, stop the container image by running the following command (replace `<CONTAINER_ID>` with the ID you got from the last command).

```shell
sudo docker stop <CONTAINER_ID>
```

Now remove it:

```shell
sudo docker rm <CONTAINER_ID>
```

Or you can stop and remove in the same command using the force tag

```shell
sudo docker rm -f <CONTAINER_ID>
```

Start the updated container:

```shell
sudo docker run -dp 3000:3000 getting-started
```

## 5. Share an image

To share a docker image we use a docker registry. [Docker Hub](https://hub.docker.com/) is the default registry. Create an account or login, then login from your terminal:

```shell
sudo docker login -u YOUR_USERNAME
```

Now create a repository on Docker Hub (on the website).

Change the tag of your local image:

```shell
sudo docker tag getting-started YOUR_USERNAME/getting-started
```

To push your image to the repository, use the docker command shown on the Docker Hub page. It will look like this (you don't need the :tagname portion because you didn't add a tag to the image name so it will default to latest):

```shell
sudo docker push YOUR_DOCKER_USERNAME/YOUR_REPOSITORY_NAME
```

## 6. Run on new instance

The image on a docker registry can be run on [Play with Docker](https://labs.play-with-docker.com/).

Just login, click on + NEW INSTANCE, and run on the terminal that will appear:

```shell
docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
```

## 7. Persist data

Data can be persisted accross container images using container volumes.

Create a named volume:

```shell
sudo docker volume create todo-db
```

Stop the container image and run it again adding the `-v` tag followed by `NAMED_VOLUME:PATH_TO_MOUNT`:

```shell
sudo docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

## 8. Bind mounts

Another way to persist data on a docker container image is to have a bind mount. Differently from a named volume, you can specify where the data will be persisted on the host, along with additional data.

To run a container to support development workflow in node:

```shell
docker run -dp 3000:3000 \
     -w /app -v "$(pwd):/app" \
     node:12-alpine \
     sh -c "yarn install && yarn run dev"
```

`-w /app` sets the “working directory” or the current directory that the command will run from
`-v "$(pwd):/app"` bind mount the current directory from the host in the container into the /app directory

## 9. Networks

In order for multiple containers to communicate with one another, they must be on the same network. To create a network run:

```shell
sudo docker network create <CONTAINER_NETWORK_NAME>
```

For this example, start a MySQL container:

```shell
sudo docker run -d \
     --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:5.7
```

To check the connection is up and running:

```shell
 sudo docker exec -it <MYQL_CONTAINTER_ID> mysql -u root -p
```

You can use [nicolaka/netshoot container](https://github.com/nicolaka/netshoot) for troubleshooting and debugging networking. Make sure to run it on the same network you created before.

```shell
sudo docker run -it --network todo-app nicolaka/netshoot
```

In the container, to find the IP adress of the MySQL container, if you used the `--network-alias` flag as previously instructed:

```shell
 dig mysql
```

Look for the IP adress in the outpu `; ; ANSWER SECTION:`. Use this IP in your app to connect to the mysql server.

Now run this on the mysql container:

```shell
 ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'secret';
 flush privileges;
```

Stop your app container, then run with the environment variables for the app to connect to MySQL:

```shell
sudo docker run -dp 3000:3000 \
   -w /app -v "$(pwd):/app" \
   --network todo-app \
   -e MYSQL_HOST=mysql \
   -e MYSQL_USER=root \
   -e MYSQL_PASSWORD=secret \
   -e MYSQL_DB=todos \
   node:12-alpine \
   sh -c "yarn install && yarn run dev"
```

You can add items to the app list through the web app you are running, and check them out in your MySQL container. To do this, access the mysql CLI like before and run:

```shell
select * from todo_items;
```

## 10. Docker compose

Install Docker Compose (Linux):

```shell
 sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Add execute permission to the binary:

```shell
sudo chmod +x /usr/local/bin/docker-compose
```

Check if instalation is ok:

```shell
docker-compose --version
```

### The docker-compose.yml file

Add the `docker-compose.yml` file to the root or your app.

Set `version` to the latest supported version, which you can see at check at [the Compose file reference](https://docs.docker.com/compose/compose-file/).

`services` lists the containers the app needs to run.

After configuring the docke-compose file, make sure no containers are running.

You don't need to define a network for the application stack, so you don't need to define one.

Now start up the application stack running:

```shell
sudo docker-compose up -d
```

The `-d` flag runs everything in the background.

To tear the container down, run:

```shell
sudo docker-compose down
```

Add the `--volumes` tag to delete created named volumes
