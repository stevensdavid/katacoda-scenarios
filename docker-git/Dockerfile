# Download base image ubuntu 18.04
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y git-core && \
    apt-get install -y openssh-server
RUN mkdir /var/run/sshd    
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
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

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
# Listen for SSH connections
EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]