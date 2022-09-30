# Managing Data & Working with Volumes.

## Understanding data categories / Different types of Data
1. Application ( Code + Environment )
    - Written and provided by the developers
    - Added to the image in the build phase
    - "Fixed": Can't be changed once image is built
    - Read-Only hence stored in images.
2. Temporary App Data
    - Fetched / Produced in running containers
    - Stored in memory or temporary files
    - Dynamic & Changing but cleared regularly
    - Read + Write temporary, hence stored in containers
3. Permanent App Data
    - Fetched / Produced in running containers
    - Stored in files or databases
    - Must not be lost even if the container stop / restarts / deleted
    - Read + Write permanent, hence store in containers with the help of volumes

## Volumes
Volumes are folders on your host machine hard drive which are mounted (made available or mapped) into the container.  
+ Volumes persist if a container shuts down. If a container re-starts and mounts a volume, any data inside of that volume is available in the container.
+ A container can read and write data into a volume.

## Two types of External Data Storage
1. Volumes ( Managed by Docker )
2. Bind Mounts ( Managed by you )
### Volumes (Managed by Docker)
Docker sets up a folder / path on your host machine, exact location is unknown to us. Managed by ```docker volume``` commands.  
A defined path in the container is mapped to the created volume / mount.
eg. /some-path on your hosting machine is mapped to /app/data.
1. **Anonymous Volumes** - They are removed when the container shuts down because they are closely attached to a specific container. To add an anonymous volume - ```Volume: ["/app/feedback"]``` - This will map the folder /app/feedback of the container to a folder created anonymously on our host machine.
2. **Named Volumes** - They are great for data which should be persistent but which you don't need to edit directly. They do not require an entry in the Dockerfile of Volumes. ```docker run -p 3000:80 --name node-app --rm -v feedback:/app/feedback node-app:volumes```

Removing Anonymous Volumes
We saw, that anonymous volumes are removed automatically, when a container is removed.

This happens when you start / run a container with the --rm option.

If you start a container without that option, the anonymous volume would NOT be removed, even if you remove the container (with docker rm ...).

Still, if you then re-create and re-run the container (i.e. you run docker run ... again), a new anonymous volume will be created. So even though the anonymous volume wasn't removed automatically, it'll also not be helpful because a different anonymous volume is attached the next time the container starts (i.e. you removed the old container and run a new one).

Now you just start piling up a bunch of unused anonymous volumes - you can clear them via `docker volume rm VOL_NAME` or `docker volume prune`

### Bind Mounts ( Managed by you )
You define a folder / path on your host machine.  
Great for persistent, editable (by you) data (eg. source code)

```
docker run -p 3000:80 --name node-app --rm -v feedback:/app/feedback -v "/Users/rizwan/Downloads/data-volumes-03-adj-node-code:/app" node-app:volumes 
```

Also allow the file sharing of the parent folder with docker.

Just a quick note: If you don't always want to copy and use the full path, you can use these shortcuts:

macOS / Linux: -v $(pwd):/app

Windows: -v "%cd%":/app

I don't use them in the lectures, since I want to show an approach that works for everyone (and I don't want to switch between both all the time), but you can use these shortcuts depending on which OS you are working on to save some typing.

### The problem with the above approach
In the Dockerfile we are generating the node_modules folder using ```npm install``` . There is no such folder on our local machine but node_modules is present in our container. So docker overwrites our container /app folder with our localhost folder.
```
docker run -p 3000:80 --name node-app --rm -v feedback:/app/feedback -v "/Users/rizwan/Downloads/data-volumes-03-adj-node-code:/app" node-app:volumes 
```

To overcome this, We need to use anonymous volume and provide a more specific path so that docker would know that we don't need to overwrite.
Preference is given to the path which is more specific

```
docker run -p 3000:80 --name node-app --rm -v feedback:/app/feedback -v "/Users/rizwan/Downloads/data-volumes-03-adj-node-code:/app" -v /app/node_modules node-app:volumes
```

The empty node_modules folder seems to be a side-effect which Docker creates / shows us because we use a bind mount + overwrite the node_modules part in the container with the node_modules folder that was created in the image.

So the empty node_modules folder basically shows up because We "omit" node_modules when mirroring our host machine folder into the container.

### A Node specific adjustment: Using Nodemon in the container
If we are suppose to console log some statement then it does not work. For the log to reflect we need to restart the node server. That means rebuilding the image and starting a new container of that image. We can use nodemon for the same that will restart the server not the container if it detects any change in the files.   
To use nodemon we need to make following changes.
In package.json
```
  "scripts": {
    "start": "nodemon server.js"
  },
  "devDependencies": {
    "nodemon": "2.0.4"
  }
```
In Dockerfile
```
CMD [ "npm", "start" ]
```

### Summary Volumes and Bind Mounts
Refer the slides

