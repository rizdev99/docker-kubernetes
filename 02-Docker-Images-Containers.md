# **Docker Images & Containers: The Core Building Blocks**
<br>

## Images & Containers
* Images: Templates / Blueprint for running the containers, Contains code + required tools.
* Containers: The running "unit of software", Multiple containers can be created based on one image.

We run containers that are based on images. With a single image we can spin up multiple containers.

## Using & Running Pre-Built (External) Images
External images can be found from [Docker Hub](https://hub.docker.com/)  
To run a container from the external images - ```docker run node``` This will check if the node image is present locally. If not then it will download it and then run the container.  The container will run and stop immediately since it is not doing any work except for providing an environment. To expose the interactive shell (REPL) use ```docker run -it node```
All running containers - ```docker ps -a``` 

## A NodeJS App: Create your own custom image
Write your own image on the top of a base image.   
Dockerfile - A keyword that would be identified by docker. It contains instructions that will be executed to build and image.  

### Dockerfile
It typically starts from the   
```FROM <image-name>``` Instruction. It allows you to build your image on top of another image. The image name should exist on your system or be publicly available on docker hub under the same name.  
<br>
```COPY <path-outside-the-image> <path-inside-the-image>``` - It copies the file from our local file system to the file system of the image. Since the container (and also the image) also contains the code, It should also be copied into the image.  
<br>
```COPY . .```  - Copy everything from current local folder to image's current folder except the Dockerfile.   
Every Image/Container has its own file system that is completely detached from the host's file system.   
<br>
```COPY . /app``` - Since it is not a good idea to copy code into the root folder of the container.  
<br>
```WORKDIR /app``` - By default the working directory is the root directory and all the commands are executed in the working directory. Change the working directory to app.  
<br>
```RUN npm install``` - This command will be ran while building the image. It downloads all the dependencies for running the node application.  
<br>
```EXPOSE 80``` - Since the containers are isolated from our local environment and our application is running on port 80. We need to configure docker image to expose the port 80 of the docker container to the host network.  
<br>
```CMD ["node", "server.js"]``` - This command will be ran while running the container. It will start the node app.

Dockerfile
```
FROM node
WORKDIR /app
COPY . /app/
RUN npm install
EXPOSE 80
CMD ["node", "server.js"]
```

## Running a Container based on our custom image.

```docker build <location-of-the-dockerfile>``` - Build our docker image  
```docker run <image-id>``` - Running the container  
```EXPOSE 80``` - This command is optional but recommended since it is only for the documentation purposes. It does not do anything functional. To expose a container's port to the host - ```docker run -p <host-port>:<container-port> <image-id>``` where p stands for publish.  
<br>
For all docker commands where an ID can be used, you don't always have to copy / write out the full id.
You can also just use the first (few) character(s) - just enough to have a unique identifier.

## Images are Read-Only!
Images are basically locked & finished. Everything in the images is read-only so it cannot be edited from outside like code changes. It order to reflect the code changes we need to rebuild the image.

## Understanding Images Layers
Every instruction in the Dockerfile is used to cache an image layer build on top of one another and it is stored like it. At last the container layer is put on top of all the image layers and it is spun up.
At the time of rebuilding an image, If any file in an instruction has been changed, Docker rebuilds the layers of that instruction and the ones that follows it. The layers before that instructions are build from the cached and are super fast.  
<br>
Every Instruction in an image creates a cacheable layer - layers help with image re-building and sharing.
Minor Optimization in Dockerfile.
```
FROM node
WORKDIR /app
COPY package.json /app/
RUN npm install
COPY . /app/
EXPOSE 80
CMD ["node", "server.js"]
```
By this optimization if the code has been changed the npm install command wont be ran again and again.

## Images & Containers - First Summary
 Container does not copy the code It utilizes the image to run the code. That is how Docker manages it.

## Managing Images & Containers
with any docker command we can use --help to see all the option. What all we can do with images and containers.
| Images | Containers |
| --- | --- |
Can be tagged (named)| Can be named
Can be listed | Can be configured in detail
Can be analyzed | Can be listed
Can be removed | Can be removed  

<br>
<br>  

## Stopping & Restarting Containers
```docker --help``` - To see all the docker commands, But it always boils down to some of the core commands.   
```docker ps -a``` - All containers including the stopped containers  
```docker run <image-id>``` - Run a new container from the image. The container runs in foreground. attached mode is the default.  
```docker start <container-id>``` - Restart the existing docker container. This command won't block the terminal as run command did. It starts the container in the background. detached mode is the default.  
```docker run -p <host-port>:<container-port> -d <image-id>``` - Use the d flag to run the container in the detach mode. You can attach yourself to a detached container again by running ```docker attach CONTAINER_NAME```  
```docker logs -f <container-name>``` - See the logs that were printed by the container. Can use -f flag to follow.  
```docker start -a <container-id>``` - Attach to the container  
<br>
By default, if you run a Container without -d, you run in "attached mode".  
If you started a container in detached mode (i.e. with -d), you can still attach to it afterwards without restarting the Container with the following command:  
docker attach CONTAINER
attaches you to a running Container with an ID or name of CONTAINER.

## Entering the Interactive Mode
To start a container in interactive mode use the -it flag. i = interactive STDIN open and t = Assign a pseudo terminal. There will be many times when we need to interact with the docker container.

## Deleting Images & Containers
To remove a running container we have to first stop that container and then use rm command. ```docker rm multiple-container-name```. We can also use ```docker container prune``` to remove all the stopped container.   
```docker images``` - List the images and then ```docker rmi image-id``` to remove the images. We can only remove the images that are not been used by containers including the stopped containers. To remove all the images that are not being used by any container use ```docker image prune``` 
<br>  
Removing Stopped Containers Automatically  
use ```--rm``` to delete the container automatically after it has been stopped. 

## BTS Inspecting Images
The container is built up on the images and is not copied. Multiple container running the image will share the code of the image.   
```docker image inspect image-id``` - To inspect the image. 

## Copying Files Into & From a Container
There will be scenario's where you need to copy a file from the container to your local system or vice-versa.   
```docker cp <source> <destination>``` can do that. It is not recommended to copy a file in case of code changes.  
```docker cp dummy/. peaceful_morse:/test``` - Copy from local machine to the container.  
```docker cp peaceful_morse:/test/test.txt dummy/``` - Copy from the container to the local machine.  
If the folder does not exist in the container then It'll be created.

## Naming & Tagging Containers and Images.
```docker run -p 3000:80 -d --rm --name goalsapp 5c97815c5303``` - To name a container.  
### Understanding Image Tags
name : tags.  
- name - Defines a group of, possible more specialized, images. Example: node. 
- tag - Defines a specialized image within a group of images. Example 14.
Combined we get a unique identifier.

```docker build -t goals:latest .``` - Tag an image.
<br>   

```docker image prune -a``` - If you want to remove all images, including tagged images.  
We can specify images with Tag in the FROM command.

## Sharing Images - Overview
Everyone who has the images can create the container based on the image!  
1. Share the Dockerfile 
    - Simply run docker build .
    - The Dockerfile instructions might need surrounding files and folders and the source code.
2. Share a Built Image
    - Download an image, run a container based on it.
    - No build step required everything is build in the image.

## Pushing images to DockerHub
Sharing via DockerHub or Private Registry
1. DockerHub (Free) - Official Docker Registry; Public, Private & Official Images
2. Private Registry - Any provider/registry you want to use. Only your own (or team) images. <br><br>

Images can be shared with the in built docker commands.  
Share - ```docker push IMAGE_NAME```  
Use  -  ```docker pull IMAGE NAME```


If we want to share on private registry we need to include the URL.
In order to push the image to the dockerhub we need to change the tag of it using ```docker tag nodeapp:latest rizdev99/node-hello-world```.  
This will create a copy of it with the updated tagname.

You need to login from the terminal in order to push the image onto docker hub. ```docker login```.

Docker does not pushes the entire image rather it sees if we are using any pre-built image from docker hub and attach itself saving space & bandwidth.

## Module Resource
Attached, you find the code / project setup snapshots for this module.

You also find a cheat sheet / short summary for this module attached.

Also consider diving into the official docs, in addition to this course: https://docs.docker.com/

- [slides-images-containers](https://drive.google.com/file/d/1-Warabda8uVPrpcNzPf7qqPgc7xdx1hX/view?usp=sharing)
- [cheatsheet-images-containers](https://drive.google.com/file/d/1fHCAzj2YaujAmO0dLz4lKNcOLJma3LqG/view?usp=sharing)
