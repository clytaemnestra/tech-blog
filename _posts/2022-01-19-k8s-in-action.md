---
layout: post
title: "K8s In Action"
description: "Notes from Books: K8s In Action"
date: 2022-01-19
tags: [books, infrastructure]
---

# Kubernetes Resources

## Pod
* all containers of a pod are on the same node -> a pod never spans two nodes
* why pods and not individual containers? because the principle of Docker & K8s is one process = one container and it's necessary to have something above containers that'd group containers that host the same app
* pod of containers allows you to run closely related processes together and pro-
vide them with (almost) the same environment as if they were all running in a single
container, but it also keeps them separated 
* all containers are located in the same set of Linux namespaces -> they share the same IP address and port space


### Logs
* `docker logs <container id>`
* `cubectl logs <pod name> -c <container name>` 

### Replication & Controllers 
* liveness probe 
    * mechanism to check if pods are healthy
    * part of YAML
* replication controller (or replica set) -> now deployment
    * checks, how many replicas of pods are
    * if a pod disappeares, it replication controller creates a new one
* horizontal scalling
    * set up higher amount of pods through command line `$ kubectl scale rc kubia --replicas=10` or directly in the YAML

## Service
* single, constant point of entry to a group of pods providing the same service
* points to pods using labels
* service has a static IP address, which never changes
    * when POD gets killed/deleted/whatever, ReplicaController creates a new one, which has new IP
        * service solves issue with pods changing IP addresses and makes sure that pods receive connection
        * multiple pods may provide the same service, hence the same IP is needed (in this case, the service IP)
* for a service to be accessible from outside, its type has to be set up as NodePort or LoadBalancer

### Types Of Services
* ClusterIP 
    * accessible only from inside the cluster
* NodePort
    * static port on each node's IP
* LoadBalancer
    * exploses the service through external load balancer
* ExternalName
    * accessible through CNAME value

## Volumes
* component of pod
* a volume is available to all containers in the pod, but it must be mounted in each container that needs to access it
* volumes are used to share data accross containers

# Docker
* images are template to create containers, containers are instances created from images 
* images are created from another images, two different images can have same parent image as their base
* check additinal information about containers:
    * enter the container -> `docker inspect <container name>`
    * in the continer -> `ps aux`