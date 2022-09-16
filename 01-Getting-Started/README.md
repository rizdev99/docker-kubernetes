# **Getting Started**

## What is Docker?
> Docker is a tool for creating and managing *containers*. 

A Container is a package of code and dependencies to run that code (eg NodeJS code + NodeJS runtime).  
The same container will always yield the same application and execution behavior. No matter by whom it might be executed. 

## Why Containers?
### Why we want independent, standardized "application packages"?
- We want to build and test in exactly the same environment as we later run our app in.
- Every team member should have the same development environment.
- Clashing tools / version between different projects  

We lock our environments for different projects using docker.

## Virtual Machines vs Docker Containers
### Can VMs not solve the problems that Docker is solving?
Yes, to an extend they can but, It costs the overhead of the Virtual OS. Waste a lot of space on your hard drive and tends to be slow.  
Pros | Cons
--- | ---
Separated Environments | Redundant duplication, waste of space
Environment-specific configurations are possible | Performance can be slow boot time can be long
Environment configuration can be shared and reproduced reliably | Reproducing on another computer server is possible but may still be tricky
<br>

So VMs solve the problem but not perfectly. That is why we have containers. Docker is just a defacto standard tool for creating and managing containers.

### How does docker & containerization solves this problem?
It uses the OS of the host. Runs docker engine on top of that. 
Docker engine can then run containers. We can easily create an image of the container and share it with others.

### Containers vs VMs
Containers | Virtual Machines
--- | ---
Low impact on OS, very fast, minimal disk space usage. | Bigger impact on OS, slower, higher disk space usage
Sharing, re-building and distribution is easy | Sharing, re-building and distribution can be challenging
Encapsulates apps/environments instead of "whole machines" | Encapsulates whole machines instead of "apps/environments"
<br>

## Docker setup - Overview
Refer [Docker Home](https://www.docker.com/)  
If requirements are met for non-linux OS, Install Docker Desktop otherwise Install docker toolbox. On linux Docker engine is natively supported.

If you can't install Docker on your system, you can also look into this online playground: [Docker Playground](https://labs.play-with-docker.com/)

## An Overview of the Docker tools
### Building Blocks - 
1. Docker Engine - Hosts linux kernel
2. Docker Desktop - Daemon & CLI 
    - Daemon - A program that keeps running and ensure docker is up.
3. Docker Hub - Service that allows us to share our images with the internet
4. Docker Compose - On top of docker. Makes it easier to manage complex docker containers

## Installing an Configuring an IDE
Install VS Code with the following recommended extensions
- Docker
- Prettier

## Getting Our Hands Dirty
Containers are always based on images. 
Take the project example and run the following commands
1. docker build . -> Takes the Dockerfile and creates an image
2. docker run -p 3000:3000 <image_id> -> To load the image and run the container
3. docker stop <container-nick-name> -> To stop the container 

## Course Outline

### 1. Foundation
1. Images & Containers
2. Data & Volumes (in Containers)
3. Containers & Networking

### 2. Real Life
1. Multi-Container Projects
2. Docker-Compose
3. "Utility Container"
4. Deploy Docker Containers on AWS

### 3. Kubernetes
1. Kubernetes: Introduction & Basics
2. Kubernetes: Data & Volumes
3. Kubernetes: Networking
4. Kubernetes: Deploying a K8s Cluster

## Resources:
1. [slides-getting-started.pdf](https://drive.google.com/file/d/1HphcXXz0IFgsyTq1SmLFn68EgorYn1t3/view?usp=sharing)
2. [first-demo-dockerrized.zip](https://drive.google.com/file/d/10eI7ZPWI87ZX3e2VQo3WvlsNORO8n0rD/view?usp=sharing)
