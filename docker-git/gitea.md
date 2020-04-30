# GUI it!

https://[[HOST_SUBDOMAIN]]-80-[[KATACODA_HOST]].environments.katacoda.com

`export GITLAB_HOME=/srv`{{execute T1}}

`docker run --detach --hostname gitlab.example.com --publish 443:443 --publish 80:80 --publish 22:22 --name gitlab --restart always --volume $GITLAB_HOME/gitlab/config:/etc/gitlab --volume $GITLAB_HOME/gitlab/logs:/var/log/gitlab --volume $GITLAB_HOME/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:latest`{{execute T1}}