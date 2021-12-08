## BlackArch Docker Setup Guide

### Thesis: There are too many guides about Kali Linux Containers
So here's another guide that does essentially the same thing but take up 4x the space

---

#### Pull the Docker Container

<br>

For reference, I am going to be using this specific container by the Offical BlackArch Team.

[https://hub.docker.com/r/blackarchlinux/blackarch](https://hub.docker.com/r/blackarchlinux/blackarch)

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


For this we need to specify the type of Docker session we want, for setup want an interactive session. Later you can set it up so that just the tools are connected to your host machine (if you run Linux/MacOS). 
<br>

```
❯ docker run --tty --interactive --rm blackarchlinux/blackarch /bin/bash
[ c3ca2e979c4d / ]#
```
<br>
Now this is a pretty good start, but if you mess around a bit you'll quickly realize there a no tools? So we're going to install right now, and if there are any other apps you want like vim or tmux feel free to install those too. Before we run the BlackArch install, keep in mind the default installation take 60GB of space, so just make sure you have at least 120GB of space before starting. 

<br>

#### Downlading a Couple Tools


```
[ c3ca2e979c4d / ]# pacman -Syu --needed --overwrite='*' blackarch

:: Synchronizing package databases...
 core is up to date
 extra is up to date
 community is up to date
 multilib is up to date
 blackarch is up to date
:: There are 2739 members in group blackarch:
:: Repository blackarch

-- SNIP --

Total Download Size:   16261.69 MiB
Total Installed Size:  56558.75 MiB
Net Upgrade Size:      56558.23 MiB

:: Proceed with installation? [Y/n] Y
```


You might want to get some coffee or go to sleep, this is gonna take a while to download. 



Once this is over, its time to commit and save your BlackArch container. Pop open a new terminal session and use $ <i>docker ps </i> to take view your running docker containers. Then make note of the container ID to create a new image based off of your installation.
<br>

```
❯ docker ps
CONTAINER ID   IMAGE
c3ca2e979c4d   blackarchlinux/blackarch
❯ 
~ ❯ docker commit c3ca2e979c4d joeylipton/blackarch:v1
```
<br>

#### Running BlackArch 

And there you go, anytime you need to spin up a set of tools for a ctf:

```
docker run -it  --network=host <username>/blackarch /bin/bash
```

<br>

#### Saving Alias in CLI
This step is just to save the 
```
echo "alias blackarck='docker run -it  --network=host <username>/blackarch /bin/bash'" >> ~/.bashrc

# Or if you use zsh 

echo "alias blackarch='docker run -it  --network=host <username>/blackarch /bin/bash'" >> ~/.zsh

```