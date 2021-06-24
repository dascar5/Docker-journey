<!-- @format -->

*This is a personal notebook I'll be filling up while learning about Docker and associated tools, daily*
# Docker notes


## Image vs Container

Image is the application we want to run. Container is an instance of that image running as a process.

**Our image in this course will be Nginx web server**

`docker container run --publish 80:80 --detach --name webhost1 nginx (downloaded image from dockerhub, started a new container from that image and opened port 80 on localhost)`

Containers and VM's aren't the same thing, container is just a restricted process in the OS.

**To CLI into a container, we do it like this:**

`docker container run -it --name proxy nginx bash`

**To start a stopped container, do this**

`docker container start -ai ubuntu`

**To CLI into an already running container, do this** (exiting it doesn't stop the container like the case with the top one, because it wasn't started with bash command, but with exec)

`docker container exec -it mysql bash`

## Docker networks

**To create a docker network where we can attach containers** (bridge driver by default)

`docker network create my_app_nett`

**To start a container on new network**

`docker container run -d --name new_nginx --network my_app_nett nginx`

**To connect a docker from my default network to my new network, I do**

`docker network connect <network id> <container id>`

**To disconnect**

`docker network disconnect <network id> <container id>`

The reason these networks exist is for protection - keep ports safe, lets say keep frontend on one network, backend on another.

**When we put containers on the same network, they can talk to each other**
`docker container exec -it nginx1 ping nginx2 (nginx1 container pings nginx2 container)`

Containers shouldn't rely on IP addresses when talking to each other, DNS automatically resolves that in the background (inter-communication between containers).

## Docker alias

**To give a container an alias we do it like this**
`docker container run -d --name elsearch1 --network round_robin --network-alias search elasticsearch:2`

**To run a command within a container and instantly exit it upon it completing**
`docker container --rm --net round_robin alpine:3.10 nslookup search (enters alpine:3.10 container, does nslookup search to see all addresses on my round_robin network)`

This was a round robin test - set up 2 identical containers with same alias (search). When you call a function on those containers, either one of them responds, depending who gets reached first. Basically, if one fails, the other one can do the same task with no interruptions (think of multiple servers accross the world for google or riot).

## Tags

**To retag a repo, do this**
`docker image tag nginx dantesvn/nginx`

**To push a repo on docker hub, we do this**
`docker image push dantesvn/nginx`

## Dockerfile

**To build an image from dockerfile**
`docker image build -t customnginx . (-t is tag, customnginx is name of image)`

Order of lines matters in dockerfile, keep those lines that change the least at the top, because everything after the changed line has to be rebuilt again.

## Volumes and Bind mounts

Containers are supposed to be immutable - don't change containers, just re deploy it with changes, what about databases though? Containers shouldn't contain that kind of data, unique data is supposed to be in a special place. Docker has two options for containers to store files in the host machine, so that the files are persisted even after the container stops: **volumes, and bind mounts**.

**Volumes** are stored in a part of the host filesystem which is managed by Docker (/var/lib/docker/volumes/ on Linux). Non-Docker processes should not modify this part of the filesystem. Volumes are the best way to persist data in Docker.
**Bind mounts** may be stored anywhere on the host system, they may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time.

**To create and name a volume, and have a container on it, we add this to the docker container run command**

`v mysql-db:/var/lib/mysql (-volume, name, location)`

Bind Mounting maps a host file or directory to a container file or directory. Can't use in Dockerfile, must be at container run.
`... run -v //C/Users/Bogdan/Desktop:/path/container`
`docker container run -d --name nginx -p 80:80 -v ${pwd}:/usr/share/nginx/html nginx (${pwd} is "print working directory", basically my current working directory)`

## Docker Compose 

**Docker Compose** is used to configure relationships between containers. It saves our docker run settings in easy to read file, which leads to one-liner developer environment startups. Its comprised of 2 separate but related things:
1) YAML formatted file that describes our solution options for containers, networks and volumes
2) A CLI tool docker-compose used for local dev/test automation with those YAML files

A template of a docker-compose.yml looks like this
```
version: '3.1'  # if no version is specified then v1 is assumed. Recommend v2 minimum

services:  # containers. same as docker run
  servicename: # a friendly name. this is also DNS name inside network
    image: # Optional if you use build:
    command: # Optional, replace the default CMD specified by the image
    environment: # Optional, same as -e in docker run
    volumes: # Optional, same as -v in docker run
  servicename2:

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```
docker-compose CLI is not a production-grade tool but it's ideal for local dev and test. 

