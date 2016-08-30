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

## Start Services


