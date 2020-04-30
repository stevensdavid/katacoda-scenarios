

How to setup your own Git server

Let's start by changing the prompts of our client and server so we can tell them apart:

`PS1="server $ "`{{execute T1}}

`PS1="client $ "`{{execute T2}}

## Setting up the server
First install `git-core` on the server machine:

`apt update && apt-get install -y git-core`{{execute T1}}

In order to allow clients to connect to the server, we need to setup SSH access. Start by installing a SSH server.

`apt install -y openssh-server`{{execute T1}}

Make sure SSH is running: 

`service ssh start`{{execute T1}}

Next add a user for the Git server and a password. The following command will set the password to `password123`:

`useradd -m git && echo 'git:password123' | chpasswd `{{execute T1}}

Let's change user to the newly created `git` account.

`su git`{{execute T1}}

Next create a SSH key for your new user. This command will place the keys in `~/.ssh` and leave the passphrase empty. It will replace any existing keys. On your real server, the passphrase should be made secure.

`ssh-keygen -t rsa -q -N '' -f ~/.ssh/id_rsa <<< y`{{execute T1}}

We can leave the `git` account now.

`exit`{{execute T1}}

## Create a repository
In order to create new repositories through the CLI, we have to create them manually on the server. Luckily, we can do this over SSH. Start by logging into the server from the client:

`ssh git@localhost`{{execute T2}}

Create a new folder and cd to it:

`mkdir -p /home/git/project-1.git && cd /home/git/project-1.git`{{execute T2}}

Now we have to initialize the folder as a bare Git repository. This means that it won't have a working tree - perfect for a server, since we don't want to make any changes to the repository from the server!

`git init --bare`{{execute T2}}

Now we want to push something to our new Git repository on our local machine. Let's logout of the server.

`exit`{{execute T2}}

On the client machine, install the Git client:

`apt install git`{{execute T2}}

Create a new folder, cd to it and initialize it as a regular Git repository:

`mkdir -p ~/git/project && cd ~/git/project`{{execute T2}}

`git init`{{execute T2}}

Create a sample file: 

`echo 'Hello' > greeting.txt`{{execute T2}}

Setup git on the client by letting it know who you are:

`git config --global user.name "Your Name" && git config --global user.email "you@example.com"`{{execute T2}}

Add the file and commit it:

`git add greeting.txt && git commit -m "Greeting added!"`{{execute T2}}

Next add the server as a remote and push your changes to it. Note that we use the file path to the repository on the remote:

`git remote add origin ssh://git@localhost/home/git/project-1.git`{{execute T2}}

`git push origin master`{{execute T2}}

The changes are now stored on the server! Test it by cloning the repository to a new folder on the client, such as `project-copy`:

`cd .. && git clone ssh://git@localhost:/home/git/project-1.git project-copy`{{execute T2}}

See that everything is there! 
 
`cd project-copy && cat greeting.txt`{{execute T2}}
