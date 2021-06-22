<!-- @format -->

*These are personal notes I took during the time I spent learning about Docker and its associated tools.*
# Docker notes


## Image vs Container

Image is the application we want to run. Container is an instance of that image running as a process.

**Our image in this course will be Nginx web server**

`docker container run --publish 80:80 --detach --name webhost1 nginx (downloaded image from dockerhub, started a new container from that image and opened port 80 on localhost`

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

`docker network connect f5ac43071c18 e8dda46a60d5 (first number is network ID, second number is container ID)`

**To disconnect**

`docker network disconnect f5ac43071c18 e8dda46a60d5`

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