**Two most common commands are:**

`docker-compose up` (setup volumes/networks and start all containers)

`docker-compose down` (stop all containers and remove cont/vol/net)

Compose can also build custom images. It will build them with docker-compose up if not found in cache.

```
version: '2'

# based off compose-sample-2, only we build nginx.conf into image
# uses sample HTML static site from https://startbootstrap.com/themes/agency/

services:
  proxy:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    ports:
      - '80:80'
  web:
    image: httpd
    volumes:
      - ./html:/usr/local/apache2/htdocs/
```
For example, in here, first service is a custom image built based on nginx.Dockerfile:
```
FROM nginx:1.13

COPY nginx.conf /etc/nginx/conf.d/default.conf
```
The dockerfile states a custom .conf file replacing the default one in nginx image:
```
server {

	listen 80;

	location / {

		proxy_pass         http://web;
		proxy_redirect     off;
		proxy_set_header   Host $host;
		proxy_set_header   X-Real-IP $remote_addr;
		proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header   X-Forwarded-Host $server_name;

	}
}
```
It's just copying this data into the default image, thus making it "different" - custom-nginx. Docker-compose up then builds it based on yaml file where all of this is connected, using httpd (apache) server and bind-mounting the html files in directory, so it display a static website.

## Swarm

How do we scale out/in/up/down?
How can we ensure our containers are re-created if they fail?
How can we replace containers without downtime?
How can we control/track where containers get started?
How can we create cross-node virtual networks?
How can we ensure only trusted servers run our containers?
How can we store secrets, keys, passwords and get them to the right container (and only that container)?

**Swarm Mode: Built-In Orchestration**

Swarm mode is a clustering solution built inside Docker. It's not enabled by default, once enabled we get access to: 
```
docker swarm
docker node
docker service
docker stack
docker secret
```
A Docker Swarm is a group of either physical or virtual machines that are running the Docker application and that have been configured to join together in a cluster. Once a group of machines have been clustered together, you can still run the Docker commands that you're used to, but they will now be carried out by the machines in your cluster. The activities of the cluster are controlled by a swarm manager, and machines that have joined the cluster are referred to as nodes.
Docker swarm is a container orchestration tool, meaning that it allows the user to manage multiple containers deployed across multiple host machines.
One of the key benefits associated with the operation of a docker swarm is the high level of availability offered for applications. In a docker swarm, there are typically several worker nodes and at least one manager node that is responsible for handling the worker nodes' resources efficiently and ensuring that the cluster operates efficiently.

**To initialize swarm on docker, do:**

`docker swarm init `

**The terminal will output a tocken you use to connect other nodes to your current leader node:**

`docker swarm join --token <your token> 192.168.65.3:2377`

**To list all current nodes (only 1 - leader node so far):**

`docker node ls`

**To run a service and have it do something:**

`docker service create alpine ping 8.8.8.8`

**To scale up this service, by making it have 3 replicas:**

`docker service update <service id> --replicas 3` 

Even if we force remove a running container on swarm, swarm will automatically start it up back again to match specified replica (if we set up 3, and deleted 1, it will automatically revive it to match 3/3). That's the point of an orchestration service, make sure it **always runs!** Docker run would never recreate a removed container, while the whole point of orchestration is to keep everything running accross all nodes.

**To shut it down, we actually have to shut down the service:**

`docker service rm <service id>`

**To create a "remote" node for testing**, we use **docker-machine**, which boots up a docker ready linux image in a VM for our swarm purposes. I am going to create 3 nodes:
```
docker-machine create node1
docker-machine create node2
docker-machine create node3
```
**To access the newly created machine, we can do:**
```
docker-machine ssh node1
docker-machine ssh node2
docker-machine ssh node3
```

**Command `docker swarm init` probably won't work on its own due to how cloud works, so we use in each of the nodes:**

`docker swarm init --advertise-addr <IP address>`

This turns node1 into swarm manager, and pasting the token into node2 and node3 makes them "workers" and connects them to node1 (manager), thus making the swarm.

Only swarm manager can use swarm commands, running `docker node ls` on node2 or node3 will throw an error.

**This command turns node2 into a manager as well:**

`docker node update --role manager node2`

**We can also make a node a manager by getting the join-token manager. Only works on nodes that haven't already joined:**

`docker swarm join-token manager`

**To list the running service accross nodes:**

`docker service ps <service name>`
