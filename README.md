# API caching - python application with AWS ALB
Here, it is designed with an AWS Application loadbalancer and AWS Elastic cache for api caching.

Using ipstack API in this python application as a demo. This API will show you the Geolocation of IP addresses.

# Architecture
![](https://i.ibb.co/nwryhRb/2.jpg)

### Features
- IP-Location finding website (Python) 
- Easy to migrate everywhere 
- AWS ALB - load balanced
- All containers are spin up with a single command

### Includes
- Launch Configuration with the userdata provided.
- Auto scaling group with capacity as 2
- Target Group(Manager and worker server attached)
- Application Load balancer
- Elastic Cache

## How to get IPstack API
> Please go through the [ipstack](https://ipstack.com/) and click "_GET FREE API KEY_" on the top right corner. 
![alt text](https://i.ibb.co/sFb3zz6/ipstack.png)

# Docker
Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. 

Docker provides the ability to package and run an application in a loosely isolated environment called a container. The isolation and security allow you to run many containers simultaneously on a given host. Containers are lightweight and contain everything needed to run the application, so you do not need to rely on what is currently installed on the host. You can easily share containers while you work too.
You can learn more about the same in the [website.](https://docs.docker.com/get-started/overview/)

# What Is a Container
A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.
You can learn more about the same in the [website.](https://www.docker.com/resources/what-container)

# Installing Docker
> Here I am using AWS Amazon linux server
```sh
amazon-linux-extras install docker -y
systemctl restart docker.service
systemctl enable docker.service
```
Please see the [website](https://docs.docker.com/engine/install/) for more information.

# Docker Swarm
A cluster management and orchestration features embedded in the Docker Engine. A swarm consists of multiple Docker hosts which run in swarm mode and act as managers (to manage membership and delegation) and workers (which run swarm services). A given Docker host can be a manager, a worker, or perform both roles. When you create a service, you define its optimal state (number of replicas, network and storage resources available to it, ports the service exposes to the outside world, and more). 

Please see the [website](https://docs.docker.com/engine/swarm/key-concepts/) for more information.

# Initializing Docker Swarm in an instance - Manager
```sh
docker swarm init
```
```
Swarm initialized: current node (ugwpr5x9fhydrx4129fvp661b) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-62bnv3a3qjqaxff2b12aqtfmahnttg087p4d2a22qsft2o8onm-dnzt2y2yzjtswx2kvro4cl2vh 172.31.9.246:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
> To make the workers join in the swarm/cluster need to enter the command/token in the worker instance.
- Here, I have provided this command/token in the userdata which is defined in the Launch Configuration.

## Userdata
```sh
#!/bin/bash

echo "ClientAliveInterval 60" >> /etc/ssh/sshd_config
echo "LANG=en_US.utf-8" >> /etc/environment
echo "LC_ALL=en_US.utf-8" >> /etc/environment

echo "password123" | passwd root --stdin
sed  -i 's/#PermitRootLogin yes/PermitRootLogin yes/' /etc/ssh/sshd_config
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
service sshd restart

amazon-linux-extras install docker -y
systemctl restart docker.service
systemctl enable docker.service
docker -a -G docker ec2-user

docker swarm join --token SWMTKN-1-5nzfzamkd9hd8ha80zz3q66httkgo0y9wkh7e0cbcwj3nvqgom-8k42ywr5abh71o0ejv8d810os 172.31.41.35:2377
```
## Working
- Installing service network
```sh
docker network create alb --driver=overlay
```
- Using Elastic cache in AWS for api caching. However, instead of Elastic cache can use a Redis container.
```sh
docker service create \
--name redis \
--replicas 1 \
--network alb \
redis:latest
```
- Creating ipstack container. These containers are created in any of the worker instance or even in the Manager instance.
```sh
docker service create \
--name stack \
--replicas 5 \
--network alb \
-p 80:8080 \
-e CACHING_SERVER=caching \
-e IPSTACK_KEY=e895**************************** \   ### ==> Your ipstack API KEY
fujikomalan/ipstack:v1
```
```sh
[root@ip-172-31-41-35 ~]# docker service ls
ID             NAME      MODE         REPLICAS   IMAGE                    PORTS
vt0mct1xjwcn   stack     replicated   5/5        fujikomalan/ipstack:v1   *:80->8080/tcp
```

# Conclusion
Here is a simle API caching python application with AWS ALB
