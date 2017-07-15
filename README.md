# Flask Postgres Apache server

> Project url: http://catalog.aavi.me/
Project IP: http://165.227.16.72/


In this tutorial, we will deploy a flask server project to a linux server (here Digital Ocean).
We will setup an internal postgres server as the database and demonstrate some advanced server management magic.

**Note** - Here we are taking a project named "[catalog](https://github.com/aviaryan/ud-catalog)".
You should replace "catalog" with *your project name* everywhere in this article.


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
15. Use a custom domain with Digital Ocean.
16. Disable root remote login and enforce key-based authentication.


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

To ssh into the server, copy the IP address from Digital Ocean interface. Here it is `165.227.16.72`.

![DO dashboard screenshot](https://i.imgur.com/QyzJMIC.png)

ssh into the server with the following command.

```sh
ssh -i ~/.ssh/catalog_root root@165.227.16.72
# ssh -i /full/path/to/key root@IP
```

You will have to enter the passphrase for the ssh key that you created. Enter it and you should be connected.

Try running `pwd` to check if things are working. Congrats, you officially now have access to your server.

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

Open ssh config in `nano`.

```sh
sudo nano /etc/ssh/sshd_config
```

Locate "Port 22" in that file. Change it to "Port 2200".

Restart ssh service.

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


## Step 9:

To configure timezone, run the following command.

```sh
sudo dpkg-reconfigure tzdata
```

Select "None of the above" and then select UTC.

![timezone configure UI](https://i.imgur.com/jMF688M.png)

When done, you should see something like this in the terminal.

```sh
Current default time zone: 'Etc/UTC'
Local time is now:      Sat Jul 15 04:50:15 UTC 2017.
Universal Time is now:  Sat Jul 15 04:50:15 UTC 2017.
```


## Step 10:

To serve Python using Apache and mod_wsgi, install the following components.

```sh
sudo apt-get install python3
sudo apt-get install python3-setuptools
sudo apt-get install apache2 libapache2-mod-wsgi-py3
# this is for Python 3
# for python2 it is python, python-setuptools and libapache2-mod-wsgi
```

Then start apache service.

```sh
sudo service apache2 restart
```


## Step 11:

Install postgresql as follows

```sh
sudo apt-get install postgresql
```

To disable remote connections, make sure you don't have any other IPs besides `127.0.0.1` in the following file.

```sh
sudo nano /etc/postgresql/VERSION/main/pg_hba.conf
# VERSION = your postgres version
# here we have
# sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```

Now to create `catalog` database, run the following to get into psql shell.

```sh
sudo -u postgres psql
```

Then when inside psql shell, run the following.

```sql
create user catalog with password 'password';
create database catalog with owner catalog;
```

Then exit psql shell with the following command.

```sql
\q
```


## Step 12:

To install git, run-

```sh
sudo apt-get install git
```


## Step 13:

Our project is at https://github.com/aviaryan/ud-catalog

First we need to clone it on server.

```sh
cd /var/www
sudo git clone https://github.com/aviaryan/ud-catalog.git catalog
```

Then we need to setup the project. See the [project README](https://github.com/aviaryan/ud-catalog) for instructions.

```sh
# install pip3
sudo easy_install3 pip
# install requirements
cd catalog
sudo pip3 install -r requirements.txt
# setup database
export DB_URI=postgresql://catalog:password@localhost/catalog
python3 manage.py db upgrade
# create initial categories
python3 create_category.py
# setup auth config
sudo nano config.py
# ^^ and add credentials there
# also change redirect_uri to http://catalog.aavi.me/gCallback
# see step 15 for why we are doing so
```


## Step 14:

Now we need to run the project using Apache and mod-wsgi. So we will first create a configuration file for your project.

```sh
sudo nano /etc/apache2/sites-available/catalog.conf
```

It should have the following components.

```xml
<VirtualHost *:80>
    ServerName 165.227.16.72

    WSGIScriptAlias / /var/www/catalog/wsgi.py

    <Directory /var/www/catalog>
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

The [wsgi file](https://github.com/aviaryan/ud-catalog/blob/master/wsgi.py) has been already added to the project. It should look like this.

```python
import sys

sys.path.insert(0, '/var/www/catalog')

from catalog import app as application

application.secret_key = 'New secret key. Change it on server'

application.config['SQLALCHEMY_DATABASE_URI'] = (
    'postgresql://'
    'catalog:password@localhost/catalog')
```

Once this is done, enable the site and restart Apache.

```sh
sudo a2ensite catalog  # enable site
sudo service apache2 reload
```

The server should be live now. Visit the IP to check (http://165.227.16.72). If an error occurs, check the logs.

```sh
sudo cat /var/log/apache2/error.log
```


## Step 15:

Now we would like to use a domain name for the server. I am using Namecheap as the domain registar but the steps should be same everywhere.

But why are we doing so? Because [Google OAuth doesn't work for bare IPs](https://stackoverflow.com/questions/14238665/).

So, go to Networking tab in Digital Ocean and create new domain. I want to use a sub-domain for this application `catalog.aavi.me`.
Add an `@` record with the target as your created server. After this the dashboard should look like the following.

![Digital Ocean Domain](https://i.imgur.com/upBFkvG.png)

Now we need to make changes in the domain registrar's side as well. Add digital ocean's nameserver records there.

![Namecheap NS](https://i.imgur.com/fQZzM9o.png)

Now go visit your sub-domain. It should work. (Here: http://catalog.aavi.me)

PS - You may need to disable the default nginx site here. `sudo a2dissite 000-default`


## Step 16:

To disable root login & password-based login through ssh, open the ssh config file.

```sh
sudo nano /etc/ssh/sshd_config
```

Make changes as shown below.

```sh
PermitRootLogin no
PasswordAuthentication no
```

Save the file and restart ssh server.

```sh
sudo service ssh restart
```


------

### References

* [Flask mod wsgi](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Disable root login](https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user)
* [Enabling and disabling sites](http://snipplr.com/view/15626/apache2--enabling-and-disabling-sites/)

