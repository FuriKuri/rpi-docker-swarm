# Docker Swarm on Raspberry PI
![PI Cluster](https://github.com/FuriKuri/rpi-docker-swarm/raw/master/pi-cluster.jpg "PI Cluster")

## Preparations
### Install OS
I used [HypriotOS](http://blog.hypriot.com/) in version 1.0.0. For an easy installation you can use [flash](https://github.com/hypriot/flash). Just insert your SD card and run the following command from your command line.

```
$ flash --hostname moby-dock-1 https://downloads.hypriot.com/hypriotos-rpi-v1.0.0.img.zip
```
Repeat this for your other RPIs. Do not forget to increase the hostname.

In addtion I configured my DHCP server, that the RPIs get the following IPs:

* 192.168.0.200
* 192.168.0.201
* 192.168.0.202
* 192.168.0.203

To connect to your RPIs use pirate as username and hypriot as password.

```
$ ssh pirate@192.168.0.200
```

## Create cluster
Just type the following command on the first RPI

```
$ docker swarm init
```
and let the other ones join the cluster (the complete command with your token should be printed after the ```docker swarm init```)

```
$ docker swarm join --token <swarm-token> 192.168.0.200:2377
```

After this all nodes should be listed with ```docker node ls```:

```
$ docker node ls
ID                           HOSTNAME     STATUS  AVAILABILITY  MANAGER STATUS
4whs5eu27bzxvp3766frrpk8c    moby-dock-3  Ready   Active
6wz46d0uezvyq6ujm9qi707j5    moby-dock-2  Ready   Active
7xfsvx083ds6syge3j5vk8gbk    moby-dock-4  Ready   Active
8q2k9se39x9kbz508h9trwpuz *  moby-dock-1  Ready   Active        Leader
```

## Start Services
Now let's start some services. For this I created a simple application written in Go. This application is a simplified verison of pong. We will start three services.
Two services will be the client named ```ping``` and ```pong```. Both clients can be request over HTTP and they will return ```HIT``` or ```MISS```, according as whether he hits the ball or not. The ```ping-pong-manager``` will request both clients until one of them returns ```MISS```.

First of all we need to create an own network, which will be used by our services:

```
$ docker network create --driver overlay --subnet 10.0.9.0/24 ping-pong-net
```

Now we can start the services. (I needed to add ```--endpoint-mode dnsrr```. Without this option the network performance was horrible.)

```
$ docker service create --replicas=2 --network=ping-pong-net --endpoint-mode dnsrr --name ping furikuri/rpi-ping-pong --hit-chance 85
$ docker service create --replicas=2 --network=ping-pong-net --endpoint-mode dnsrr --name pong furikuri/rpi-ping-pong --hit-chance 85
$ docker service create --replicas=2 --network=ping-pong-net --endpoint-mode dnsrr --name ping-pong-manager furikuri/rpi-ping-pong --mode server
```

In addition we will start a simple proxy on every node. So every node will be able to serve the service, irrespective of whether or not the node has a running service instance of ```ping-pong-manager```.

```
$ docker service create --mode=global --network=ping-pong-net --name ping-pong-proxy --publish 3000:3000 furikuri/rpi-ping-pong-proxy
```

All services use the service name and the service discovery mechanism form docker. For example the ```ping-pong-manager``` will do a GET request to ```http://pong:3000```.

Let's check if all services are running:

```
$ docker service ls
ID            NAME               REPLICAS  IMAGE                         COMMAND
0s9upwuahval  pong               2/2       furikuri/rpi-ping-pong        --hit-chance 85
31bsg8nx30aa  ping-pong-proxy    global    furikuri/rpi-ping-pong-proxy
bg9omcs2p5gw  ping               2/2       furikuri/rpi-ping-pong        --hit-chance 85
e1j2dfgey2a2  ping-pong-manager  2/2       furikuri/rpi-ping-pong        --mode server
```


Now we can start a game from outside:

```
# theo at toad in ~/projects/rpi-docker-swarm on git:master ● [23:07:48]
→ curl http://192.168.0.200:3000
PING --> HIT
PONG --> HIT
PING --> HIT
PONG --> HIT
PING --> HIT
PONG --> HIT
PING --> HIT
PONG --> HIT
PING --> HIT
PONG --> HIT
PING --> HIT
PONG --> HIT
PING --> HIT
PONG --> HIT
PING --> MISS
The winner is PONG
```

## Services Visualisation
The [docker-swarm-visualizer](https://github.com/ManoMarks/docker-swarm-visualizer) offers a nice and easy way to visualize all docker services in your docker swarm cluster. For our RPI we use the image from [@alexellisuk](https://twitter.com/alexellisuk), who offers the image ```alexellis2/visualizer-arm:latest```. This will run the visualizer on an ARMv6 or ARMv7 device.

```
$ docker run -it -d -p 8080:8080 -e HOST=192.168.0.200 -v /var/run/docker.sock:/var/run/docker.sock alexellis2/visualizer-arm:latest
```

![Ping Pong Services](https://github.com/FuriKuri/rpi-docker-swarm/raw/master/ping-pong.png "Ping Pong Services")

## Task List
- [ ] Add a service for game score visualisation 
- [ ] Add instance id in logs to differentiate the client responses