# Building an example app

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
