---
layout: post
title:  "Docker technical notes - docker-composer"
date:   2020-01-24 00:00:00 -0500
categories: tech-docker-container
---

## docker-composer tool

The tool creates and manages a stack of docker services.

use 'docker-compose' command to starts / stop service. it can also be used to build images. It needs a compose yaml file, by default is docker-compose.yml file

sample files:

	C:\UserData\capco\cardinal-model\docker-compose.yml
	C:\UserData\capco\cardinal-infinispan\app.yml
	C:\UserData\cibc-api\reference-rest-api\compose\docker-compose-oracle.yml
	C:\UserData\cibc-api\reference-rest-api\compose\docker-compose-cibcapi.yml

### build or rebuild image for a service

	docker-compose build <service-name> 	
	
	e.g. C:\UserData\capco\cardinal-model\docker-compose.yml
		build:
		  context: .
		  dockerfile: cardinal-db-bootstrap-docker/Dockerfile

### create and/or start a service and its containers from a docker-compose yaml file. 

--no-start : do not start containers.
	
	docker-compose [-f <docker-compose yaml file>] up [--no-start] -p \<project|app name> [\<service-name>]

### stop a service. 

remove its containers, volumes and networks, but not images. 		
By default, anonymous volumes attached to containers will not be removed. You can override this with `-v`
  
	docker-compose [-f <docker-compose yaml file>] down -v -p \<project|app name> [\<service-name>]	
	
### start, restart or stop an existing service and its containers

	docker-compose start|restart|stop \<service-name>  
	docker-compose rm \<service-name>  // remove stopped service and its containers
	
  **Removing the service does not remove any volumes created by the service. Volume removal is a separate step.**

### use "docker-compose run" or "docker run" to run a one-off command on a container. 

the command line options will override 		docker-compose yaml service config.

	docker-compose [-f <docker-compose yaml file>] run [options] \<service-name> [COMMAND] [ARGS...]
	e.g. 
	docker-compose run --name \<container-name> -p 1521:1521 -d \<service-name>	[COMMAND] [ARGS...]
	
Options:

    -d, --detach          Detached mode: Run container in the background, print
                          new container name.
    --name NAME           Assign a name to the container
    --entrypoint CMD      Override the entrypoint of the image.	default is /user/local/bin/<entrypoint.sh file>
    -e KEY=VAL            Set an environment variable (can be used multiple times)
    -l, --label KEY=VAL   Add or override a label (can be used multiple times)
    -u, --user=""         Run as specified username or uid
    --no-deps             Don't start linked services.
    --rm                  Remove container after run. Ignored in detached mode.
    -p, --publish=[]      Publish a container's port(s) to the docker vm host	// <Docker VM port>:<container port>
    --service-ports       Run command with the service's ports enabled and mapped
                          to the host.
    --use-aliases         Use the service's network aliases in the network(s) the
                          container connects to.
    -v, --volume=[]       Bind mount a volume (default [])	
    -T                    Disable pseudo-tty allocation. By default `docker-compose run`
                          allocates a TTY.
    -w, --workdir=""      Working directory inside the container
	
	
### You can build an image from a Dockerfile, and you can pull an image from a Docker Registry, but what happens when you supply both?  

If you add build: "." to a service that will build a Docker image out of the Dockerfile that exists in the directory when you build or up your composer file.

If you add image: "postgres:10.3-alpine" to a service that will pull down that image from the Docker Hub when you pull or up your compose file.	

** But what happens if you define both a build and image property like this: **
	web-service:
	  build: "."
	  image: "nickjj/myimage:1.0"

When the image gets built, instead of using your COMPOSE_PROJECT_NAME or folder name + service name as the name of the image it will use what you have in the image property.

In the above example you would end up with an image named and tagged as nickjj/myimage:1.0 which will be built out of the Dockerfile in the current directory. the built image is stored in docker local registery.

This is really handy because it’s basically a shortcut to tag your images so that you can quickly push them to your Docker Registry of choice, such as the Docker Hub.

