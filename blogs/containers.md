# Containers in a Week  

## Baremetal Infrastructure 
## Virtualized Infrastructure
## Containerized Infrastructure 

## What is Virtualization ?

### Hypervisor (Virtual Box, Hyper- V, ESX/ESXi)

### Type-I
- Does not provide the host OS
### Type- II 
- Provides with Host OS

### CRI (Container Runtime Interface): 

    - Manages containers life cycle.  
    - Docker , Podman, Containered , CRI-o
    - Runs on HOST OS. 
    - Dont have GUEST OS, means they share underlying HOST OS.
    - All container runs as a processes and control is provided by CRI. 
    - Container need container's images that contains application related binaries/libs. It share the packages from HOST OS as well.
    - It uses 2 kernal features cgroup and namespace and its not there in windows kernal so its developed for linux. Recommended host is Linux. 
    - If Host OS is linux then it will only run the linux comptible application same with windows , cross platform is not supported in containers.
    - Docker has 2 editions : 
        - COmmunity Edition
        - Enterprise Edition called MKE 
    - 3 Terminology 
        - Dockerfile : Docker instuctions to run application.
        - Image : Outcome of the build
        - Container : Using docker image , we can create one or more containers.
    - Dockerhub :
        - Public Registry for images , having collection of images for different use cases.
        - Images can have one or more versions. 
        - To push the images we should be registered user. 
        - Official repo is called "library". 

        - docker system info 

        -  docker image : list the images from  ls  : /var/lib/docker 

        - /etc/docler/daemon.json is the file when we need to change some default configuration 

        - Docker has some management commands : 

        - Container Management commad : <>
### How to install the Docker in CentOS: 

### Managing the Docker Containers 

### Managing the Docker Images 

### Save and Load images

### How to commit the container state as an image

### Building an Image using Dockerfile

###  Docker Container (App) Export as a Tar ball and Import as an Image



### Docker Registry 

### Docker Volumes


# Kubernetes 

## Architecture

## Setup & Configuration

## Working with POD & Multi-Container Pod

## kubectl 

## Kubernetes Pattern 

## ReplicaSet
