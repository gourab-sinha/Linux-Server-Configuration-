# Linux Server Configuration Project
The Linux Server Configuration Project is the last step in Udacity's Full Stack Nanodegree track. The project demands students to deploy an existing project successfully to the server via [Amazon's Lightsail](https://aws.amazon.com/lightsail) service.
The project is currently hosted at http://ec2-13-127-69-43.ap-south-1.compute.amazonaws.com/ and http://13.127.69.43/

*The URL generated here was based on the public IP provided by lightsail. The Public IP for this project is: 13.127.69.43
NOTE- To generate a URL use a service like [this](http://www.nmonitoring.com/ip-to-domain-name.html?ip=52.15.228.45&pingsub=1&ln=en) 

The instructions here can be easily adapted to deploy such projects on amazon's lightsail platform.

## Create a Lightsail Instance.
1. Visit the amazon lightsail web site provided in the link above.
2. Create a log in.
3. After logging in click on create an instance.
4. Choose OS only option and select UBUNTU
5. There is an option here to create keys here. Select it and you'll see a default key file. Download this file to your machine for later use.
6. Select a payment plan. In this instance the $5 basic plan was selected.
7. The instance will start shortly.
## Access the SSH Instance
1. On your machine create a folder for this project.
2. Make sure you have already downloaded and installed [VirtualBox](https://www.virtualbox.org/wiki/Download_Old_Builds_5_1) and [Vagrant](https://www.vagrantup.com/).
3. Open the git bash terminal and navigate to your project folder.
4. Here run `vagrant init ubuntu/trusty64` and `vagrant up`.
5. After vagrant is installed run it with `winpty vagrant ssh`.
6. Open the default key file downloaded earlier in an editor and copy its text. Paste this into a new file called lightsail.rsa
7. Move lightsail to the ubuntu system running in your vagrant machine. Move the file to /.ssh/ directory.
8. To protect this file with permissions type `chmod 600 ~/.ssh/lightsail.rsa` 
9. To connect with the SSH instance run `ssh -i ~/.ssh/lightsail.rsa ubuntu@13.127.69.43`
## Installation of updates 
To keep your work secure regular updates are neccessary. Once you're connected to SSH instance run `sudo apt-get update`. After this process type `sudo apt-get upgrade` 

## Configure the UFW(Uncomplicated Firewall)
 - To configure the firewall run `sudo /etc/ssh/sshd_config file` and look for the line port that has the value 22 in it. Change this value to 2200. Save the Changes with Ctrl+x , press Y and then Enter.
 - To check the status of firewall  type `sudo ufw status` This should return Inactive right now. To configure run the statement below in order:
				
	 `sudo ufw default deny incoming`
	 `sudo ufw default allow outgoing`
	 `sudo ufw allow ssh`
	 `sudo ufw allow 2200/tcp`
	 `sudo ufw allow www`
	 `sudo ufw allow 123/udp`
	 `sudo ufw deny 22` *This will deny the default port being run on lightsail service.
	 `sudo ufw enable` You will be asked to select y/n, press y.

To check the configuration type `sudo ufw status`. The output must look like this:

```
To                         Action      From
--                         ------      ----
22                         DENY        Anywhere
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
22 (v6)                    DENY        Anywhere (v6)
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
````
## Create New User Grader and Give Sudo Access

 1. Run `sudo adduser grader`
 2. Give the user a password. In this instance it was set to student.
 3. Fill out the other information(*this is optional)
 4. To log in as grader run `su - grader`
 5. To give grader sudo access run `sudo visudo`
 6. Look for a line that reads root ALL=(ALL:ALL) ALL
 7. Below this line type grader ALL=(ALL:ALL) ALL
 8. Save this and close visudo. Your grader show now have sudo access. To check log with `su - grader` and run `sudo -l`. After entering your password, the output must look like this:
 ```
 Matching Defaults entries for grader on
    ip-XXX-XX-X-XXX.ap-south-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bi
n\:/snap/bin

User grader may run the following commands on
        ip-XXX-XX-X-XXX.ap-south-1.compute.internal:
    (ALL : ALL) ALL
```
The X here represents the private IP

## Provide Grader SSH Login
1. Log out of your SSH instance and in your local machine run `pwd`
2. If the output is \home\vagrant this your home directory. Here run `cd .ssh`
3. In the .ssh directory create a file called grader_auth. Go back to your home directory.
4. Run `ssh-keygen` and store the result in grader_auth that was created. You will be asked for a passphrase, which in this instance was set to grader. 
5. Return to your virtual machine with `ssh -i ~/.ssh/lightsail.rsa ubuntu@13.127.69.43` and login as grader. You can choose to have your virtual machine open in a seperate terminal at this point.
6. In grader run touch `.ssh/authorized_keys`
7. On local machine run `sudo cat ~/.ssh/grader_auth.pub`. Copy the file contents and paste them into the authorized_keys file in grader using nano or vim. Make sure the content stays on a single line.
8. Inside grader run `chmod 700 .ssh`  and `chmod 644 .ssh/authorized_keys` to set permissions.
9. Run `sudo nano /etc/ssh/sshd_config` and here set PasswordAuthentication to no. Save and exit
10. Logout of the virtual machine and in your local machine type `ssh -i ~/.ssh/grader_auth -p 2200 grader@13
.127.69.43`  A popup or prompt will ask you for the passphrase. Enter that and you're now logged in as grader.
*NOTE- Some users can experience an error while logging in as grader from their local machine their message may read *access denied, (public Key)*. To avoid this you could set permission on your grader_auth file in .ssh directory with `chmod 600 grader_auth` command.

### Configure Timezone to UTC
To change timezone run `sudo dpkg-reconfigure tzdata` in grader. A set of instruction will appear for the user to follow. The UTC option can be found under **None of the above** option.

### Install Apache
1. Inside grader run `sudo apt-get install apache2`
2. To check if your apache installation was successful type in you public IP in your browser. If the default apache page renders correctly, the installation is complete.
### Install mod_wsgi
1. Run `sudo apt-get install python-setuptools libapache2-mod-wsgi`
2. Restart Apache at this point with `sudo service apache2 restart`
### Install and configure Postgresql
1. Install postgresql with `sudo apt-get install postgresql`
2. Run `sudo nano /etc/postgresql/9.5/main/pg_hba.conf` and check if your output(without comments) looks like this:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
### Python Installation
Usually python is pre installed with the updates. You could check this by running `python` in the command prompt. The output should look like this:
```
Python 2.7.12 (default, Dec  4 2017, 14:50:18)
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
### Create Postgresql User Catalog
1. With postgresql installation, the user gets a preconfigured user called postgres. First logout of your grader account and return to your virtual machine. To switch to the above mentioned user run `sudo su - postgres`
2. Run `psql` to start working with postgresql
3. Create the catalog user by running `CREATE ROLE catalog WITH LOGIN;`
4. For the user catalog to create databases run `ALTER ROLE catalog CREATEDB;`
5. To give catalog a password run `\password catalog`.
6. To view results of this process run `\du`. The output should look like this :
```
									List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalog   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```
7. Exit the psql with `\q` and logout of postgres user with Ctrl + d;
### Create User Catalog and postgresql database
1. In your virtual machine run `sudo adduser catalog`. Enter a password, which in this instance was catalog and fill out optional information.
2. To give catalog sudo access run `sudo visudo`.
3. Below grader which you gave permissions to earlier type `catalog ALL=(ALL:ALL) ALL`. Save and exit visudo.
4. Login as catalog and to check permission run `sudo -l`.
5. In catalog user run `createdb catalog` to create a database.
6. To check if the database was create run `psql` and the `l`
7. Exit from user catalog.
### Install Git and Clone Project
1. In your virtual machine run `sudo apt-get install git`
2. Cd into the directory `/var/www`. Here create a directory called cakeryCatalog. Change to this directory.
3. Here clone your github project by running `sudo git clone https://github.com/luckyrose89/Item-Catalog-Application.git cakeryCatalog`
4. At this point change the ownership of the cakery catalog to ubuntu by running `sudo chown -R ubuntu:ubuntu cakeryCatalog/`.
5. Cd into /var/www/cakeryCatalog/cakeryCatalog and change the name of your main python file by running `mv yourfilename.py __init__.py`
6. Run nano on init or vim and edit the line `app.run(host='0.0.0.0', port=8000)` and just write `app.run()`
7. Inside database_setup.py replace the lines in following manner
```
#engine = create_engine("sqlite:///catalog.db")
engine=create_engine('postgresql://catalog:YOUR_CATALOG_PASSWORD@localhost/catalog')
```
Make this correction in all the files where this line occurs.
