

How to setup your own Git server

## Setting up the server
First install `git-core on Host 1:

`sudo apt update && sudo apt-get install git-core`{{copy}}

In order to allow clients to connect to the server, we need to setup SSH access. Start by installing a SSH server.

`sudo apt install openssh-server`{{copy}}

Make sure SSH is running: 

`service ssh status`{{copy}}

We also need to allow SSH access through the Ubuntu firewall:

`sudo ufw allow ssh`{{copy}}

Next add a user for the Git server and a password:

`sudo useradd -m git && passwd git`{{copy}}

Let's change user to the newly created `git` account.

`sudo su git`{{copy}}

Next create an ssh key for your new user. The command will ask for a file location and a passphrase: accept the default file and leave the passphrase empty. In your real server, the passphrase should be made secure.

`ssh-keygen -t rsa`{{copy}}


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
