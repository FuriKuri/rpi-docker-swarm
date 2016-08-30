# Docker Swarm on Raspberry PI

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
