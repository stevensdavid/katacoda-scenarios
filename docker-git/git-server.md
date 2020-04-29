

How to setup your own Git server

## Setting up the server
First install `git-core` on the server machine:

`sudo apt update && sudo apt-get install -y git-core`{{execute T1}}

In order to allow clients to connect to the server, we need to setup SSH access. Start by installing a SSH server.

`sudo apt install -y openssh-server`{{execute T1}}

Make sure SSH is running: 

`sudo service ssh start`{{execute T1}}

Next add a user for the Git server and a password:

`sudo useradd -m git && passwd git`{{execute T1}}

Let's change user to the newly created `git` account.

`sudo su git`{{execute T1}}

Next create an ssh key for your new user. The command will ask for a file location and a passphrase: accept the default file and leave the passphrase empty. On your real server, the passphrase should be made secure.

`ssh-keygen -t rsa`{{execute T1}}

## Adding an authorized user
In order to connect to the Git server without having to enter its password, let's add the client's public SSH key to the list of authorized keys on the server. Start by generating a SSH key on the client if you do not already have one:

`ssh-keygen -t rsa`{{execute T2}}

As before, the default settings are sufficient for this tutorial. In real life you'll want a more secure passphrase.

Next, add the client's public key to the server:

`cat ~/.ssh/id_rsa.pub | ssh git@localhost "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"`{{execute T2}}

Of course, `localhost` would be replaced with the IP address of your server if this were to be replicated on a real machine.

## Create a repository
In order to create new repositories through the CLI, we have to create them manually on the server. Luckily, we can do this over SSH. Start by logging into the server from the client:

`ssh git@localhost`{{execute T2}}

Create a new folder and cd to it:

`mkdir -p /home/git/project-1.git && cd /home/git/project-1.git`{{execute T2}}

Now we have to initialize the folder as a bare Git repository. This means that it won't have a working tree - perfect for a server, since we don't want to make any changes to the repository from the server!

`git init --bare`{{execute T2}}

Now we want to push something to our new Git repository on our local machine. Let's logout of the server.

`exit`{{execute T2}}

On the client machine, create a new folder, cd to it and intialize it as a regular Git repository:

`mkdir -p ~/git/project && cd ~/git/project`{{execute T2}}

`git init`{{execute T2}}

Create a sample file: 

`echo 'Hello' > greeting.txt`{{execute T2}}

Add it and commit it:

`git add greeting.txt && git commit -m "Greeting added!"`{{execute T2}}

Next add the server as a remote and push your changes to it:

`git remote add origin ssh://git@localhost/home/git/project-1.git`{{execute T2}}

`git push`{{execute T2}}

The changes are now stored on the server! Test it by cloning the repository to a new folder on the client, such as `project-copy`:

`cd .. && git clone ssh://git@localhost:/home/git/project-1.git project-copy`{{execute T2}}

See that everything is there! 
 
`cd project-copy && cat greeting.txt`{{execute T2}}
