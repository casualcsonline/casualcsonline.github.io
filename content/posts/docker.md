---
title: "Docker 101"
date: 2021-02-24T11:24:33-08:00
author_name: Tyler Lubeck
author_email: tyler+casualcs@tylerlubeck.com
tags:
  - docker
  - containers
  - intro
draft: true
---

I've recently had a handful of discussions with folks about what Docker is and how it works. The world **definitely** needs another "getting started with docker" guide, and I figured it was high time I write one up too.

<!--more-->

## What We'll Cover

By the end of this article, you should have a decent grasp of how Docker images and containers work under the hood. We'll cover:

- How a docker container works.
- The relationship between a docker image and a docker container.
- Why Docker might be helpful for you.
- How to create a basic docker image.

## What We Won't Cover

- How Docker works for networks, and how containers can talk to each other.
- How the Docker Daemon actually runs containers. We won't even explain what the Docker Daemon is.
- Any information about other container runtimes.
- How any of this works on operating systems that aren't linux.

## Definitions

Before we dive in too deeply, let's make sure we're all speaking the same language.

Image
: An "Image" is a cookie cutter for a docker container. When you have an image, you can stamp out as many containers based on that image as you like.

Container
: A "Container" is an instance of your "Image". Following the "cookie cutter" comparison, this is a single cookie. It may look and act like all of the other cookies on your baking sheet, but it acts as its own independent morsel.

Dockerfile
: A set of instructions to create an "Image". Think of it like a cookie recipe.

Namespace
: A "Namespace" is a way to isolate your cookies from your neighbor's cookies. Everything we need to know about our delicious chocolate chip cookies are contained in a set of namespaces, and within our namespaces we don't even know that Jim's oatmeal raisin cookies exist.

Container Runtime
: Really, Docker is just one type of "Container Runtime". Much like you can write the same program in almost any language, you can create and run containers in several different runtimes. For today, we'll acknowledge that these other runtimes exist and then go right back to ignoring them.


## What is Docker, really?

The thing we know as "Docker" is just some fancy UX around a technology known as Namespaces. Namespaces are, basically, boxes made of fancy one-way mirrors. When you put things in this box, they can't see out and only things that aren't in other boxes can see in.

Docker does a great job of combining a few types of boxes and making it easy for us to put things in them. The main box we're concerned with today is known as a "process namespace" - any running process in this box is only aware of other processes if they're also inside of this box. We'll also talk about a "mount namespace" or "filesystem namespace"[^filesystem-namespace] - any running process in this box is only aware of files that we've placed inside of this box.

Let's go back to our cookie cutter example. If a docker image is a cookie cutter, than a computer is made up of cookie dough. We can take our cookie cutter and stamp it down on our cookie dough and BAM, we have a gingerbread person. Likewise, we can take our docker image and stamp it down on our computer and BAM we have a container with its associated process and filesystem namespaces. If we take our cookie cutter and put it away for the season[^cookie-cutters-in-july], we can take it out again next year and stamp it down _in a new batch of dough_ and get _a gingerbread person of the exact same shape_. Likewise, we can take our docker image over to somebody else's computer and run a container _that looks exactly the same_.

### How is this different from a virtual machine?

Sticking with our analogy, starting a virtual machine is like creating a whole new batch of cookie dough. A docker image assumes you already have some cookie dough around, and is happy to make use of what you've got. 

The big impact here is that it's _wicked_ fast to start up a container from an image, whereas starting a virtual machine takes a fair bit of time. Containers also require far fewer resources than virtual machines, since they play nicely with the operating system they're running under rather than having to create an entirely new operating system.

## Why does any of this matter?


I think that the most compelling reason to start using docker is that it makes it trivial to run your code in different places without any changes. If you have a docker image that works on your computer, it'll almost certainly work on somebody else's computer as well.

This means no more mucking around with code dependencies when you try to run your code somewhere else. So long as you've set up your cookie cutter docker image with the versions of the dependencies you need, you can stamp out a docker container on another machine and have it start right up.


## Your first image and container

The starting point for any image is called a `Dockerfile`. It acts as the recipe for your cookies, and can be used to make cookie dough whenever you like. From this `Dockerfile`, we can create a docker Image that we'll eventually run as a docker Container.


### The Dockerfile

Let's set up a basic dockerfile and walk through what happens:

```dockerfile
FROM ubuntu             # Line 1
ENTRYPOINT ["cat"]      # Line 2
CMD ["/etc/issue"]      # Line 3
```

On Line 1, we specify a jumping off point for our recipe. This is like saying "go get some flour and some chocolate chips" - you don't need to grow wheat or cacao trees, you just need some of the finished products so you can combine them in a way you like.

On Line 2, we tell the image what the container should do when it starts up. This is like preheating your oven - get some things set up so you can get your cookies baking easily. Note that the **Image** *doesn't actually run this command*. It just tells sets us up so that when we eventually create a **Container** from this Image, that **Container** will run this command.

On Line 3, we specify some default arguments that will be passed to our Entrypoint when the Container eventually starts. These arguments can be overriden by the Container when it starts up. In this particular instance, I've chosen the name of the first file I can think of that will definitely exist in this image[^please-run-this] - there's nothing special about this particular file.

### From a Dockerfile to a Docker Image

To create an **Image** from this `Dockerfile`, we'll run the following in our shell. If you want to actually run it, you'll likely have to remove the comments:
```bash
docker build \              # Line 1
    --file Dockerfile \     # Line 2
    --tag chocolatechip \   # Line 3
    .                       # Line 4
```

