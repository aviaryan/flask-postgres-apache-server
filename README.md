# DO flask postgres server

In this tutorial, we will deploy a flask server project to Digital Ocean.
We will setup an internal postgres server as the database and demonstrate some advanced server management magic.

**Note** - Here we are taking a project named "catalog". You should replace "catalog" with *your project name* everywhere in this article.


## Table of Contents

1. Start a new server on Digital Ocean.
2. SSH into the server.
3. Update all currently installed packages.
4. Change the SSH port from 22 to 2200. Make sure to configure the firewall to allow it.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
6. Create a new user account named `grader`.
7. Give `grader` the permission to sudo.
8. Create an SSH key pair for `grader` using the ssh-keygen tool.
9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.
11. Install and configure PostgreSQL.
	- Do not allow remote connections.
	- Create a new database user named `catalog` that has limited permissions to your `catalog` application database.
12. Install git.
13. Clone and setup your git project (here Catalog).
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser.


<a name="step1"></a>
## Step 1:

To start a new server on Digital Ocean, create a new droplet. Use the following specifications -

- Distribution - Ubuntu 16.04 x64
- Size - `$5/mo`
- Datacenter region - San francisco
- Create a new ssh-key called `catalog_root`.
	- Open your terminal and run `ssh-keygen`.
	- You will be asked for the file path to save your key. Enter `/Users/USER_NAME/.ssh/catalog_root` where *USER_NAME* is your username.
	- Set up a passphrase, here we are setting it to "password".
	- Done, you should have your key pair now.
- In the DO interface, click on *New SSH key* and paste the contents of `~/.ssh/catalog_root.pub` in it. Name the key as `catalog_root`. Enter.
- Keep number of droplets to `1`.
- Finish. Click on Create button.


## Step 2:

- To ssh into the server, copy the IP address from Digital Ocean interface. Here it is `165.227.16.72`.

![DO dashboard screenshot](https://i.imgur.com/QyzJMIC.png)

- ssh into the server with the following command.

```sh
ssh -i ~/.ssh/catalog_root root@165.227.16.72
# ssh -i /full/path/to/key root@IP
```

- You will have to enter the passphrase for the ssh key that you created. Enter it and you should be connected.
- Try running `pwd` to check if things are working. Congrats, you officially now have access to your server.

![ssh into server view](https://i.imgur.com/w2fRR9e.png)


## Step 3:

To update currently installed packages, run the following command-

```sh
sudo apt-get update
sudo apt-get upgrade
# sudo is actually not required in case of DO but never mind :)
```


## Step 4:

To change ssh port to 2200, follow the given steps-

- Open ssh config in `nano`.

```sh
sudo nano /etc/ssh/sshd_config
```

- Locate "Port 22" in that file. Change it to "Port 2200".

- Restart sshd service.

```sh
sudo service ssh restart
```


## Step 5:

To configure UFW to allow given connections, follow the steps:

```sh
sudo ufw allow 2200
sudo ufw allow 80
sudo ufw allow 123
```

Then enable ufw.

```sh
sudo ufw enable
```

Now ssh port has been changed to 2200. Try exiting the ssh connection and re-connecting with the following command.

```sh
ssh -p 2200 -i ~/.ssh/catalog_root root@165.227.16.72
```

Notice the `-p 2200` there. Clean, isn't it.


## Step 6:

To create a user called `grader`, run the following command.

```sh
sudo adduser grader
```

Fill in the details, use `password` as the password for this user.

To check if the new user has been created successfully or not, run `ls /home/grader`. If the command exits without any problems, then that means
the path is valid.


## Step 7:

To give `grader` sudo permission, we first create `grader` file inside `sudoers.d`.

```sh
touch /etc/sudoers.d/grader
```

Then we open the file in `nano` (`sudo nano /etc/sudoers.d/grader`) and add the following-

```sh
grader ALL=(ALL) NOPASSWD:ALL
```


## Step 8:

We will now have to setup a ssh key-pair for grader.

Follow the steps in [Step 1](#step1) to create a new ssh key at `~/.ssh/catalog_grader`.
Don't give a password this time as ssh keys are themselves meant to be credentials so you can say that they are themselves passwords.

Copy the contents of `catalog_grader.pub` file.

On the server terminal, run -

```sh
su - grader
mkdir .ssh
chmod 700 .ssh
nano .ssh/authorized_keys
# paste the contents and save the file
chmod 644 .ssh/authorized_keys
```

Restart the ssh service.

```sh
sudo service ssh restart
```

Now you should be able to login as `grader`. Exit current connection and do the following.

```sh
ssh -p 2200 -i ~/.ssh/catalog_grader grader@165.227.16.72
```

