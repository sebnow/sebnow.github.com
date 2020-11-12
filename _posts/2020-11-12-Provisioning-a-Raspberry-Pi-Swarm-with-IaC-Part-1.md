---
layout: post
title: "Provisioning a Raspberry Pi Swarm with IaC: Part 1"
categories: [tech]
---

The Raspberry Pi is a cheap and fun piece of kit for geeks. My QNAP NAS
is running a few too many applications on Docker, and I have a few
Raspberry Pi's laying around, so I thought it'd be a fun weekend project
to turn them into a Docker Swarm.


## The Swarm

Before getting into the automation side of things, let's set up
a prototype of the swarm itself. I'll be setting up;

  * A Raspberry Pi Rev 1 Model B,
  * A Raspberry Pi Rev 3 Model B,

The Raspberry Pi's will be running [Raspberry Pi OS][] Buster
(2020-08-20). While the Rev 3 is 64 bit, I'll be using the `armhf` on
both. All the machines are on the same network.

Follow the official [instructions][raspbian-install] to burn the image
to the SD cards and spin up the Pi's. We'll be needing Docker, so follow
[the instructions][docker-install] for installing that too.

The Docker Swarm has two types of nodes; managers and workers. The
managers keep the state of the swarm in a distributed store by using the
[Raft Consensus Algorithm][raft]. For best results, the amount of
managers should be odd. I only have three machines to work with, so I'll
stick with a single manager for now. This cluster doesn't need to be
highly available. The Rev 3 is the most powerful RPi, so I'll be
electing it as the manager.

```
pi@manager:~$ sudo docker swarm init
Swarm initialized: current node (eseiud89bm9gl3eail1tnnznx) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-6814zqm6wpp4q43fe9a9m2ezxcsecjnlmvcayvm5ydzpwkegdh-4n5qaw4ujge9xcmi2vcr0mcrn 192.168.0.2:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
```
pi@manager:~$ sudo docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
eseiud89bm9gl3eail1tnnznx *   manager             Ready               Active              Leader              19.03.13
```

Docker is nice enough to provide instructions for the workers. Let's do
that.

```
pi@worker:~$ sudo docker swarm join --token SWMTKN-1-6814zqm6wpp4q43fe9a9m2ezxcsecjnlmvcayvm5ydzpwkegdh-4n5qaw4ujge9xcmi2vcr0mcrn 192.168.0.2:2377
This node joined a swarm as a worker.
```

All management commands need to be run on the manager, so let's check
the list of nodes;

```
pi@manager:~$ sudo docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
eseiud89bm9gl3eail1tnnznx *   manager             Ready               Active              Leader              19.03.13
bcm9cnme7ewvx224klfun1qfn     worker              Ready               Active                                  19.03.13
```

Now let's run an application on the swarm. For demonstration purposes
let's just use some random image, like `nginxdemos/hello`.

```
pi@manager:~$ sudo docker pull nginxdemos/hello
Using default tag: latest
latest: Pulling from nginxdemos/hello
550fe1bea624: Pull complete
d421ba34525b: Pull complete
fdcbcb327323: Pull complete
bfbcec2fc4d5: Pull complete
0497d4d5654f: Pull complete
f9518aaa159c: Pull complete
a70e975849d8: Pull complete
Digest: sha256:f5a0b2a5fe9af497c4a7c186ef6412bb91ff19d39d6ac24a4997eaed2b0bb334
Status: Downloaded newer image for nginxdemos/hello:latest
docker.io/nginxdemos/hello:latest
```
```
pi@manager:~$ sudo docker service create -p 8080:80 nginxdemos/hello:0.2
image nginxdemos/hello:0.2 could not be accessed on a registry to record
its digest. Each node will access nginxdemos/hello:0.2 independently,
possibly leading to different nodes running different
versions of the image.

9ldqvus6y2zzp64pj187mmb52
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```
```
pi@manager:~$ sudo docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                  PORTS
9ldqvus6y2zz        jolly_rhodes        replicated          1/1                 nginxdemos/hello:0.2   *:8080->80/tcp
```

```
pi@manager:~$ sudo docker service ps jolly_rhodes
ID                  NAME                IMAGE                  NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
hfebnhjrg0v7        jolly_rhodes.1      nginxdemos/hello:0.2   manager             Running             Running 7 minutes ago
```

Fantastic. We have a service running in the swarm. Let's see if it
works;

```
pi@manager:~$ links -dump http://localhost:8080
   NGINX Logo

   Server address: 10.0.0.35:80

   Server name: 2c704cbf842b

   Date: 12/Nov/2020:19:53:43 +0000

   URI: /

   [ ] Auto Refresh
                  Request ID: e620aa94f6e13add9a69cd03340b1551
                               © NGINX, Inc. 2018
```

Now you can do all the typical things with Docker Swarm; adding
replicas, deploying stacks and adding more nodes. In the next post we'll
go through how this can be automated using Ansible to easily scale to
more nodes.


[raspbian-install]: https://www.raspberrypi.org/documentation/installation/installing-images/
[docker-install]: https://docs.docker.com/engine/install/debian/
[Raspberry Pi OS]: https://www.raspberrypi.org/software/operating-systems/
[raft]: https://docs.docker.com/engine/swarm/raft/
