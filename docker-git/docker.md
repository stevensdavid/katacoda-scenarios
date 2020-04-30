# Dockerize it!
In order to make the server configuration more maintainable and consistent between machines, it can be beneficial to contain it within a Docker image. This process essentially consists of containerizing the commands that were executed directly on the server in the previous step.

## Writing the Dockerfile
The first step of creating the Dockerfile is deciding which operating system to run the Git server on. For this tutorial, we will use Ubuntu 18.04. This is done by adding the line `FROM ubuntu:18.04` to the top of the Dockerfile. You can do this in the editor window.

`FROM ubuntu:18.04`{{copy}}

Next, we need to install our dependencies: the git core server software and SSH server:

`RUN apt-get update`{{copy}}

`RUN apt-get install -y git-core && apt-get install -y openssh-server`{{copy}}

As we are going to use the SSH daemon to accept incoming connections, we have to create the directory it requires.

`RUN mkdir /var/run/sshd`{{copy}}

Running a SSH daemon in a docker container has one peculiarity: by default, the user will be kicked off after login. This is fixed by the following line:

`RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd`{{copy}}

We now have to create the `git` user. Since we cannot provide input to the commands during the Docker build process, we have to set the password in the Dockerfile. This is done with the `chpasswd` command.

`RUN useradd -m git && echo 'git:password123' | chpasswd`{{copy}}

At this point, we should change user to the `git` user in order to create the appropriate SSH keys. When generating the keys, we use the command line flags to provide input instead of relying on the CLI of `ssh-keygen`. Of course, on a real server you should enter a secure passphrase by specifying it in after the `-N` flag instead of leaving it empty as we have done here.

`USER git`{{copy}}

`RUN mkdir -p ~/.ssh && ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa`{{copy}} 

In order to start the SSH server, we have to change back to the root user.
`USER root`{{copy}} 

The SSH daemon clears the environment variables when it is run. In order to make the environment variables in the Dockerfile available we have to save them in the user's profile:

`ENV NOTVISIBLE "in users profile"`{{copy}}

`RUN echo "export VISIBLE=now" >> /etc/profile`{{copy}}

Finally, we expose port 22 to allow incoming SSH connections and start the SSH server using `sshd`.


`EXPOSE 22`{{copy}} 

`CMD ["/usr/sbin/sshd", "-D"]`{{copy}} 

Our complete Dockerfile is now
```Dockerfile
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y git-core && \
    apt-get install -y openssh-server
RUN mkdir /var/run/sshd    
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
RUN useradd -m git && \
    echo 'git:password123' | chpasswd
USER git
RUN mkdir -p ~/.ssh && \
    ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa
USER root
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```

## Building and running the Docker image
**Sadly, Katacoda does not appear to allow dockerized SSH services.** Therefore, running the docker image will not work in the web client. However, it should work on your local machine.

The Dockerfile can be built by executing
`docker build -t git-container .` locally.

Once it is done, we can run the container by executing
`docker run -d -P --name git-server git-container` locally.

The flag `-P` published all exposed ports (in our case, port 22) through the host interface. We can find the port mapping by running

`docker port git-server 22`

and the IP address of the container by running

`sudo docker inspect -f "{{ .NetworkSettings.IPAddress }}" git-server`

## Configuring the client
The client is then configured in the same way as when the server was not running in a container. All interactions between the client and the server work in the same way. The only difference is that the IP address and port that the client connects to should be the ones that were found with the Docker CLI.
