---
title: "Say Hello to Docker Swarm"
date: 2017-10-04T11:27:57+03:30
draft: false
---
# Say Hello to Docker Swarm (part 1)
In this tuturial I want to show you, how you can create cluster of machines using [Docker Swarm](https://docs.docker.com/engine/swarm/) and 
how to run your services on docker swarm.I assume you have basic knowledge of Docker.On part 2 (that will poblish soon), we will implement our services on Golang, and 
will run them on docker swarm, so stay tuned.

### Install Docker Machine
In this tutorial I use [docker-machine](https://docs.docker.com/machine/overview/) to simulate physical machines.First you must 
install docker-machine.Installation is pretty straightforward.Follow this link [Install Docker Machine](https://docs.docker.com/machine/install-machine/)  

### Start Docker Machines
Let's start 3 docker machine. Start your terminal and type these commands:
```
docker-machine create --driver virtualbox node1
docker-machine create --driver virtualbox node2
docker-machine create --driver virtualbox node3
```
Check that all 3 machines up and running by:
```
docker-machine ls
```
```
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
node1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.09.0-ce   
node2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.09.0-ce   
node3   -        virtualbox   Running   tcp://192.168.99.102:2376           v17.09.0-ce   
```

### Create cluster
In docker swarm there are two types of node : Manager and worker.In this demo, one of these node is act as a manager node and the two other 
act as worker nodes.In production envrionment you probably need more than one manager node for high availability.
#### Manager nodes
To become node1 as a manager, type these commands:
```
eval $(docker-machine env node1)
docker swarm init --advertise-addr=$(docker-machine ip node1)
```

#### Worker nodes
To join other nodes as worker to the cluster type these commands:
```
NODE1_IP=$(docker-machine ip node1)
WORKER_TOKEN=$(docker swarm join-token --quiet worker)
eval $(docker-machine env node2)
docker swarm join --token ${WORKER_TOKEN} ${NODE1_IP}:2377
eval $(docker-machine env node3)
docker swarm join --token ${WORKER_TOKEN} ${NODE1_IP}:2377
```
Verify that the cluster is up and running by :
```
eval $(docker-machine env node1)
docker node ls
```
```
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
21mppgndfyk916c0vsbcloigg    node2     Ready   Active        
38k2gjdu88ac7zrhy4eqls967 *  node1     Ready   Active        Leader
wgzt7w359eewjc6pqmw1kyfmb    node3     Ready   Active        
```
Notice that node1 is the leader of cluster.Worker nodes can't be used to view or modify cluster state, that 's why before I issue *'node ls'* command, I select node1. Ok, until now we have created a cluster of machines using docker swarm.Let's run a service on this cluster.I want to start 2 instance of 
Nginx on this cluster but you can run any service that you want :
```
docker service create --name nginx --replicas 2 --publish 8080:80 nginx
```
It pulls nginx image if it does not exist and after that it runs Nginx on 2 of 3 nodes.Verify that your service is up and running by:
```
docker service ps nginx
```
```
ID            NAME     IMAGE  NODE   DESIRED STATE  CURRENT STATE          ERROR  PORTS
vyuxtfvi9m3u  nginx.1  nginx  node2  Running        Running 8 seconds ago         
wam2d8heo6mg  nginx.2  nginx  node1  Running        Running 8 seconds ago         
```
You can see that on my computer it runs on node2 and node1.Let's double check that our service is running by sending http requets 
to nginx on nodes:
```
curl $(docker-machine ip node1):8585
```
In your response you must see : "Welcome to nginx!". Send request to other nodes as well:
```
curl $(docker-machine ip node2):8585
curl $(docker-machine ip node3):8585
```
Did you notice that even there is no Nginx service on node3,we can still get response from Nginx? Thanks to swarm **routing mesh**.
The routing mesh enables each node in the swarm to accept connections on published ports for any service running in the swarm, 
even if there’s no task running on the node.For more information click [here](https://docs.docker.com/engine/swarm/ingress/).

#### Leave nodes from cluster
What will happen if you remove any node from the cluster that runs our service? For example what would happen if I remove node2 from the cluster? Remember that, on my machnie, node2 is responsible for running Nginx.Let's remove node2 from the cluster:
```
eval $(docker-machine env node2)
docker swarm leave
```
Now check the state of our Nginx service:
```
ID            NAME         IMAGE  NODE   DESIRED STATE  CURRENT STATE           ERROR  PORTS
t3cy4lzq6yg3  nginx.1      nginx  node3  Running        Running 16 seconds ago         
vyuxtfvi9m3u   \_ nginx.1  nginx  node2  Shutdown       Running 37 seconds ago         
wam2d8heo6mg  nginx.2      nginx  node1  Running        Running 4 hours ago            
```
Amazing! Nginx now is running on node3. Swarm understood that nginx service is under replicated and runs another instance of Nginx on another node.

#### Remove the service
Remove Nginx service by:
```
docker service rm nginx
```
#### Stop and remove nodes
If you 've done, you can stop and remove docker machines:
```
docker-machine stop node1 node2 node3
docker-machine rm node1 node2 node3
```
## Conclusion
In this tutorial we 've learnt how to create 3 machines,join them to Swarm cluster and ask the Swarm to run our services on those machines.
The beauty of swarm is that you are not worry where your tasks are running.You can leverage swarm routing mesh, to route requests to 
our services.In the next part, we 're going to implement our services in Go, and run them on the cluster.

