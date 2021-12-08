## BlackArch Docker Setup Guide

### Thesis: There are too many guides about Kali Linux Docker Containers
So here's another guide that does essentially the same thing

---

#### Pull the Docker Container

<br>

For reference, I am going to be using this specific container by the Offical BlackArch Team.

https://hub.docker.com/r/blackarchlinux/blackarch

<br>

```
❯ docker pull blackarchlinux/blackarch
Using default tag: latest
latest: Pulling from blackarchlinux/blackarch
63cb4f2464b7: Pull complete
Digest: sha256:d3473de10204177cb1f1a61a71fbb162530aa508f1c3c99ac2fc44ec4a8ffeb2
Status: Downloaded newer image for blackarchlinux/blackarch:latest
docker.io/blackarchlinux/blackarch:latest

❯ docker images
REPOSITORY                     TAG       IMAGE ID       CREATED        SIZE
blackarchlinux/blackarch       latest    4a676bf8bd5a   2 days ago     385MB
~ ❯
```

<br>

#### Starting the BlackArch Container

<br>

For this we need to specify the type of Docker session we want, for setup want an interactive session. Later you can set it up so that just the tools are connected to your host machine (if you run Linux/MacOS). 

<br>

```
❯ docker run --tty --interactive --rm blackarchlinux/blackarch /bin/bash
[ c3ca2e979c4d / ]#
```
<br>
Now this is a pretty good start, but if you mess around a bit you'll quickly realize there a no tools? So we're going to install right now, and if there are any other apps you want like vim or tmux feel free to install those too.

<br>





