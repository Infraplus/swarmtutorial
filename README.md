# Swarm Tutorial
Webserver Docker Image that display Ip address of the running container (Used for Docker Swarm Tutoriel)
 
# Prerequistes 
Docker installed
Docker-machine installed (optionnal, you can create your own VMs)

## Docker installation:
```
echo "deb https://apt.dockerproject.org/repo ubuntu-$(grep CODENAME /etc/lsb-release | awk -F'=' '{print $NF}') main" | sudo tee /etc/apt/sources.list.d/docker.list</br>
sudo apt-get update</br>
sudo apt-get install apt-transport-https ca-certificates -y</br>
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80  --recv-keys 58118E89F3A912897C070ADBF76221572C52609D</br>
sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual docker-engine -y</br>
```

For more information: https://docs.docker.com/engine/installation/linux/ubuntulinux/

## Docker-machine installation (Docker should be installed):
```
sudo curl -L https://github.com/docker/machine/releases/download/v0.8.2/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && sudo chmod +x /usr/local/bin/docker-machine</br>
```

For more information:https://docs.docker.com/machine/install-machine/

# How to:
You can create your image or pull it directly from dockerhub:

## Create local image
```
$git clone git@github.com:shitana/webserverimage.git</br>
$cd webserverimage/</br>
$sudo docker build -t webserverphp:latest .</br>
--snip--</br>
$ sudo docker images</br>
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE</br>
webserverphp                latest              bbe0e090daa3        41 hours ago        307 MB</br>
```

## Pull image from dockerhub
```
$docker pull salmenhitana/webserverphp
```

## Run the container:
```
$sudo docker run -d -p 8080:80 webserverphp</br></br>
The webpage should return the running container IP address</br>
$curl localhost:8080</br>
172.17.0.2  </br>
```

# Docker swarm tutorial
Create a web service who listen on port 8080 and display the IP address of the host where is running.
* Create a web container (optionnal)

## Prepare environment 
### Create machines
```
$ sudo docker-machine create --driver virtualbox --virtualbox-memory "1024" manager1 
$ sudo docker-machine create --driver virtualbox --virtualbox-memory "1024" worker1 
$ sudo docker-machine create --driver virtualbox --virtualbox-memory "1024" worker2
```

### List available machine:
```
$ sudo docker-machine ls
NAME       ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER    ERRORS
manager1   -        virtualbox   Running   tcp://192.168.99.100:2376           v1.12.5   
worker1    -        virtualbox   Running   tcp://192.168.99.103:2376           v1.12.5   
worker2    -        virtualbox   Running   tcp://192.168.99.102:2376           v1.12.5   
```

### SSH connexion
```
$ docker-machine ssh MACHINE_NAME
```

## Prepare Swarm
### Create swarm (Manager1)
```
docker@manager1:~$ docker swarm init --advertise-addr 192.168.99.100
```

### Join Swarm (worker1 and worker2)
```
docker@workerX:~$ docker swarm join --token SWMTKN-1-3qu42by3xxkxh... MANAGER_IP:2377
```

### List swarm node (Manager1)
```
docker@manager1:~$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
1gvlzqrut1fapq320a367ebdc    worker1   Ready   Active        
b0j984s3hr6qa4ls96ntw06it    worker2   Ready   Active        
beipilfnnv9giucqfuk4lg6xm *  manager1  Ready   Active        Leader
```
## Service management
### Create service
```
docker@manager1:~$ docker service create --replicas 3 --publish 8080:80 --name swarmtuto salmenhitana/webserverphp
```
### Inspect service
```
docker@manager1:~$ docker service ps swarmtuto
ID                         NAME         IMAGE                      NODE      DESIRED STATE  CURRENT STATE             ERROR
eavuxhf521q3lziwhs0adkvsp  swarmtuto.1  salmenhitana/webserverphp  worker2   Running        Preparing 13 seconds ago  
95msg6fyaece6swbdk1jgh4k8  swarmtuto.2  salmenhitana/webserverphp  worker1   Running        Preparing 13 seconds ago  
5uiosevqepk8cc244vw6xqwzt  swarmtuto.3  salmenhitana/webserverphp  manager1  Running        Preparing 13 seconds ago
```

### Check Swarm loadbalancer
From your host machine, request your webserver

* Retrieve ip addresses of your docker machines:
```
$ sudo docker-machine ls
** $ sudo docker-machine ip MACHINE_NAME
```

* Send GET request (curl IP:8080)
```
$ for machine in manager1 worker1 worker2; \
  do \
    IP=$(sudo docker-machine ip $machine); \
    echo "Send 3 request to $machine"; \
    for i in {0..2}; \
    do \
      curl $IP:8080; echo ; \
    done; \
  done
```
OUTPUT
```
Send 3 request to manager1
10.255.0.6
10.255.0.8
10.255.0.7
Send 3 request to worker1
10.255.0.6
10.255.0.8
10.255.0.7
Send 3 request to worker2
10.255.0.6
10.255.0.8
10.255.0.7
```
