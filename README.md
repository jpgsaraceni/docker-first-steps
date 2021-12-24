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
