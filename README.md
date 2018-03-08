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
9. To connect with the SSH instance run `ssh -i ~/.ssh/lightrail_key.rsa ubuntu@13.127.69.43`
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


