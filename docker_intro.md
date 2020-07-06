# Docker Intro

Status: DevOps
Tags: docker

# Installing Docker

## Linux

- got to [docs.docker.com](http://docs.docker.com) and navigate to Linux
- choose your Distro - be sure to pay attention to the supported versions

```bash
# uninstall old versions
sudo apt-get remove docker docker-engine docker.io containerd runc
# using the convenience script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## Windows

### Docker Toolbox

### Docker Desktop

- OG support for Docker
- gives you a set of tools to make working with Docker on Windows easy
- This will use Oracle VirtualBox, then install Linux and then run Docker
- However, this is the old and deprecated way - now you can automate stuff by using Docker Desktop

- similar to the previous solution, but instead of VirtualBox this uses the native virtualization tool Hyper-V
- still a Linux system is created, but now on Hyper-V
- Docker then again runs on this Linux instance within Hyper-V
- This only works on Windows 10 Enterprise / Pro edition or on Windows Server 2016
- Also, Windows being Windows you cannot run Windows apps on the linux-based Docker, for this you need Windows Server 2016 and explicitly tell Docker to run Windows Containers instead of Linux Containers
- Hyper-V and VirtualBox cannot coexist either, you have to choose one - If you are already set up with one there are migration guides on the Docker website

## Mac

### Docker Toolbox

- same as with Windows
- Docker in a  Linux VM on the OSX-System

### Docker Desktop

- Similar to Hyper-V OSX replaces VirtualBox with the native HyperKit virtualization tool
- This again runs a Linux image on your Mac and there are not OSX containers at this time

## Test your Setup

- got to [hub.docker.com](http://hub.docker.com)
- search for whalesay
- copy the command on the righthand side

```bash
# pull this image from dockerhub
docker pull docker/whalesay
# run the image
docker run docker/whalesay cowsay Ay, Ay, Captain!
```

# Basic Docker Commands

- get an image

```bash
doker pull YOUR-APP-NAME
```

- list all containers

```bash
docker ps -a
```

- docker run image - creates container

```bash
docker run YOUR-APP-NAME
```

- stop container

```bash
docker stop YOUR-APP-NAME
```

- remove container

```bash
docker rm YOUR-APP-NAME
```

- list all container IDs

```bash
docker ps -a -q
```

- stop all containers

```bash
docker stop $(docker ps -a -q)
```

- remove all containers

```bash
docker rm $(docker ps -a -q)
```

- remove all images

```bash
docker rmi $(docker images -a -q)
```

- check out a container
- you can access a container over its container ID displayed in `docker ps -a`

```bash
docker inspect CONTAINER-ID
```

- check container logs with

```bash
docker logs CONTAINER-ID
```

# Running Docker

## Using Ports

- assuming that you run a docker app that sends results to an internal port, i.e. `5000`
- you cannot access that port `5000` since it is internal in the running container
- however, you can route the output of that port `5000` to another port the system docker is running on
- `80:5000` will rout docker output from port `5000` to port `80` on your system

```bash
docker run -p 80:5000 APP-NAME
```

## Environment Variables

- you can pass arguments to an app like this

```bash
docker run -p 80:5000 APP-NAME:argument
```

- or use `-e` (environment) to set environment variables
- configures env vars will be stored in the container meta-data, which you can inspect once again with `docker inspect ubuntu`

```bash
docker run -e ENV_VAR=somevalue ubuntu
```

## Changing the Container Name

- use `-d` (detach) to run the container in the background
- and the `–-name` tag to specify an app name (otherwise docker assigns a name at random)

```bash
docker run -p 38282:8080 --name uniqueappname -e APP_COLOR=blue -d user/your-app
```

- you can also run multiple instances of an app on multiple ports
- the docker container port remains the same but the host (your system) port remains the same

```bash
docker run -p 8080:5000 APP-NAME
```

## Directory Paths

- you can instanciate a DB inside of docker
- anything will be deleted once you stop and remove that intance though
- route that data to somewhere outside of docker to make it permanent
- similar to the ports we can map a location on your host system to a path inside docker
- example for running mysql in docker but storing data on the host

```bash
docker run -v /your/system/dir:/docker/system/dir mysql
```

# Creating your own Image

- the following is a sample Dockerfile from which you can build your image

```docker
FROM Ubuntu  # all Dockerfiles start with a FROM instruction, followed by an argumentRUN apt-get udpateRUN apt-get install python
RUN pip install flask
RUN pip install flask-mysql
COPY . /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source_code/app.py flask run  # this runs when image is run as a container
```

- specify your Dockerfile as the build source
- `-t` is a tag one can specify that is associated with the image

```bash
docker build Dockerfile -t /Users/username/myapp
```

- if your are in the dir that holds the Dockerfile you can also run

```bash
docker build -t APP-NAME .
```

- you can make an image public by pushing it to dockerhub
- replace `dockerhub-username` with your own username

```bash
docker push dockerhub-username/myapp
```

## More on the Dockerfile

- you can add the `CMD` instruction to define a command that will run when the container starts
- i.e. for ubuntu

```docker
CMD sleep 5
```

- or as JSON array

```docker
CMD ["sleep", "5"]
```

```bash
docker run ubuntu-sleeper # will run ubuntu and sleep for 5 seconds
```

- you can also run

```bash
docker run ubuntu-sleeper sleep 5 # which overrides the defaults
```

## Entrypoint instruction

- is what runs first when container is started

```docker
FROM Ubuntu
ENTRYPOINT ["sleep"]  # will run sleep command on ubuntu
```

- we then only specify for how long ubuntu will sleep

```bash
docker run ubuntu-sleeper 10
```

- to add a default to `ENTRYPOINT` add the `CMD` instruction beneath it

```docker
FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

```bash
docker run ubuntu-sleeper # sleeps for 5 seconds
docker run ubuntu-sleeper 10 # sleeps for 10 seconds
```

- override existing `ENTRYPOINT` instructions with

```bash
docker run --entrypoint sleep2 ubuntu-sleeper 10
```

- what port will be accessible can be also defined in the Dockerfile

```docker
EXPOSE 8080
```

- as well as the working directory inside the docker container

```docker
WORKDIR /opt
```

# Networking with Docker

- before we used the so called bridge network where we would assign ports to one another (it's the default)
- now we can also run Docker using the host network, which means multiple instances of an app cannot run in parallel since all ports are common to each other on the container network

```bash
docker run ubuntu --network=host
```

- another option is to define the `none` argument for the network parameter
- docker will run containers in an isolated network now

```bash
docker run ubuntu --network=none
```

- you can checkout all networks that are available with

```bash
docker network ls
```

- you can specify a new network by running

```bash
docker network create --driver bridge --subnet 182.18.0.0/16 custom-isolated-network
```

- IP adress and settings can once again be found by running

```bash
docker inspect CONTAINER-ID
```

## Embedded DNS

[DNS = Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System)

> The Domain Name System (DNS) is a hierarchical and decentralized naming system for computers, services, or other resources connected to the Internet or a private network. It associates various information with domain names assigned to each of the participating entities. Most prominently, it translates more readily memorized domain names to the numerical IP addresses needed for locating and identifying computer services and devices with the underlying network protocols. By providing a worldwide, distributed directory service, the Domain Name System has been an essential component of the functionality of the Internet since 1985.

- built in DNS of Docker hast the IP address `127.0.0.11`
- container in Docker can reach each other over their assigned IP address or their name
- better to use a name, since IP addresses can change when container reboot

```python
mysql.connect(container_name)  # here the container would likely be named "mysql"
```

# Docker Storage

## File System

- Docker stores data in `/var/lib/docker` (Linux) `/usr/local/lib/docker` (OSX) (or check location in Docker Desktop app)

```
/var/lib/docker
	- aufs
	- containers
	- image
	- volumes
```

## Layered Architecture

- Docker images that are built from Dockerfiles that are similar, will use Images that have been build before to reduce size
- i.e. Dockerfile1 is used to build app1, then Dockerfile2 is used to build app2
    - the Dockerfiles are the same for the first 3 layers of the Dockerfile
    - Docker sees this, and will reuse resources - already downloaded for app1 - for app2

```docker
# Dockerfile1
FROM Ubuntu  # Layer 1

RUN apt-get udpate && apt-get -y install python  # Layer 2

RUN pip install flask flask-mysql  # L3

COPY . /opt/source-code  # L4

ENTRYPOINT FLASK_APP=/opt/source_code/app.py flask run  # L5

# in bash run
docker build Dockerfile1 -t username/app1
```

```docker
# Dockerfile2
FROM Ubuntu

RUN apt-get udpate && apt-get -y install python

RUN pip install flask flask-mysql

COPY app2.py /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source_code/app.py flask run

# in bash run
docker build Dockerfile2 -t username/app2
```

- these savings in storage are possible in the Layered Architecture since it only stores the **changes** that have been made compared to the previous layer (built layers are stored in Cache)
- this also makes image building much faster and updates are also quickly implemented, by the same technique of reusing
- Image layers are READ-ONLY and shared between containers that are run from the same image
- now, running a container from an image (`docker run username/app`) will add a new layer to the architecture (Container Layer - would be Layer 6 in this example), which the will have READ & WRITE permissions and only exist while the container exists

### Copy-on-Write

- if you are working on an app and want to test some code changes, you can safely do that without modifying the actual source code that lives in the image layers
    - but wait - image layers are read-only
    - Docker will safe a copy of the file you are modifying in the container layer, therefore, nothing will change in the image layers
    - again all changes are lost when the container is removed

### Volume mounting

- to not loose changes we make in the container layer, we can add persistent storage

```bash
# create persistent volume on docker host
docker volume create some_volume

# mount volume when running docker
# /var/lib/mysql is the default path mysql will store data to
docker run -v some_volume:/var/lib/mysql mysql

# also - just running
docker run -v some_other_volume:/var/lib/mysql mysql  # automatically creates a volume and mounts it
```

### Bind Mounting

- another possibility is called **Bind Mounting**

```bash
# specify a path where you want your data stored on your system
docker run -v /usr/data/mysql:7var/lib/mysql mysql
```

- so now data will be in `/usr/data//mysql` instead of the default docker path

### `-v` works, though it is deprecated

```bash
# this is the new and preferred way
# here you be more specific of what you want exactly
# source is on the host
# target is the directory in the container
docker run --mount type=bind,source=/usr/data/mysql,target=/var/lib/mysql mysql
```

### Storage Drivers

- there are a couple
    - AUFS - ZFS - BTRFS - Device Mapper - Overlay - Overlay2
- Docker chooses them automatically
- but they can also be chosen manually
- on OSX Docker chose Overlay2 for me

# Docker Config

- if we need multiple services to run our app efficiently we can use `docker compose` instead of running each container individually

```bash
# boot up whole stack of services for your app with
docker compose up
```

- Docker compose config is stored in a YAML-file called `docker-compose.yml`
- makes your life easier by storing changes in the docker-compose file

```yaml
# sample docker-compose file
web:
	images: "user/your-app"
database:
	image: "mongodb"
messaging:
	image: "redis:alpine"
orchestration:
	image: "ansible"
```

## Setting up a Stack

```bash
# set up a bunch of containers
# in-memory storage
docker run -d --name=redis redis
# database
docker run -d --name=db --link db:db postgres:9.4
# voting app that is linked to redis
docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
# report results from the voting app to the DB
docker run -d --name=result -p 5001:80 --link db:db result_app
# worker (is a .NET programm that connect the DBs to the app)
docker run -d --name=worker --link db:db --link redis:redis worker
```

- linking stuff together in this way is deprecated, but important to know

## Writing a docker-compose File for the Stack

```yaml
redis:
	image: redis
db:
	image: postgres:9.4
vote:
	image: voting-app
	port:
		- 5000:80
	links:
		- redis
result:
	image: result-app
	ports:
		- 5001:80
	links:
		- db
worker:
	image: worker
	links:
		- redis
		- db
```

- the `docker-compose.yml` assumes that all necessary images are already available on the host
- if that is not the case the `image: XXX` command can be substituted with `build: ./image-dir` telling Docker to build the image from the given path

## Newer Versions of docker-compose

- the example shown above are for docker-compose version 1
- you cannot specify networking in version 1, therefore versions 2 and up where developed

### docker-compose:version2

- Docker now creates a dedicated bridge network for the app you are running and then attaches all containers to that network
    - the services running on there can then use their network names to call upon one another
    - **this mean you do not nee to use links in version 2**
- Furthermore, dependencies can be declared in version 2
    - i.e. tell voting-app to hang on until redis has started

```yaml
version: 2
services:
	redis:
		image: redis
	db:
		image: postgres:9.4
	vote:
		image: voting-app
		port:
			- 5000:80
		depends_on:
			- redis
	result:
		image: result-app
		ports:
			- 5001:80
	worker:
		image: worker
```

### docker-compose:version3

- there is also a version 3 now which has brought additional features to the table including the integration of Docker Swarm

## Networking in docker-compose

```yaml
version: 2
services:

	redis:
		image: redis
		networks:
			- back-end

	db:
		image: postgres:9.4
		networks:
			- back-end

	vote:
		image: voting-app
		networks:
			- front-end
			- back-end
		depends_on:
			- redis

	result:
		image: result-app
		networks:
			- front-end
			- back-end

	worker:
		image: worker

# add networks here
networks:
	front-end: XXX
	back-end: XXX
```

# Docker Registry

- central storage for all Docker containers
- Docker Registry provides the container when we run `docker run nginx`
- `nginx` here is the name of the image, but according to Docker naming conventions `nginx` would be incomplete without specifying also the user like `user/image`, therefore Docker assumes a user name which is equal to the image name in this case `nginx/nginx`
- also, if not otherwise specified, Docker assumed that the images are hosted on DockerHub (docker.io)
    - in case of `nginx` > `docker.io/nginx/nginx`
- another well-known registry is [gcr.io](http://gcr.io) from Google (kubernetes)
- private registries can be used for proprietary stuff

```bash
# login to your teams registry
docker login private-reg.io
# start a container from the private registry
docker run private-reg.io/user/secret-app
```

- committing changes to private images requires you to specify the registry

```bash
# rsetup registry on port 5000
docker run -d -p 5000:5000 --name reg registry:2
# tag image with private image URL
# > here localhost:5000 - since it is running locally
docker image tag ye-image localhost:5000/ye-image
# now push your image to the registry
docker push localhost:5000/ye-image
# and also retrieve the image from anywhere on the host by running
docker pull localhost:5000/ye-image
# or use IP address if you are in the same environment
docker pull 192.168.56.100:5000/ye-image
```

# Docker Engine

- installing Docker actually installs 3 things
    - Docker CLI
        - receives your commands and used the REST API to talk to the deamon
        - does not need to be on the same host as the deamon, but still use the same engine
    - REST API
    - Docker Deamon

```bash
# connect to remote engine
docker -H=remote-docker-engine:2345

# running app nginx over remote engine
docker -H=10.123.2.1:2375 run nginx
```

## Containerization

- use namespace to isolate workspaces
    - i.e. mount, network and process ID all have their own namespace, which isolates them from the other services

### Example: process ID namespace

- the parent system or host (linux) starts with one process ID (PID: 1), which then triggers all other processes
- these processes are unique and cannot have the same name / ID
- a containerized child system will have its own PID: 1, BUT there is no clear cut between the host and child system, which means that processes that are in the child namespace will also appear on the host - just under a different name / ID

## cgroups

- Docker container can be limited in how much of the resources of the host they are allowed to use

```bash
# limit the CPU usage of the container to 25%
docker run --cpus=.25 yourapp

# limit memory to 200MB
docker run --memory=200m
```

# Container Orchestration

- apps usually can only handle so much load at a time
- so when user numbers increase you may need to run additional containers of the same app, to deal with the higher load
- this is a manual task and can become difficult when scaling, i.e. when your docker instance crashes or containers fail
- therefore, we have orchestration tools
- usually multiple instances of docker are run in parallel with a certain number of containers with your app running inside each container
- orchestration tools also provide advanced networking between the containers, Docker instances and across different hosts, clustering, load balancing, storage sharing, config management, security features, etc.

## Docker Swarm

- Docker native orchestration solution
- some features are lacking when deploying large, complex production-grade apps
- combine several services into one cluster
- then run Docker Swarm to distribute these services over multiple hosts (this is good for load balancing and availability)

### High-level Docker Swarm Intro

- to use Swarm you first need multiple host systems with Docker installed
    - then one host is designated to be the master or Swarm Manager and the rest of the hosts are the Slaves or Workers

```bash
# launch swarm
docker swarm init
# add worker
docker swarm join --token SOME-TOKEN  # this command needs to be run on the workers and manager host
# run 3 nodejs across all available swarm clusters
# auto load-balancing, monitoring, restarts automatically, ...
docker service create --replicas=3 nodejs
# serve to specific port
docker service create --replicas=3 -p 8080:80 ye-webapp
# serve to specific network
docker service create --replicas=3 -network frontend ye-webapp
```

## Kubernetes

- Googles solution for container orchestration
- somewhat complex setup, but provides good features to customize deployments and also supported by many vendors

### High-level Kubernetes Intro

- adaptive user-load scaling
- lots of plugins for other services
- there are multiple containers that run your app aka. Nodes
- each Node has Kubernetes installed
- a group of Nodes is a cluster
- even if one Node goes down you still have the others running in your cluster which ensures your app stays up

#### Kubernetes Components

- API Server - UI Front-end, CLI
- etcd - meta-data storage
- kubelet - Agent that makes sure container are running on the Nodes as expected
- Container Runtime - underlying software to run containers (here: Docker)
- Controller - responds to failures in the system, i.e. restarts containers or Nodes
- Scheduler - distributing work or containers to Nodes

```bash
# run a service
kubectl run nodejs
# checkout health of cluster
kubectl cluster-info
# checkout running nodes
kubectl get nodes

# start 1000 webservers
kubectl run --replicas=1000 nodejs
# scale to 2000 webservers
kubectl scale --replicas=2000 nodejs

# syou can also specify the image
kubectl run ye-app --image=ye-web-app --replicas=50

# update services incrementaly
kubectl rollin-update nodejs --image=nodejs:2
# update services incrementaly
kubectl rollin-update nodejs --rollback
```

## Mesos

- difficult setup but provides very advances features

## Resources

Many Thanks go out to FreeCodeCamp to sharing KodeKlouds amazing video tutorial on Docker.
I found it to be very engaging and it definetly gave me a good understanding of the tool.
Also, the course on KodeKloud's plattform is quite nice to test out the new concepts and commands.
The course is less guided compared to my courses on DataCamp and CodeCademy, which I like, since it requires you to think more for yourself.

- [KodeKloud Video on FreeCodeCamp's YouTube](https://www.youtube.com/watch?v=fqMOX6JJhGo&t=1332s)
- [KodeKloud Website with free course](https://kodekloud.com/p/docker-labs)
- [Instructor Mumshad Mannambeth's LinkedIn](https://www.linkedin.com/in/mumshad-mannambeth-29987029/)
- [DockerHub](https://hub.docker.com/)
- [Docker Documentation](https://docs.docker.com/)