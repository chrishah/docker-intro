# Hands on with Docker - Running and interacting with prebuild images

__Disclaimer__: The following session was designed by my esteemed colleague Philipp Resl [@reslp](https://github.com/reslp).

The following session assumes that you have Docker running on your computer (we tested with Docker version 18.09.7, build 2d0083d).
If you're running on a Linux host it further assumes that you have it set up in such a way that you don't have to prepend `sudo` to the `docker` command each time. There is a number of ways you can do that, e.g. by creating a docker group and adding your user to it or by running the Docker daemon as a non-root user (find some more info/instructions <a href="https://docs.docker.com/engine/install/linux-postinstall/" title="Post-installation steps for Linux (last accessed 24.04.2020)" target="_blank">here</a>). If for some reason you don't want to or can do any of these things, all of the below should also work if you simply prepend `sudo` to the `docker` command.

## Intro

A successful Docker installation runs in the background out of sight of the user. At its core is the so-called Docker deamon (or `dockerd`) which manages different docker objects such as containers, images or volumes.
The way to interact with the Docker daemon and tell it what to do is through a command line interface (CLI) program which is called `docker`. We refer to this here as the `docker` command. 
The docker command is very powerful and it allows us to change many aspects of how Docker runs containers, manages images or creates volumes. In this first live coding session, we will be introducing the `docker` command and show how to run and interact with Docker images.

## Requirements

The focus of this live coding session is the basic usage of the `docker` command. We will also use several basic Linux/Unix commands like `ls`, `pwd`, `cd` and we assume that you are familiar with them. You can look, e.g. [here](https://swcarpentry.github.io/shell-novice/) for a short introduction on the Linux command line to refresh your knowledge. Additionally, to be able to replicate the commands presented here you will need a working Docker installation.

## Definitions

Several terms will come up regularly during the live coding-sessions and in this document. Therefore we have summarized them here again:

- Docker image: A read only file which contains all the code, library, dependencies etc. for an application to run. They are basically templates of the software and are the basis of Docker containers.
- Docker container: A container is a writeable copy of an image inside an enclosed environment. When an image lives inside a container it can be modified and the container can be executed.
- Host: This is the computer which runs the Docker installation. On this computer we create containers, images etc. It is also the computer on which we execute the `docker` command. In the code segments of this document all commands executed on the host will be indicated with `(host)-$`.

## First steps with the docker command

You can test your Docker installation by executing the following command, which will print the version of the installed Docker engine:

```
(host)-$ docker -v
Docker version 19.03.8, build afacb8b
```

To get an idea what we can do with docker you can run the docker command without any flag:

```
(host)-$ docker

Usage:	docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/Users/sinnafoch/.docker")
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var and default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/Users/sinnafoch/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/Users/sinnafoch/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/Users/sinnafoch/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  builder     Manage builds
  checkpoint  Manage checkpoints
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  deploy      Deploy a new stack or update an existing stack
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.
```

This may look overwhelming at first but it illustrates that Docker is capable of many different things and there are many different ways how to do them. Also, during your daily docker use, you may actually only need a subset of what is listed above. Because Docker can do so many different things the `docker` command is organized in sub-commands which correspond to different aspects of Docker. Docker sub-commands can be further customized with traditional command-line flags.

***Getting help***
> If you would like to know about the different options you can use the docker command like so to display additional help: `docker COMMAND --help`. For example `docker run --help` will only display options associated with the docker run command.

### Lets run our first container from a pre-built image

Probably the first container every new Docker user runs is the [hello-world](https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) container. We will also follow this tradition to execute the hello-world docker container:

```
(host)-$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:8e3114318a995a1ee497790535e7b88365222a21771ae7e53687ad76563e8e76
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

A lot is going on when this command is executed apart from printing the traditional "Hello" message. It also provides some additional information about what just happened: As you can see from the output above when executing the command `docker run hello-world:latest` `docker` communicates with the docker deamon and requests a container of the hello-world image. The docker daemon realized that this image is not yet available on our computer, so it downloads it from the [Docker Hub](https://hub.docker.com) (this is usually referred to as *pulling*). The Docker daemon stores the hello-world image on the host and creates a virtualized runtime environment (the *container*). When this container is executed it can produce some output (in case of hello-world this is the message above), which is displayed on the terminal screen.

***DockerHub***
> Docker Hub is a large online repository of custom Docker images made by other users. We will have a closer look on how it works in the next session. 

As already mentioned `docker run` automatically pulls an image if it is not already available on the host. It is however also possible to just pull it without immediately creating a container. This can be done with `docker pull`. We will now pull an plain ubuntu image. Note also that we are pulling a specific version (which is indicated by the colon after the image name). 

```
(host)-$ docker pull ubuntu:18.04
```

***Be explicit with image versions***
> It is good practice to always specify the version of an image when creating a container. This ensures reproducibility and the same behavior during every run. In the case of hello-world we did not specify a version, which always results in docker checking for and pulling (if necessary) the `latest` version. This could break your workflow if the image is updated because if a newer version is available it will automatically download it. This new image then replaces the old one.

## Managing containers and images

Once you have accumulated many images and run different containers it becomes important to manage the available images and running (or stopped) containers. The `docker` command also comes to the rescue here

### List available images

To list all images off which you can base containers you can use the `docker images` command:

```
(host)-$ docker images
REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
hello-world                       latest              bf756fb1ae65        3 months ago        13.3kB
ubuntu                            18.04               ccc6e87d482b        3 months ago        64.2MB
(host)-$
```

This gives an overview of your downloaded images as well as intermediate images which are created when you build them yourself. Each image has an ID consiting of letters and numbers. This ID can be used to remove an image. For example you could run `docker image rm bf756fb1ae65` to remove the hello-world image from your computer. Image removal only works when there are no containers relying on that image.

### List containers

To list all running containers you can execute `docker container ls`. If you have no currently running containers the output from this command will be an empty list. 
```
(host)-$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
(host)-$
```

We can also list all containers regardless if they are currently running or not.

```
(host)-$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                           PORTS               NAMES
36f65c44b177        ubuntu:18.04        "sleep 30"               9 minutes ago       Exited (0) 8 minutes ago                             intelligent_lewin
52c9c0117a2f        hello-world         "/hello"                 16 minutes ago      Exited (0) 16 minutes ago                            tender_germain
22c3563c46a5        ubuntu:18.04        "/bin/bash"              About an hour ago   Exited (0) About an hour ago                         happy_burnell
3a2e784dd2f8        hello-world         "/hello"                 About an hour ago   Exited (0) About an hour ago                         loving_hermann
```

Depending on how much you have already played around there may be a looong list of containers that have been run before and after a while it gets messy. Two things can help here:
- name your containers with the `--name` flag
- tell docker to remove containers after they're done executing with the `--rm` flag.

We will use those two right away, but first let me remove all containers that are done executing.
```
(host)-$ docker container prune 
```

## Executing commands within a container

Lets try something a bit more advanced: In the last section we saw how the hello-world container displayed some text on our terminal screen before it exits back to our command prompt. This very simple container only runs for a few seconds and the only thing it does is to display the message above. However, often it is desired to change the execution of a container as it runs or run specific commands inside the container. In fact this is probably one of the most common use cases for many scientists. Let's see how we can execute (almost) any command inside a docker container:

For this example we will use a more complete container (note that we will also name the container) based on the official ubuntu:18.04 image:

```
(host)-$ docker run --rm ubuntu:18.04 sleep 10
(host)-$
```

Running the above command will download the ubuntu:18.04 image (if you followed the above steps until here it's already available) and then execute the sleep command inside a new ubuntu:18.04 container. All the sleep command does is to tell the container to wait for 10 seconds until it exists. This addmittedly very simple command should illustrate an important point: You can basically run any program from inside your container as long as it is installed in it.

Here are some additional examples with the ubuntu:18.04 container.

Show the OS version installed in the container:

```
(host)-$ docker run --rm ubuntu:18.04 cat /etc/os-release
NAME="Ubuntu"
VERSION="18.04.3 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.3 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

List the content of the `/` directory in the container:

```
(host)-$ docker run --rm ubuntu:18.04 ls /
bin
boot
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
```


## Working inside a container:


You may ask yourself now how it would work if you wanted to run multiple commands inside your container or how you could prevent your container from exiting immediately after execution of a command. This can be done by providing the `-i -t` flags (usually used as `-it`). 

Lets get inside an ubuntu container (note that this time we will name it, but ommit the `--rm` flag):

```
(host)-$ docker run -it --name ${USER}s_ubuntu ubuntu:18.04
root@f11c02f856a7:/# #do whatever you want


root@f11c02f856a7:/# exit
```

Inside our container we can do all kinds of things: Create files, install software download files from the internet etc. All of this works in a familiar ubuntu environment provided by Docker.

***What happens in the container, stays in the container***
> Changes you make in interactivte mode inside a container are restricted to the currently running container. Each docker run command will spawn a new container instance which only contains what is in the underlying Docker image.

## Restarting stopped containers

Let's see if the containers we just ran are still there.

```
(host)-$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
86649d0e7bf4        ubuntu:18.04        "/bin/bash"         3 seconds ago       Exited (0) 2 seconds ago                        user3s_ubuntu
81df4b1c0577        ubuntu:18.04        "/bin/bash"         9 seconds ago       Exited (0) 6 seconds ago                        user2s_ubuntu
66e3531f60d6        ubuntu:18.04        "/bin/bash"         15 seconds ago      Exited (0) 12 seconds ago                       user1s_ubuntu
...
```

They have not disappeared (because we did not use the `--rm` flag) - they have just stopped running and we can identify them based on the names we specified. Docker saves a copy of each executed container. Consequently the changes we made inside the ubuntu container previously (if any) should still be there somewhere. We just have to find the correct container and execute it again to get to our files again. The docker command has an option to restart stopped containers. 

We can do that based on the name of the container.
```
(host)-$ docker start -ia user1s_ubuntu
```

Or based on the unique container ID.
```
(host)-$ docker start -ia 66e3531f60d6
```

***info***
> Note that -ia is the equivalent to -it in docker start.

***Reminder***
> If you don't want to keep a copy of the container after it's done you can add the flag `--rm` to your `docker run` command.

## Sharing data with the host system

Often, it is desired to share data from the host computer with the container. For example you may want to analyse files you created inside your container or you may want to copy files from inside your container to your computer. Docker provides two ways to do this: Docker volumes and bind-mounting whole directories. We will introduce both approaches here:

### Mounting directories

It is often necessary to copy files between the host and the container or access files from the host via the container. 
You can bind-mount directories directly to your containers using the `-v` flag in `docker run`.
One could say the `-v` flag works like this: Take the location on the host on left side of the colon and make it available as new directory in the container, specified on the right side of the colon. Here the right side can be a longer path as well, it is however important that the path is absolute (starts with / ). This is referred to as binding, mounting or bind-mounting. You will come across all three terms online.

```
(host)-$ docker run --rm -v $(pwd):/data ubuntu:18.04
```

This command will mount the current working directory on your host to the `/data` folder inside the ubuntu container. You can now make changes to that folder inside your container and the changes will be synced with the folder on the host computer.

We will now create a `testfile_from_host` in the current directory using the host computer. Then we will start a container mounting this directory. Inside the container we will create another `testfile_from_container` from within the container. All changes persist also when we exit the container:

```
(host)-$ pwd
/home/ubuntu
(host)-$ touch testfile_from_host
(host) $ ls
testfile_from_host
(host)-$ docker run -it --rm -v $(pwd):/data ubuntu:18.04
root@a0f138701fc5:/# cd /data
root@a0f138701fc5:/data# ls
testfile_from_host
root@a0f138701fc5:/data# touch testfile_from_container
root@a0f138701fc5:/data# ls
testfile_from_container	testfile_from_host
root@a0f138701fc5:/data# exit
exit

(host)-$ ls
testfile_from_container testfile_from_host

(host)-$

```
### Docker volumes

A Docker volume is a special place in the host file-system which is used to store data generated by the runnning container (they are just folders on your host computer, stored in place we don't normally access, e.g. on Linux Docker stores them in `/usr/lib/docker/volumes`). Docker will automatically create a volume for each running container. The idea behind this is to keep files created during runtime seperated from the image to make it easy to transition to different image versions. 
Apart from these automatically created volumes, we can also create one manually (named volume):

```
(host)-$ docker volume create ${USER}_data
ubuntu_data
```

With `docker volume ls` we can list our current volumes:

```
(host)-$ docker volume ls
DRIVER              VOLUME NAME
local               ubuntu_data
local               user1_data
local               user2_data
local               user3_data
```

***Volumes are handy***
> Volumes are especially handy to share data between more complex setups with multiple containers. e.g. databases
	

After we created the volume we can tell Docker to make it available when a container is run. This is done like this:

```
(host)-$ docker run -it --rm --mount type=volume,source=${USER}_data,target=/data ubuntu:18.04
```

Now, inside the container we can move to the bound volume and create some dummy data:

```
root@eca8560a6bd1:/# cd /data
root@eca8560a6bd1:/data# ls
root@eca8560a6bd1:/data# mkdir testdata
root@eca8560a6bd1:/data# touch file_inside_the_container
root@eca8560a6bd1:/data# ls
file_inside_the_container  testdata
root@eca8560a6bd1:/data# exit
```
We can now run a completely different container, have it include the same volume and then list its contents:

```
(host)-$ docker run -it --rm --mount type=volume,source=${USER}_data,target=/data alpine:3.11
Unable to find image 'alpine:3.11' locally
3.11: Pulling from library/alpine
cbdbe7a5bc2a: Pull complete
Digest: sha256:9a839e63dad54c3a6d1834e29692c8492d93f90c59c978c1ed79109ea4fb9a54
Status: Downloaded newer image for alpine:3.11
/ # cd /data
/data # ls
file_inside_the_container  testdata
/data #
```

Very nice. The volume is now part of both containers. We could now make additional changes to the files and then restart the Ubuntu container to look at the changed files.

To remove a volume you can run:

```
(host)-$ docker volume rm ${USER}_data
ubuntu_data
```

# Summary

In this first live-coding session we had a first look at the `docker` command and how we can use it to run and interact with containers from pre-built images. We have also seen how we can share data between containers and between the contatiner and the host system. The main command to create and run a container is `docker run`. We can change it's behavior with command-line flags such as `-it` to make the container interactive ore `-v` to mount folders or volumes. We saw that it is possible to list running containers with `docker container ls` and view all available images with `docker images` (an alternative command would be `docker image ls`). We can create Docker volumes with `docker volume create` and delete them with `docker volume rm`.

The commands and examples provided here are really only the tip of the iceberg. There are many more things you can do, which would have been outside of the scope of this first introduction. If you are curious what else you can do, here are some interesting links from the Docker documentation:

- Extensive Reference of the [docker](https://docs.docker.com/engine/reference/commandline/cli/) CLI.
- [More](https://docs.docker.com/storage/volumes/) on Docker Volumes
- [Docker Hub](https://hub.docker.com)

# Contact

- Christoph Hahn - <christoph.hahn@uni-graz.at> - [@chrishah](https://github.com/chrishah)
- Philipp Resl - <philipp.resl@uni-graz.at> - [@reslp](https://github.com/reslp)
