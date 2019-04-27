---
layout: post
title:  "Server Setup"
date:   2017-02-01 02:59:59
categories: Basic_Server_Setup
---

Hello Guys, 
Today we will learn about the basic server commands, sudo user creation, ssh properties, tmux and many more. This blog-post is for those who are new to server world .

<h2>Prerequisite</h2>
DigitalOcean droplet with root access.

<h2>SSH</h2>

Lets start with the very first basic command to enter your server.

<code>ssh root@(your ip-address)</code>

Two possible conditions:
Firstly, it will prompt for the root password, enter the password you assign during your account creation.
Secondly, if your have already added your <a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2">ssh-key</a>(your system public-key) during the droplet creation, then it wont ask for the password and you will see the output like this:

{% highlight ruby%}
The authenticity of host '12.34.56.78 (12.34.56.78)' can't be established.
RSA key fingerprint is b1:2d:33:67:ce:35:4d:5f:f3:a8:cd:c0:c4:48:86:12.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '12.34.56.78' (RSA) to the list of known hosts.
user@12.34.56.78's password: 
Now try logging into the machine, with "ssh 'user@12.34.56.78'", and check in:

  ~/.ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.
{% endhighlight %}

Press enter, you will now be inside your server.

If you see the above message carefully, it has added fingerprint of the server to your local machine, you can see that fingerprints inside 
<code>nano .ssh/known_hosts</code>

Now, first we will see how our  ssh-config works inside the server, 

<code>nano /etc/ssh/sshd_config</code>

<code>Port 22</code> : specifies the ssh port, you can modify your ssh port , say 1432

Now, restart the ssh service.

Now when you try to login to your server you have to pass new ssh part. 

E.g:

<code>ssh root@12.13.x.x -p 1432</code>

Now, we will disable the root login to enhance our security , but before doing that we will create a sudo user .

<h2>Sudo User Creation</h2>

If this is your first new user, and you are currently logged in as the root user, you can use the following syntax to create a new user:

<code>adduser newuser</code>

If you are logged into a user that you added previously and gave sudo privileges, you can create a new user by invoking sudo with the same command:

<code>sudo adduser newuser</code>

Either way, Debian will prompt you for more information about the user you are creating. The first piece of information you need to choose is the password for the new user.

It will ask you to select a password and then confirm it by repeating it.

Afterwards, it will ask you for personal information about the user. You can feel free to fill this out or to leave it blank. The user will operate in entirely the same way regardless of your decision.

Now, the newuser , just created , wont have the administrative rights, so if the user you created is a primary user, then we have to grant that user Administrative Priviledges.

<h2>Grant Users Administrative Privileges</h2>

If you are logged in as a root, then type:
<code>visudo</code>

When you type that command, you will be taken into a text editor with the file where we have to add our new user to grant our access rights.

Now add the line 
<code>newuser    ALL=(ALL:ALL) ALL</code> 

at the end of line.

Now once you logout you can ssh to your server with:

<code>ssh newuser@12.34.x.x -p 1432</code>

To delete a user :

As a regular user with sudo privileges, you can delete a user using this syntax:

<code>sudo deluser --remove-home username</code>

If you are logged in as root, you do not need to add the sudo before the command:

<code>deluser --remove-home username</code>

Once your sudo user is in place, you can edit the sshd_config file and restrict the root_login.

Note: Be sure before you make any change to ssh properties

You can switch between users with the command:

<code>sudo su - (username)</code>

Now, working inside a server terminal is very irritating, so to ease with your work, there is a terminal multiplexer: TMUX

<h2>TMUX</h2>

I will show you the basic commands for tmux, that are usefull for day to day operations:

Its a good practice to start tmux session as a root user:

If you are not root user , then first switch to root : 

<code>sudo su</code>

Now, start tmux session using:

<code>tmux</code>

At the bottom of the screen you can see the active terminal as <code>0:bash*</code>

Lets add a window to your tmux session:

<code>ctrl-b c</code>

At bottom you can see that one more window is added to the session.

Now to switch between the windows:

<code>ctrl-b (no. assign to window)</code>

Eg. <code>ctrl-b 0</code> or <code>ctrl-b 1</code>

To kill the particular window, first switch to that window:
then

<code>ctrl-b x</code>

To detach from the tmux session,

<code>ctrl-b d</code>

Note: this command wont kill your tmux session, its just that you will be logout of that session.

To again login into the same session, do 
<code>tmux ls</code>

Now attach to your desired tmux session, 

<code>tmux attach -t (session name)</code>

And you will be login again to the same tmux session. 

Thats all for the basic tmux working.

You can even assign name to your tmux session.

<code>tmux new -s session_name</code>

For ansible deplyment srcipt follow this <a href="https://github.com/sumitgupta0001/ansible-scripting">link . </a> Check for the user_creation, ssh_port modification roles.

