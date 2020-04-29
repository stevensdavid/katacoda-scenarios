# Dockerize it!
In order to make the server configuration more maintainable and consistent between machines, it can be beneficial to contain it within a Docker image. This process essentially consists of containerizing the commands that were executed directly on the server in the previous step.

## Writing the Dockerfile
The first step of creating the Dockerfile is deciding which operating system to run the Git server on. For this tutorial, we will use Ubuntu 18.04. This is done by adding the line `FROM ubuntu:18.04` to the top of the Dockerfile

`echo 'FROM ubuntu:18.04' > Dockerfile`{{execute T1}}

Next, we need to install our dependencies: the git core server software and SSH server:

`echo 'RUN apt-get update' >> Dockerfile`{{execute T1}}
`echo 'RUN apt-get install -y git-core && apt-get install -y openssh-server' >> Dockerfile`{{execute T1}}

We now have to create the `git` user. Since we cannot provide input to the commands during the Docker build process, we have to set the password in the Dockerfile. This is done with the `chpasswd` command.

`echo 'RUN useradd -m git && echo 'git:password123' | chpasswd' >> Dockerfile`{{execute T1}}

At this point, we should change user to the `git` user in order to create the appropriate SSH keys. When generating the keys, we use the command line flags to provide input instead of relying on the CLI of `ssh-keygen`.

`echo 'USER git' >> Dockerfile`{{execute T1}}

`echo 'RUN mkdir -p ~/.ssh && ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa' >> Dockerfile`{{execute T1}} 

In order to start the SSH server, we have to change back to the root user.
`echo 'USER root'>> Dockerfile`{{execute T1}}

Finally, we expose port 22 to allow incoming SSH connections and start the SSH server using `sshd`.

`echo 'EXPOSE 22' >> Dockerfile`{{execute T1}}
`echo 'CMD ["/usr/sbin/sshd", "-D"]'>> Dockerfile`{{execute T1}}

Our complete Dockerfile is now
```Dockerfile
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y git-core && apt-get install -y openssh-server
RUN service ssh start
RUN useradd -m git && echo 'git:password123' | chpasswd
USER git
RUN mkdir -p ~/.ssh && ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa
USER root
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```

Verify that this is the case by running
`cat Dockerfile`{{execute T1}}

## Building and running the Docker image
The Dockerfile can be built by executing
`docker build -t git-container .`{{execute T1}}

Once it is done, we can run the container by executing
`docker run -p 22:22 git-container`{{execute T1}}

The flag `-p 22:22` maps port 22 of the container to port 22 of the machine itself, meaning that any SSH connections to the server will be forwarded to the container. This means that it will no longer be possible to SSH directly to the server, which can make it more difficult to maintain. As such, it might be worth using a different port for the container. The consequence of this would be that the client would need to connect to `git:portXX@host-ip` instead of `git@host-ip`. 

## Configuring the client
The client is then configured in the same way as when the server was not running in a container. 

`cat ~/.ssh/id_rsa.pub | ssh git@localhost "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys" && ssh git@localhost "mkdir -p /home/git/project-1.git && cd /home/git/project-1.git && git init --bare" && sudo apt install git && mkdir -p ~/git/project && cd ~/git/project && git init && git config --global user.name "Your Name" && git config --global user.email "you@example.com"`{{execute T2}}

Create a file, add it and commit it.
`echo 'Hello' > greeting.txt && git add greeting.txt && git commit -m "Greeting added!"`{{execute T2}}

As the client is not able to know that the Git server is running in a container, we add it as a remote the same way as before. This is where we would have to specify the port if a different mapping was used.

`git remote add origin ssh://git@localhost/home/git/project-1.git`{{execute T2}}

Finally, we can push the changes to the server.

`git push origin master`{{execute T2}}

The repository has now been updated in the container, and is once again visible for any other clients.

`cd .. && git clone ssh://git@localhost:/home/git/project-1.git project-copy`{{execute T2}}

`cd project-copy && cat greeting.txt`{{execute T2}}
