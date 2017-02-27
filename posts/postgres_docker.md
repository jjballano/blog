# Docker + Postgres 
#####(27/02/2017)

I wanted to try Docker and thought it would be a good idea to start with a very simple stuff, just add a postgres to my current pet project. 
I'm completely newbie in Docker, so I could make some mistakes in this post. I'll try to keep it updated when learn new things.

As I'm working on a Mac (same for Windows), I need [docker-machine](https://docs.docker.com/machine/) tool in order to create a VM in VirtualBox and then run Docker on it. If you are on a Linux, just skip the "docker-machine" stuff.

First step, create and run the VM:

```
docker-machine create MY_VM_NAME --driver virtualbox

docker-machine start MY_VM_NAME

eval $(docker-machine env MY_VM_NAME)
```

MY_VM_NAME is the name you want for this particular VM

If you didn't execute the 'eval' sentence, you'd get an error "Cannot connect to the Docker daemon. Is the docker daemon running on this host?" as it is needed in order to get your shell configured.

As docker containers don't keep the state when they are stopped, we need to create a volume assign to a particular folder in our local machine.
I think you can't use relative paths so you can add the absolute path or use $(pwd) to set the current folder as the place to link the volume

```
docker create -v THE_FOLDER_TO_PLACE_THE_VOLUMEN --name ANY_NAME_FOR_THE_VOLUME busybox

i.e: docker create -v $(pwd)/data --name myVolume busybox
```

Once we have the volume created, we can look for the image we want to use in [Docker Hub](hub.docker.com). I have search for postgres and use the latest version (9.6.2 right now). I have use the [official image](https://hub.docker.com/_/postgres/), which means with no prefix. 

```
docker run --name CONTAINER_NAME -p 5432:5432  -e POSTGRES_PASSWORD=ANY_PASSWORD -d --volumes-from VOLUME_NAME_TO_USE postgres:VERSION

docker run --name postgres9.6.2 -p 5432:5432  -e POSTGRES_PASSWORD=aa -d --volumes-from myVolume postgres:9.6.2
```

I could set no version but I want to use a specific version so any other colleague will use the same, no matter which is the latest version when he/she pull it.
As you can see, we forward the port 5432 from the container to our local environment, so we can access from outside.
We also set the password for the database (default user is postgres). You can set more params, find more info in the [image doc](https://hub.docker.com/_/postgres/)

Last thing is to know which IP you need to access to in order to use the database. You can find it running:

```
docker-machine ip MY_VM_NAME
```

So now you have a postgres up&running ready to be used by your application. I still to see how to set a fixed IP so I wouldn't need to change the configuration of my application every time the IP change


The good thing with Docker is that you can have a postgres ready very quick, test with different versions (pull & run diffent ones) or even different databases. You could even automatize your integration tests to start/stop the container they need to test.
But my favourite thing of this is that your local machine remains clean :)


Extra ball:

- Delete things:
```
docker rm MY_CONTAINER
docker rmi MY_IMAGE
docker-machine rm MY_VM
```

- Stop things:
```
docker stop MY_CONTAINER
docker-machine stop MY_VM
```
(Make nonsense to stop an image, as it is a static thing)

- List things:
Running containers
```
docker ps 
```

All containers
```
docker ps -a
```

All images
```
docker images
```

All VM
```
docker-machine ls
```