On Line 1, we tell docker we want to build some stuff. 

On Line 2, we tell docker the recipe we want to build from. `Dockerfile` is actually the default here, so we don't _have_ to say this, but I like being explicit.

On Line 3, we tell docker what we want to "tag", or "name" the resulting image.

On Line 4, we tell docker the "context" we want to use to build our image. We're going to gloss over this - just use `.` (the current directory) for now.

When you run the command, you'll see output very similar to the following:

```
❯ docker build --file Dockerfile --tag chocolatechip .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM ubuntu
latest: Pulling from library/ubuntu
83ee3a23efb7: Pull complete
db98fc6f11f0: Pull complete
f611acd52c6c: Pull complete
Digest: sha256:703218c0465075f4425e58fac086e09e1de5c340b12976ab9eb8ad26615c3715
Status: Downloaded newer image for ubuntu:latest
 ---> f63181f19b2f
Step 2/3 : ENTRYPOINT ["cat"]
 ---> Running in 6216f1bc7328
Removing intermediate container 6216f1bc7328
 ---> 16e1ee9a39fb
Step 3/3 : CMD ["/etc/issue"]
 ---> Running in b50691f519fd
Removing intermediate container b50691f519fd
 ---> d9c66e31bdfa
Successfully built d9c66e31bdfa
Successfully tagged chocolatechip:latest
```

We now have a docker image called `chocolatechip`! Notice that each Step corresponds to a line in our `Dockerfile`. Also notice that the very first step resulted in some stuff being "Pulled" - this is docker going to the grocery store to buy flour and chocolate chips.

### From a Docker Image to Docker Containers

Now that we have a docker image, we can run it as many times as we like:

```
❯ docker run chocolatechip
Ubuntu 20.04.1 LTS \n \l

❯ docker run chocolatechip
Ubuntu 20.04.1 LTS \n \l

❯ docker run chocolatechip
Ubuntu 20.04.1 LTS \n \l

❯ docker run chocolatechip
Ubuntu 20.04.1 LTS \n \l
```

Each time, the output is identical to the previous time. The command also completes nearly instantly! All of the hard work of going to get flour and chocolate chips is already done - we just have to stamp out cookies.

Now we can check to see that we actually made a bunch of different cookies:

```
❯ docker ps --all  # List all containers that are running or have run in the past
CONTAINER ID   IMAGE           COMMAND            CREATED              STATUS                          PORTS     NAMES
e969783110b9   chocolatechip   "cat /etc/issue"   About a minute ago   Exited (0) About a minute ago             reverent_williams
9df76df03001   chocolatechip   "cat /etc/issue"   About a minute ago   Exited (0) About a minute ago             pedantic_keller
caebbe2d859a   chocolatechip   "cat /etc/issue"   About a minute ago   Exited (0) About a minute ago             xenodochial_johnson
96f65378d930   chocolatechip   "cat /etc/issue"   About a minute ago   Exited (0) About a minute ago             jolly_neumann
a71ff73e2c32   chocolatechip   "cat /etc/issue"   About a minute ago   Exited (0) About a minute ago             jovial_wescoff 
```

We have a bunch of different containers that are all based on our `chocolatechip` image, and have run the same COMMAND - `cat /etc/issue`, a combination of the `ENTRYPOINT` and `CMD` steps from `Dockerfile`, just as we expected.

### Doing something slightly different

Now that we have our `chocolatechip` cookie cutter, we might want to deviate _just a little bit_ from our main shape - add some sprinkles or something. That's why we specified the file we wanted to view as a `CMD`. We can swap out the `CMD` by passing some more arguments to `docker run`:

```
❯ docker run chocolatechip /etc/debian_version
bullseye/sid
```

And listing our docker containers again:
```
❯ docker ps --all
CONTAINER ID   IMAGE           COMMAND                  CREATED              STATUS                        PORTS     NAMES
fd6f86dd15a2   chocolatechip   "cat /etc/debian_ver…"   18 seconds ago       Exited (0) 18 seconds ago               jolly_solomon
e969783110b9   chocolatechip   "cat /etc/issue"         6 minutes ago        Exited (0) 6 minutes ago                reverent_williams
9df76df03001   chocolatechip   "cat /etc/issue"         6 minutes ago        Exited (0) 6 minutes ago                pedantic_keller
caebbe2d859a   chocolatechip   "cat /etc/issue"         6 minutes ago        Exited (0 6 minutes ago                 xenodochial_johnson
96f65378d930   chocolatechip   "cat /etc/issue"         6 minutes ago        Exited (0) 6 minutes ago                jolly_neumann
a71ff73e2c32   chocolatechip   "cat /etc/issue"         7 minutes ago        Exited (0) 7 minutes ago                jovial_wescoff
```

And we see that we have a new container, still based on our `chocolatechip` cookie cutter image, but doing something just _slightly_ different.



## Wrap Up

And that's our whirlwind tour of ~~baking~~ docker! By now you should know what some of the more common 


[^filesystem-namespace]: While I can't find any evidence that the term "filesystem namespace" is _wrong_, I also can't find any that it's _correct_. It doesn't require me to explain what a "mount" is though, so I'm sticking with it.
[^cookie-cutters-in-july]: Cookie Cutters are only useful in December, right?
[^please-run-this]: I want you to be able to run the examples I show here
