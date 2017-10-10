---
title: "Docker Swarm (Part 2)"
date: 2017-10-08T11:50:42+03:30
draft: false
---
# Introduction
In the [previous post](https://mostafa-asg.github.io/posts/say-hello-to-docker-swarm/), we 've learned how to create a cluster of 
machines using *Docker Swarm*, and how to ask Swarm to run 2 instances of Nginx on those machines.In this post, we will learn how to run 
our custom applications on the cluster.  
We 're going to implement two services using Go, one of them is *fontend* service and the other 
is *backend* service. The user can interact only with *frontend* service.*Frontend* uses *Backend* service to serves the requests.
*Frontend* service is a web application that showes a html page to the user.User can type his name on the textbox and triggers a request.When *frontend* 
accepts the request from the user, it sends a requets to *backend* service with the user's provided name,and shows the response of the *backend* service to the user.  
The diagram below depicts the intraction model:
![services](/static/say-hello-to-docker-swarm-2/services.png#center)  

## Create swarm cluster
Let's create a cluster of 3 machines using [docker machine](https://docs.docker.com/machine/overview/):
```
docker-machine create --driver=virtualbox node1
docker-machine create --driver=virtualbox node2
docker-machine create --driver=virtualbox node3

NODE1_IP=$(docker-machine ip node1)

eval $(docker-machine env node1)
docker swarm init --advertise-addr=$NODE1_IP
MANAGER_TOKEN=$(docker swarm join-token --quiet manager)
WORKER_TOKEN=$(docker swarm join-token --quiet worker)

eval $(docker-machine env node2)
docker swarm join --token ${WORKER_TOKEN} ${NODE1_IP}:2377

eval $(docker-machine env node3)
docker swarm join --token ${WORKER_TOKEN} ${NODE1_IP}:2377
```
make sure that we send requests to manager node by:
```
eval $(docker-machine env node1)
```
For more information on creating docker swarm cluster, check the [previous article](https://mostafa-asg.github.io/posts/say-hello-to-docker-swarm/).

## Backend Service
Our *Backend* service, is very simple.It is just a greeting service.It gives your name as part of a url, and return a message like this:
```
Welcome {{name}} - Response from : {{ip}}
``` 
which {{name}} is a name that comes with url, and {{ip}} is the container ip address of *Backend* service.  
Create a folder called backend:
```
mkdir backend
cd backend
```
Save these lines of code as backend.go :
```
package main

import (
	"net"
	"errors"
	"net/http"
	"fmt"
)

func main(){

	http.HandleFunc("/greeting/" , handle)
	http.ListenAndServe(":7070",nil)

}

func handle(w http.ResponseWriter , r *http.Request) {
	name := r.URL.Path[ len("/greeting/"): ]
	ip , err := getIPAddress()
	if err != nil {
		ip = "Cannot find any IP address"
	}
	fmt.Fprintf( w , "Welcome %s - Response from : [%s]" , name , ip )
}

func getIPAddress() (string, error) {
	ifaces, err := net.Interfaces()
	if err != nil {
		return "", err
	}
	for _, iface := range ifaces {
		if iface.Flags&net.FlagUp == 0 {
			continue // interface down
		}
		if iface.Flags&net.FlagLoopback != 0 {
			continue // loopback interface
		}
		addrs, err := iface.Addrs()
		if err != nil {
			return "", err
		}
		for _, addr := range addrs {
			var ip net.IP
			switch v := addr.(type) {
			case *net.IPNet:
				ip = v.IP
			case *net.IPAddr:
				ip = v.IP
			}
			if ip == nil || ip.IsLoopback() {
				continue
			}
			ip = ip.To4()
			if ip == nil {
				continue // not an ipv4 address
			}
			return ip.String(), nil
		}
	}
	return "", errors.New("are you connected to the network?")
}
```
To test your *backend* service first run the program :
```
go run backend.go
```
On another terminal send request to *backend* service :
```
curl http://localhost:7070/greeting/Mostafa
```
After */greeting/* you can type any name. You must see the desired output.
### Creating docker image
To run *backend* service inside a container, we need *docker image* of our backend service.To create *docker image* we need **Dockerfile**. So create a file named *Dockerfile* inside the backend folder with these lines:
```
FROM ubuntu:14.04

COPY backend /myapp/

WORKDIR /myapp

EXPOSE 7070

ENTRYPOINT ["./backend"]
```
To build docker image type these lines:
```
go build backend.go
docker build -t localhost:5000/backend:1 .
```
The first line, compiles the application and creates a executable file called *backend*. The second line builds the docker image. It creates a image named **backend** with version(tag) 1. *localhost:5000* is crucial here.
It indicates the image *repository*. Later we will push this image to this repository. We will create this repository on this address later. Check that image has created successfully by :
```
docker images
```
```
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
localhost:5000/backend   1                   79fb7460e002        5 seconds ago       194 MB
```
**NOTE :** This is not a minimal docker image. To reduce the size of the created image check this [article](https://blog.codeship.com/building-minimal-docker-containers-for-go-applications/)
## Frontend Service
Create a folder called *frontend* outside the *backend* folder.
```
cd ..
mkdir frontend
cd frontend
```
Save these lines of code as frontend.go :
```
package main

import (
	"io/ioutil"
	"fmt"
	"html/template"	
	"net/http"
	"net"
	"errors"
)

type Model struct {
	IPAddress string
}

var templates = template.Must( template.ParseFiles("index.html") )

func indexHandler(w http.ResponseWriter , r *http.Request) {

	ip , err := getIPAddress()
	if err != nil{
		ip = "?.?.?.?"
	}

	model := Model { IPAddress:ip, }
	templates.ExecuteTemplate( w , "index.html" , model )
}

func greetingHandler(w http.ResponseWriter , r *http.Request) {

	resp , err := http.Get("http://backend:7070/greeting/" + r.FormValue("fullname"))
	if err != nil {
		fmt.Fprintf(w , "Error \n%s" , err.Error() )	
		return
	}	
	defer resp.Body.Close()
	
	ba , _ := ioutil.ReadAll(resp.Body)

	fmt.Fprintf(w , string(ba) )
}

func getIPAddress() (string, error) {
	ifaces, err := net.Interfaces()
	if err != nil {
		return "", err
	}
	for _, iface := range ifaces {
		if iface.Flags&net.FlagUp == 0 {
			continue // interface down
		}
		if iface.Flags&net.FlagLoopback != 0 {
			continue // loopback interface
		}
		addrs, err := iface.Addrs()
		if err != nil {
			return "", err
		}
		for _, addr := range addrs {
			var ip net.IP
			switch v := addr.(type) {
			case *net.IPNet:
				ip = v.IP
			case *net.IPAddr:
				ip = v.IP
			}
			if ip == nil || ip.IsLoopback() {
				continue
			}
			ip = ip.To4()
			if ip == nil {
				continue // not an ipv4 address
			}
			return ip.String(), nil
		}
	}
	return "", errors.New("are you connected to the network?")
}

func main() {

	http.HandleFunc("/",indexHandler)
	http.HandleFunc("/greeting",greetingHandler)
	http.ListenAndServe(":8585", nil)

}
```
It requires *index.html*. Save these lines as index.html:
```
<html>
    <body>
        <h1>{{ .IPAddress }}</h1>
        <form action="/greeting" method="post">
            Your name : <textarea name="fullname" ></textarea><br />
            <input type="submit" value="Send"  />
        </form>
    </body>
</html>
```
It uses [Go html templates](https://golang.org/pkg/html/template/) to render the output. Notice the {{ .IPAddress }}. It is the IP address of *frontend* service. I did this on purpose to show you 
each of these services run on different nodes. As same as *backend* service, this service also needs Dockerfile to build the *docker image*. Save these lines as Dockerfile :
```
FROM ubuntu:14.04

COPY frontend /myapp/
COPY index.html /myapp/

WORKDIR /myapp

EXPOSE 8585

ENTRYPOINT ["./frontend"]
```
To build docker image type these lines:
```
go build frontend.go
docker build -t localhost:5000/frontend:1 .
```
Now if you list your images, you must have two images:
```
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
localhost:5000/frontend   1                   14599282b8ca        3 seconds ago       196 MB
localhost:5000/backend    1                   79fb7460e002        15 minutes ago      194 MB
```
## Create Local Docker Repository
Until now, we have two images on **node1** only. But we have a cluster of 3 machines.If you run a service using docker swarm,each node must get the docker image before it can run the service.
We must create a local repository using *docker service*. Run this command:
```
docker service create --name registry --replicas 1 -p 5000:5000 registry:2
```
Check it is successfully running by :
```
docker service ps registry
```
```
ID            NAME        IMAGE       NODE   DESIRED STATE  CURRENT STATE          ERROR  PORTS
u28ffx31hzpw  registry.1  registry:2  node2  Running        Running 7 seconds ago    
```
Make sure that **CURRENT STATE** is 'Running'. After that push the images to this registry:
```
docker push localhost:5000/backend:1
docker push localhost:5000/frontend:1
```
In this state you are sure that each node can access this repository to pull the images. How? because we ran this registry as a service on swarm mode, all requests on port 5000 
on each node will be redirected to this registry service using [Swarm routing mesh](https://docs.docker.com/engine/swarm/ingress/).
## Docker Compose
Create a file called *compose.yml* with these lines:
```
version: '3'
services:
  backend:
    image: localhost:5000/backend:1
    deploy:
      replicas: 1
  frontend:
    image: localhost:5000/frontend:1
    ports:
      - "8585:8585"
    depends_on:
      - backend
    deploy:
      replicas: 2
```
I want *backend* service only be visible to *frontend* service and users cannot directly interact with *backend* service. I want users only interact with *frontend* service, so I published the frontend service port, but I did not publish *backend* service port. To make *backend* service visible to *frontend* service, I use **depends_on** keyword in docker compose.  
Now we are ready to run our services. Becasue we are on swarm cluster, instead of using *docker-compose up* , we should use *docker stack deploy* :
```
docker stack deploy --compose-file compose.yml myservices
```
Check that all replicas of your services is up and running:
```
docker stack ps myservices
```
```
ID            NAME                   IMAGE                      NODE   DESIRED STATE  CURRENT STATE          ERROR  PORTS
l798nv6r4nza  myservices_frontend.1  localhost:5000/frontend:1  node1  Running        Running 9 seconds ago         
2yme5txy4ijv  myservices_backend.1   localhost:5000/backend:1   node3  Running        Running 1 second ago          
ndvpv5mq697y  myservices_frontend.2  localhost:5000/frontend:1  node2  Running        Running 9 seconds ago         
```
Notice that on my computer, *frontend* service is not running on node3, however I can still send requests on port 8585 on this node.Thanks to the [swarm routing mesh](https://docs.docker.com/engine/swarm/ingress/). To test services functionality you must open your browser and navigate to {{NODE_IP}}:8585. You cand find node ip addresses by these commands
```
docker-machine ip node1
docker-machine ip node2
docker-machine ip node3
```
Replace {{NODE_IP}} with appropriate ip address and have fun with **swarm cluster** :)
