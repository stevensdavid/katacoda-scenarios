# Download base image ubuntu 18.04
FROM ubuntu:18.04
RUN apt update
RUN apt install -y git-core && \
    apt install -y openssh-server
RUN useradd -m git && \
    echo 'git:password123' | chpasswd
# Change to the git user with fewer privileges
USER git
# Creates a private key. 
# More secure would be to store it outside container and use VOLUME,
# as this will store the generated key as part of the image.
RUN mkdir -p ~/.ssh && \
    ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa
USER root
# Listen for SSH connections
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]