# Docker Container Creation Process
Steps to create and run containers:
1. Create A Dockerfile
2. Use the Dockerfile to create a container image
3. Use the container image to create a running container

## Docker File Example
Use a Dockerfile to create a running container:
- This sample dockerfile has the commands FROM and CMD
- `FROM`: Defines the base Images (Parent Images)
- `CMD`: Execution script when the containe is running.

## Docker Build Command
Create a container image using build command:
```
docker build -t mp-app:v1 .
```
where:
- `-t`: tag
- `my-app`: repository
- `:v1`: versioning
- ` .`: current directory dockerfile to build

![Docker Commands](/images/docker-1.png)
<br>

# Docker Objects
## Dockerfile
- A Dockerfile is a text file that contains instructions needed to create an image
- The Dockerfile is created using any editor from the console or terminal
- Let`s go over a view instructions taht Docker provides:
    - FROM
        - Defines base image
    - RUN 
        - Executes Arbitrary Command
    - CMD 
        - Defines default command for container execution

## Docker Image
- Read-Only tmeplate with instructions for creating Docker Container
- Built using instructions in a Dockerfile; a new read-only mage layer is created for each instructions
- A writeable layer is added when an image is run as a container 
- Layers can be shared between images, which saves a disk space and network bandwidth.

### Container image naming
```
hostname/repository:tag
```
where:
- `hostname`: image registry
- `repository`: a grpup  of container images
- `tag`: contain information specific version or variant of an image

example:
```
docker.io/ubuntu:18.04
```
where:
    - `hostname`: docker.io -> Docker Hub Registry
    - `ubuntu`: repository name
    - `18.04`: ubuntu version or variant

## Docker Container
A docker container:
- is a runnable instance of an image
- Can be created, stopped,  started or deleted using  the Docker API or CLI
- Can connect to multiple  networks, attach storage, or create new image based  on its current state.
- Is well isolated from other containers and its host machine.

# Docker Architecture
- Based on client-server architecture
- Provides a complete application environment
- Includes the client, the host, and the resgistry component

## Docker process overview
1. Using either the Docker Command Line Interface (CLI) or REST APIs, the Docker client sends instructions or commands to the docker host server
2. The docker host server, referred as the host includes the Docker Daemon known as dockerd.
3. The Daemon listens for Docker API requests or commands such as `docker run` and processes those commands
4. the daemon builds, runs, and distributes containers to the registry
5. the registry store the images.
![docker process](/images/docker-2.png)<br>

# Excersise: Introduction to Containers, Docker and IBM Cloud Container Registry
This lab will demonstrate how to pull an image from Docker Hub and run an image as a container using docker, and build an image using a Dockerfile and push an image to IBM Cloud Container Registry. 

## Objectives
In this lab, you will:
- Pull an image from Docker Hub
- Run an image as a container using docker
- Build an image using a Dockerfile
- Push an image to IBM Cloud Container Registry

### Verify the environment and command line tools
1. Verify that docker CLI is installed.
```
docker --version
```
2. Verify that ibmcloud CLI is installed.
```
ibmcloud version
```
3. Clone the git repository that contains the artifacts needed for this lab, if it doesn’t already exist.
```
[ ! -d 'CC201' ] && git clone https://github.com/ibm-developer-skills-network/CC201.git
```
4. Change to the directory for this lab by running the following command. cd will change the working/current directory to the directory with the name specified, in this case CC201/labs/1_ContainersAndDcoker.

### Pull an image from Docker Hub and run it as a container
1. Use the docker CLI to list your images.
```
docker images
```
2. Pull your first image from Docker Hub.
```
Pull your first image from Docker Hub
```
3. List images
```
docker images
```
You should now see the hello-world image present in the table.
![images](/images/docker-3.png)<br>
4. Run the hello-world image as a container.
```
docker run hello-world
```
You should see a ‘Hello from Docker!’ message.
5. List the containers to see that your container ran and exited successfully.
```
docker ps -a
```
6. Note the CONTAINER ID from the previous output and replace the <container_id> tag in the command below with this value. This command removes your container.
```
docker container rm <container_id>
```
7. Verify that that the container has been removed. Run the following command.
```
docker ps -a
```
Congratulations on pulling an image from Docker Hub and running your first container! Now let’s try and build our own image.

### Build an image using a Dockerfile
1. The current working directory contains a simple Node.js application that we will run in a container. The app will print a hello message along with the hostname. The following files are needed to run the app in a container:
    - app.js is the main application, which simply replies with a hello world message.
    - package.json defines the dependencies of the application.
    - Dockerfile defines the instructions Docker uses to build the image.
2. Use the Explorer to view the files needed for this app. Click the Explorer icon (it looks like a sheet of paper) on the left side of the window, and then navigate to the directory for this lab: CC201 > labs > 1_ContainersAndDocker. Click Dockerfile to view the commands required to build an image.
![docker](/images/docker-4.png)<br>

3. Run the following command to build the image:
```
docker build . -t myimage:v1
```

### Run the image as a container
1. Now that your image is built, run it as a container with the following command:
```
docker run -dp 8080:8080 myimage:v1
```
The output is a unique code allocated by docker for the application you are running.
2. Run the curl command to ping the application as given below.
```
curl localhost:8080
```
3. Now to stop the container we use docker stop followed by the container id. The following command uses docker ps -q to pass in the list of all running containers:
```
docker stop $(docker ps -q) <containerId>
```
4. Check if the container has stopped by running the following command.
```
docker ps
```

### Push the image to IBM Cloud Container Registry
1. The environment should have already logged you into the IBM Cloud account that has been automatically generated for you by the Skills Network Labs environment. The following command will give you information about the account you’re targeting:
```
ibmcloud target
```
2. The environment also created an IBM Cloud Container Registry (ICR) namespace for you. Since Container Registry is multi-tenant, namespaces are used to divide the registry among several users.

3. Ensure that you are targeting the region appropriate to your cloud account, for instance us-south region where these namespaces reside as you saw in the output of the ibmcloud target command.
```
ibmcloud cr region-set us-south
```

4. Log your local Docker daemon into IBM Cloud Container Registry so that you can push to and pull from the registry.
```
ibmcloud cr login
```
5. Export your namespace as an environment variable so that it can be used in subsequent commands.
```
export MY_NAMESPACE=sn-labs-$USERNAMEs
```
6. Tag your image so that it can be pushed to IBM Cloud Container Registry.
```
docker tag myimage:v1 us.icr.io/$MY_NAMESPACE/hello-world:1
``` 

7. Push the newly tagged image to IBM Cloud Container Registry.
```
docker push us.icr.io/$MY_NAMESPACE/hello-world:1
```