### Read Only Volumes
The idea is that the host file should be able to write to the container but the container should not make any changes to our host file for that to work we append ```:ro``` after we specify the bind mount. But our app is writing to 2 folders that are 1. /app/feedback and 2. /app/temp. Since we are already using named volume for feedback folder hence we need not worry about it, But we have to use anonymous volume for /app/temp folder

```
docker run -p 3000:80 --name node-app --rm -v feedback:/app/feedback -v "/Users/rizwan/Downloads/data-volumes-03-adj-node-code:/app:ro" -v /app/node_modules -v /app/temp node-app:volumes
```

### Managing Docker Volumes
> Volumes are managed by docker  

We can use the following commands with docker volumes
```

Usage:  docker volume COMMAND

Manage volumes

Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove all unused local volumes
  rm          Remove one or more volumes
```

### Using "COPY" vs Bind Mounts
Since we are using the Bind Mounts to map our code to the container then what is the need of COPY . . instruction in the Dockerfile. Can we remove the COPY . . instruction from the Dockerfile.  
Yes, We can remove the instruction and everything we work just fine. But we are using Bind Mounts for development purpose only. When we run the code on production we would not be using bind mounts and in production we rely on the snapshot of the code Hence use COPY . . instruction in the Dockerfile.

#### Don't copy everything: Using dockerignore files
To ignore some files while copying files from the host machine into the container. Use .dockerignore file just like .gitignore

Adding more to the .dockerignore File
You can add more "to-be-ignored" files and folders to your .dockerignore file.

For example, consider adding the following to entries:

Dockerfile
.git
This would ignore the Dockerfile itself as well as a potentially existing .git folder (if you are using Git in your project).

In general, you want to add anything which isn't required by your application to execute correctly.

### Working with env variables and .env files
- ARGuments & ENVironment Variables
> Docker supports build-time arguments and runtime environment variables  

| ARG | ENV |
--- | ---
Available inside of Dockerfile, Not accessible in CMD or any application code | Available in Dockerfile and in application code.
Set on image build (docker build) via --build-arg | Set via ENV in Dockerfile or via --env on docker run

They allow us to create more flexible images and containers. 

### ENVironment Variables
To use env variables in node we can access them by using ```server.listen(process.env.PORT)```. Different languages have different ways to use the environment variables.  
To announce them in Dockerfile we add a line ```ENV PORT 80``` this says that there is an env variable PORT with the default value of 80.  
ENV variables are often typed in UPPERCASE.  
We can use the variable using ```EXPOSE ${PORT}```  
To change the env variable on runtime we use --env or -e flag. This will overwrite the default value. 
```
docker run -d -p 3000:3000 --env PORT=3000 --name node-app --rm -v feedback:/app/feedback -v "/Users/rizwan/Downloads/data-volumes-03-adj-node-code:/app:ro" -v /app/node_modules -v /app/temp feedback-node:env
ad103fe7a95f63589128a2d0e9e2671954b703fc1bbb96aa7d841520920e2b01
```
Instead of --env we can also use -e flag. To declare multiple env variable we can use multiple e flag.  
We can also declare all of our env variables in one file and then use it in the command.    
File that contains env variables is generally named .env  
An example of .env file
```
PORT=80
```
The command to run a container with environment file
```
docker run -d -p 3000:3000 --env-file ./.env --name node-app --rm -v feedback:/app/feedback -v "/Users/rizwan/Downloads/data-volumes-03-adj-node-code:/app:ro" -v /app/node_modules -v /app/temp feedback-node:env
```

### Environment Variables & Security
One important note about environment variables and security: Depending on which kind of data you're storing in your environment variables, you might not want to include the secure data directly in your Dockerfile.

Instead, go for a separate environment variables file which is then only used at runtime (i.e. when you run your container with docker run).

Otherwise, the values are "baked into the image" and everyone can read these values via docker history <image>.

For some values, this might not matter but for credentials, private keys etc. you definitely want to avoid that!

If you use a separate file, the values are not part of the image since you point at that file when you run docker run. But make sure you don't commit that separate file as part of your source control repository, if you're using source control.

### Build-time arguments
To build our dockerfile with more flexibility we can use arguments. With the help of arguments we don't need to change the dockerfile again and again.  
We cannot use arguments in our code. We can only use it in our Dockerfile. In Dockerfile too we cannot use it in every command.   
To declare an argument in Dockerfile ```ARG DEFAULT_PORT=80```  
To use an argument in Dockerfile ```ENV PORT ${DEFAULT_PORT}```. 
To change the argument while building an image ```docker build -t feedback-node:web-app --build-arg DEFAULT_PORT=3000 .```  

## Resources
1. [slides-data-volumes](https://drive.google.com/file/d/1eSYtkS73CWc5lcF_BDsDP3y5tqFzgPa4/view?usp=sharing)
2. [Cheat-Sheet-Data-Volumes](https://drive.google.com/file/d/1wLnSFFVTnsr9pgHQkEYVTX4wpg4bkd5G/view?usp=sharing)