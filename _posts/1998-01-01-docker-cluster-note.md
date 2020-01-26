---
layout: post
title:  "Docker technical notes - container cluster"
date:   2020-01-24 00:00:00 -0500
categories: tech-docker-container
---

## Ansible / docker-machine / Kubernetes tools 

Each of the tool creates and manages a cluster of docker VM machines.

- install docker engine libs and confs on cluster nodes. 
- upgrade docker installations.
- monitor docker VMs.

**AWS ECS**:  Elastic Container Service. It is a highly scalable, high performance container orchestration service. It supports Docker container. It allows you to easily deploy, run and scale containernized applications on AWS. A container instance is actually a EC2 instance running on ECS agent. 

When you have a containerized application, it’s important to be able to easily deploy them in the cloud, not only running them locally using Docker for Mac/Windows or from a Linux box locally. 

docker-machine tool is used to create remote virtual machines (VM) easily and manage those containers runnong on VM machines. In short, it allows you to control the docker agent on a remote VM. 

docker-machine --help
docker-machine ls
docker-machine stop default
docker-machine start default
docker-machine env default --shell <cmd>
docker-machine ssh <docker machine instance>
	> then on the same terminal, run "sudo <cmd>"
	

