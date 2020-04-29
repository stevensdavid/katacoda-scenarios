

How to setup your own Git server

## Setting up the server

First install `git-core on Host 1:

`sudo apt-get install git-core`{{copy}}

Next add a user and a password:

`sudo useradd git && passwd 123`{{copy}}

Next add create an ssh key for your new user, accept the default settings:

`ssh-keygen -t rsa`{{copy}}

Make sure SSH is running: 

`service ssh status`{{copy}}

## Adding an authorized user
On host 2, add your public key to the repo:

`cat ~/.ssh/id_rsa.pub | ssh git@remote-server "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"`{{copy}}

## Create a repository

On host 1, create a new folder, cd to it:

`mkdir -p /home/git/project-1.git && cd /home/git/project-1.git`{{copy}}

Now we have to initialize the folder as a bare Git repository. This means that it won't have a working tree - perfect for a server, since we don't want to make any changes to the repository from the server!

`git init --bare`{{copy}}

Now we want to push something to our new Git repository on our local machine.

On host 2, create a new folder, cd to it and intialize it as a regular Git repository:

`mkdir -p ~/git/project && cd ~/git/project`{{copy}}

`git init`{{copy}}

Create a sample file: 

`echo 'Hello' > greeting.txt`{{copy}}

Add it and commit it:

`git add greeting.txt && git commit -m "Greeting added!"`{{copy}}

Next add the server as a remote and push your changes to it:

`git remote add origin ssh://git@localhost/home/git/project-1.git`{{copy}}

`git push`{{copy}}

The changes are now stored on the server! Test it by cloning:

`cd .. && git clone ssh://git@localhost:/home/git/project-1.git project-copy`{{copy}}

See that everything is there! 
 
`user@localhost:~ $ cd project-copy && cat greeting.txt`{{copy}}