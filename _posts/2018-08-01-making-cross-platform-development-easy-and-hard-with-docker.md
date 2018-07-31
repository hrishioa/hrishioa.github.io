---
layout: post
title: "Development across platforms with Docker"
date: 2018-01-05 02:25:28
image: 'Docker/martin-reisch-272883-unsplash.jpg'
description: 'Making things easier and harder'
tags:
categories:
twitter_text:
---

Containerization has only recently become easy and cheap (in terms of memory, cost and development time) through tools like [Docker](https://www.docker.com/). This has made orchestrating large microservice and distributed architectures far easier, as well as cutting down development time for building and releasing tools. As we saw in the case of [Oyente](https://oyente.github.io/benchmarks/) as well as a few other projects I've listed here, releasing complex pieces of software - especially for development and research purposes - has become much easier. Complicated and ever-updating lists of dependencies and kernels and distros can be effectively frozen in place and easily shared as Docker images.

Years after release, `docker pull hrishioa/oyente` will still provide you with the exact same environment that the tool was in at the point of release, which makes the fact that it comes with it's own perfectly sandboxed OS and sliceable filesystem just the icing.

While Docker and its assorted set of tools have been a boon for all of these reasons, it's been a primary component of my development toolchain for a different reason: cross-platform software development. By which I don't mean developing *for* multiple platforms, I mean developing *from* multiple platforms. Through the course of my day I often need to work seamlessly through Windows, MacOS and Linux machines, and while one of those is definitely not like the other it still doesn't make switching between endlessly changing package managers, keyboard shortcuts and development environments any easier. Docker makes my life a hell of a lot easier, and here we'll cover enough to get started using it - and a case study in setting up my blog environment as a portable image.

## Basic commands

Unfortunately, downloading docker has become somewhat harder, now that you need to register and create an account to do it. If you still want to, head over to the [Docker Store](https://store.docker.com/search?type=edition&offering=community) to download the free community edition after creating an account. There's a dedicated installer for your platform, and keep in mind the Windows installer will ask you [to enable Hyper-V](https://docs.docker.com/machine/drivers/hyper-v/). This wasn't much of a problem for me, seeing as the last time I used a VM was to try out the first ever alpha of ChromeOS. However, keep in mind that this will disable Virtualbox, and I'll understand if you'd rather keep your VMs.

Once Docker is installed, running

```sh
$ docker run hello-world
```

should test your docker installation and make sure things are running well.

Now, there are any number of Docker guides on the internet which are great for learning Docker in depth - [the Docker docs](https://docs.docker.com/get-started/#docker-concepts) are a good example, and you can definitely chuck this guide if you're going deep - so here we're going to focus on the basic tools and commands to help you make your environments portable, while maintaining some idea of what's happening underneath.

Docker has two primary *types of objects* , if we're to call them that: Images and Containers. **Images** are similar to the persistent memory or hard drive data on a system, containing the OS or application data. This is usually created once and used often, but remains unchanged after creation. **Containers** are created from images, and encapsulate the image along with some process memory to create what is effectively a complete sandboxed operating system. A container is a runtime instance, residing in memory and lost once it is destroyed. The image is usually what you create and share, often spawning any number of containers.

Let's try listing the containers and images currently present on your system. Before we do so, make sure to start the Docker daemon (usually the executable installed by the installer and give it a little while to finish setting up.

```sh
$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE

$ docker containers ls 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

They're both empty. but the columns are instructive. Docker images are often stored in repositories (not unlike those in git), contain tags specifying a version or release, as well as Image IDs for unique identification. Containers have an ID as well, along with which image they were created from, the running command (each container has a primary command run when the container spins up), the current status (up or down and how long), which ports are exposed to the host system, as well as a plaintext name for identification.


Let's run our first container, and let's say we pick the latest LTS version of Ubuntu, 18.04. Creating a new container is usually done through the `docker run` command:

```sh
$ docker run -it --name first_container ubuntu:18.04
Unable to find image 'ubuntu:18.04' locally
18.04: Pulling from library/ubuntu
c64513b74145: Pull complete
01b8b12bad90: Pull complete
c5d85cf7a05f: Pull complete
b6b268720157: Pull complete
e12192999ff1: Pull complete
Digest: sha256:3f119dc0737f57f704ebecac8a6d8477b0f6ca1ca0332c7ee1395ed2c6a82be7
Status: Downloaded newer image for ubuntu:18.04 
root@c5f6d53ad1b9:/#
```

Let's use the output to understand what's going on. Images are usually specified in the format `name:tag`. We're asking docker to create an instance from the image ubuntu of tag 18.04, naming the container `first_container`. The `-i` switch keeps STDIN open on the process we're running, and the `-t` switch specifies a pseudo-terminal so that we can interact with the container. The default process set to run on this image is bash. As the command executes, we see docker looking for a local copy of the image, and failing which it fetches the image from the [repository](https://hub.docker.com/_/ubuntu/) on Docker Hub. Images are built as layers, often on a storage driver like [AUFS](https://docs.docker.com/storage/storagedriver/aufs-driver/), and we'll see how this is done later. The layers make changing and building on intermediate versions of an image much easier. Once the image is available locally the container is booted, allocated a random ID, and we're presented with a root prompt on the now running container. If you open a separate terminal and run the previous commands again, you can see that things have changed:

```sh
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              18.04               735f80812f90        4 days ago          83.5MB

$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c5f6d53ad1b9        ubuntu:18.04        "/bin/bash"         6 minutes ago       Up 6 minutes                            first_container
```

If you typed exit at the container's prompt or closed the terminal session before running `docker container ls`, no running containers will show up. `docker container ls --all` should show you all the containers.

Herein we can see the power of Docker. the entirety of the ubuntu image is 83.5 MB in size, and boots up in a few seconds on most modern laptops. This is really what makes Docker viable for quick encapsulation of development spaces, unlike Virtual Machines.

Next, let's leave the container running and see if can run a separate command on the same container. If you closed the container from before, you can also restart it:

```sh
$ docker start -i first_container
```

Should have you back up. Failing this, you can always create a new container with `docker run` but a different name.

With a container runnning, `docker exec` allows you to run more commands within the same container. Let's first try something that isn't interactive:

```sh
$ docker exec first_container cat /etc/passwd
```

This should dump the contents of the passwd file from the running container directly to STDOUT. You can also run an interactive session, like so:

```sh
$ docker exec -it first_container /bin/bash
```

Which should give you a second prompt. Now that's all well and good, but before we move on, let's do some cleanup. Working within Docker, especially when creating your ownn images, can create a lot of crud if you're not careful. Docker keeps all the stopped containers and old images around. There wasn't a solution to this before, needing the user to individually (or collectively but carefully) clean up containers and images. Now, Docker's [pruning](https://docs.docker.com/config/pruning/#prune-images) functionality allows for easier cleanup. To prune unused images and stopped containers, run

```sh
$ docker system prune
```

For specific changes, you can always run `docker rm <container_name>` and `docker rmi <image_name>` to remove specific containers and images.


## Building images

Next we'll look at how images are built, building one for ourselves. Images are often built using Dockerfiles. You can consider Dockerfiles like recipes, where each line represents an additional operation on an image until the final image is built. They're really quite simple to read and write.

Create a new folder somewhere, navigate to it and create a file named `Dockerfile`. Let's try and build a sample python hello-world server in a Container. First some boilerplate. We're starting with the ubuntu image we used earlier, so that goes in.:

```
FROM ubuntu:18.04
MAINTAINER Hrishi Olickel (hrishioa@gmail.com)
```

All this instructs Docker to do is to use the ubuntu image with tag 18.04 as the base image for building our own, as well as some metadata for posterity's sake.

Let's try building this image:

```sh
$ docker build -t myimage .
Sending build context to Docker daemon  1.536kB
Error response from daemon: the Dockerfile (Dockerfile) cannot be empty
```

We're attempting to build the image using the `docker build` command. The `-t` switch specifies a tag name, and `.` tells Docker to use the current directory as a working directory. It looks for a Dockerfile in this directory and tries to build, and fails. Docker's right; we haven't really done anything yet!

If we're writing a simple python server, we'd need python. Add another line to the Dockerfile:

```sh
RUN apt-get update && apt-get -y upgrade
RUN apt-get install -yq python
```

Dockerfile lines usually follow the `COMMAND <arguments>` structure. The `RUN` command takes an image and executes a new command on it to create another layer. All we're doing here is to make sure the apt repos are up-to-date, and then installing python, with the `-y` switch turned on. Image creation isn't interactive, so the process will abort at any rhetorical question.

Try again:
```sh
$ docker build -t myimage .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM ubuntu:18.04
 ---> 735f80812f90
Step 2/4 : MAINTAINER Hrishi Olickel (hrishioa@gmail.com)
 ---> Running in 0473c58a6d1a
Removing intermediate container 0473c58a6d1a
 ---> d7dffa7d2684
Step 3/4 : RUN apt-get update && apt-get -y upgrade
 ---> Running in 177b62efa677
Get:1 http://archive.ubuntu.com/ubuntu bionic InRelease [242 kB]
...................
Step 4/4 : RUN apt-get install -yq python
 ---> Running in 8503eba9707b
Reading package lists...
Building dependency tree...
...................
Processing triggers for libc-bin (2.27-3ubuntu1) ...
Removing intermediate container 8503eba9707b
 ---> e9007552682a
Successfully built e9007552682a
Successfully tagged myimage:latest
```

First image! We can see what's happening here. At each stage of the process, Docker is creating an intermediate container with the last image, running the command, saving the new filesystem as the next layer and deleting the intermediate image. Now the layers we saw when pulling the ubuntu image makes more sense. Once this is all done, the image is named and tagged. We didn't provide a tag so the default tag of latest is added.

Now let's consider what we'd like our webserver to serve. Being old-fashioned myself, let's serve some good old HTML. First, we need to create the file in our local filesystem.

```sh
$ echo '<h1>HELLO!</h1>' > index.html
```

Next, we need to ask Docker to create a directory, move the file into it, and change the working directory to the one we just made.

```
RUN mkdir /mysite
WORKDIR /mysite
COPY index.html /mysite
```

Of course, we could run the echo command directly in the Dockerfile, but this way we can copy any part of the host's (the system creating the image) filesystem over to the image. Let's build our image again:

```sh
$ docker build -t myimage .
Sending build context to Docker daemon  3.072kB
Step 1/7 : FROM ubuntu:18.04
 ---> 735f80812f90
Step 2/7 : MAINTAINER Hrishi Olickel (hrishioa@gmail.com)
 ---> Using cache
 ---> d7dffa7d2684
Step 3/7 : RUN apt-get update && apt-get -y upgrade
 ---> Using cache
 ---> 41af45683a36
Step 4/7 : RUN apt-get install -yq python
 ---> Using cache
 ---> e9007552682a
Step 5/7 : RUN mkdir /mysite
 ---> Running in 7b590fcf24be
Removing intermediate container 7b590fcf24be
 ---> 027a95797ce6
Step 6/7 : WORKDIR /mysite
 ---> Running in 48dd05f00287
Removing intermediate container 48dd05f00287
 ---> 62455d10d3ff
Step 7/7 : COPY index.html /mysite
 ---> 8f88ea6f63f0
Successfully built 8f88ea6f63f0
Successfully tagged myimage:latest
```

We can see the advantage of layers. The previously cached layers from installing python are used instead of rebuilding them from scratch. This saves a great deal of time when working with large and complicated build files.

Now that we've got the files in the image, we need to start a server. We're going to make use of python's SimpleHTTPServer and run a server on port 8000. Except this time, we're going to use the `CMD` command. Docker has two commands that are separate from the standard `RUN` command: `CMD` and `ENTRYPOINT`. Both of these are used to indicate that this command should be run in the final runtime instance of the container and not run and saved as part of the image. `CMD` can be overriden by the parameters we specify in `docker run`, while `ENTRYPOINT` cannot. 

```
CMD ["python", "-m", "SimpleHTTPServer", "8000"]
```

If you've followed along, the build should be successful. Let's try running our new image:

```sh
$ docker run -it --name tmp myimage:latest
Serving HTTP on 0.0.0.0 port 8000 ...
```

Python is up and running, and seems to be serving on port 8000. But *whose* port is it serving at? Navigating to [http://localhost:8000](http://localhost:8000) should show no indication of a server running. Running `docker container ls` shows no ports listed:

```sh
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
d899b0570ac3        myimage:latest      "python -m SimpleHTT…"   4 seconds ago       Up 3 seconds                            tmp
```

This is the next step. In order to connect the docker container to the outside, we need to expose a port on the container and publish a port on the host. Exposing a port can be done in the Dockerfile:

```
EXPOSE 8000
```

Once we've rebuilt the image, we can run the command the specify that we wish to publish the port to port 8001 on the local host:

```sh
$ docker run -it -p 8001:8000 --name tmp myimage:latest
```

Now, navigating to [http://localhost:8001](http://localhost:8001) should connect to port 8000 and through it the python server on the container.

That's the end of building our first Docker image. The full Dockerfile is below - 

```
FROM ubuntu:18.04
MAINTAINER Hrishi Olickel (hrishioa@gmail.com)
RUN apt-get update && apt-get -y upgrade
RUN apt-get install -yq python
RUN mkdir /mysite
WORKDIR /mysite
COPY index.html /mysite
CMD ["python", "-m", "SimpleHTTPServer", "8000"]
EXPOSE 8000
```

Once this is done, you can also follow the quick process outlined [here](https://docs.docker.com/docker-cloud/builds/push-images/) to tag and push your images to Docker Hub, so that they're accessible publicly.

## Building a full Jekyll/Gulp environment

This is where I hope to highlight some of the problems and some of the advantages of such an approach. I've been using Jekyll and Gulp for managing the website you're reading this on for some time. It's acquired some bloat and curd over the years, but it still works and delivers mostly static content. It's an easy arrangement for me - Jekyll takes care of markdown formatting, gulp and assorted packages take care of activereload and theme-based formatting, Github Pages provides hosting, etc.

However, it is a major pain to set up on a new system, especially if that system is Windows. What this means for me is that if I'm not at my usualy laptop, I tend to put off writing anything - often forever. My primary solution to that has been to create a Docker image with a current working build system that I can quickly spin up, start writing, git push and shut down. 

Without ado, the full Dockerfile is below:

```
FROM ubuntu:16.04
MAINTAINER Hrishi Olickel (hrishioa@gmail.com)

RUN apt-get update && apt-get -y upgrade \
    && apt-get install -yq --no-install-recommends \
      ruby \
      python \
      ruby-dev \
      make \
      build-essential \
      libssl-dev \
      git \
      nodejs \
      curl \
      sudo

RUN echo 'export GEM_HOME=$HOME/gems' >> ~/.bashrc
RUN echo 'export PATH=$HOME/gems/bin:$PATH' >> ~/.bashrc

RUN /bin/bash -c "source ~/.bashrc"

RUN gem install jekyll bundler

RUN apt-get install --yes curl
RUN curl --silent --location https://deb.nodesource.com/setup_8.x | sudo bash -
RUN apt-get install --yes nodejs
RUN apt-get install --yes build-essential

RUN mkdir /Blog && cd /Blog

RUN cd /Blog \
    && git clone https://*************@github.com/hrishioa/hrishioa.github.io

WORKDIR /Blog/hrishioa.github.io

RUN rm -rf node_modules && npm install && npm install -g gulp
```

It looks quite small, but it over two hours to write. The primary issue lies in getting multilingual environments set up, with each package or language having it's own suite of package managers, installers for said package managers and so on. Previously in the case of python, installing proved quite easy - `apt-get install python`. However, if we were to want numpy or jupyter on our system, the installation process is far more complex. Some of the installers are script-based and need to be downloaded into a particular directory, some others are scared of sudo and need a good amount of coaxing, so on and so on. A lot of distributions today come with docker images you can build from (node or ruby in this case), which should usually do the trick. In this case however, neither the node or ruby distributions played well with each other, with permissions issues everywhere. 

So why do it? Many reasons, one of which is that once you solve the problem for one platform and image, you've solved it anywhere you'd like to work. For another, docker images aren't subject to code rot as much as complex environments often are. So if you'd like to preserve a completely executable snapshot of a working project, it's often the best solution. The Dockerfile also serves as an easy to understand way to retrace your steps as opposed to a 20GB Virtual Machine Image that works, but you're afraid to touch for fear of breaking and keep copying endlessly (not me of course, a friend of mine).

Docker is a powerful tool, no doubt. The container image creation and orchestration software could get better (or I could at using them, which is far more likely), but even in our current state it is definitely a useful tool for development.