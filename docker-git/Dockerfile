# This is your Editor pane. Write the Dockerfile here and 
# use the command line to execute commands
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y git-core && apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
RUN useradd -m git && echo 'git:password123' | chpasswd
USER git
RUN mkdir -p ~/.ssh && ssh-keygen -q -t rsa -N '' -f ~/.ssh/id_rsa 
USER root
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile
EXPOSE 22 
CMD ["/usr/sbin/sshd", "-D"]