<h1 align="center">LAMP Stack Server</h1>

**Overview:** The goal of this project is to configure and run an Apache HTTP server in a Linux OS Server, set up Git for version control, create a Relational Database, run containerized Node.js applications with Docker, use Kubernetes as container orchestration, and create CI/CD pipelines and Python automation.

I will be developing from scratch a customized LAMP stack first locally and then eventually connecting the entire application to the internet and will be documenting all my learning paths with the steps, errors, and cool solutions that I have found.

# Table Of Contents

1. [Finding a Machine](#finding-a-machine)
2. [Linux Ubuntu Server 22.04 LTS](#linux-ubuntu-server-2204-lts)
    - [Installation Steps](#installation-steps)
    - [Boot Problem](#boot-problem)
    - [Setting up OpenSSH connection](#setting-up-openssh-connection)
    - [Users and Groups Setup](#users-and-groups-setup)
3. [Apache](#apache)
    - [Apache Installation Steps](#apache-installation-steps)
    - [Apache Management/Configuration](#apache-managementconfiguration)
    - [Apache Commands](#apache-commands)
4. [Docker](#docker)
 - [Docker Installation Steps](#docker-installation-steps)
5. [Stackstorm installation](#stackstorm-installation)
    - [Setting up Stackstorm Web UI](#setting-up-stackstorm-web-ui)
6. [Configuring CI/CD with Jenkins and Github](#configuring-cicd-with-jenkins-and-github)
 - [Starting and running Jenkins container](#starting-and-running-jenkins-container)
 - [Configuring Github](#configuring-github)
7. [Additional Tools and Packages](#additional-tools-and-packages)
    - [Hardware Lister](#hardware-lister)
8. [Technologies Used](#technologies-used)
9.  [Languages Used](#languages-used)
10.  [Frameworks, Libraries & Programs Used](#frameworks-libraries-and-programs-used)
11.  [Testing](#testing)
    - [](#)
12. [Deploy](#deploy)
13. [Credits](#credits)
    - [](#)


# Finding a Machine

My first thought regarding which machine to use to start the server was about buying a Rasperry Pi, but due to the high price and no stock availability on the sites I have searched I decided not to get one. Then I remembered about an old HP Probook Laptop that was sitting in the closet gathering dust, and the configuration was not bad at all to start this project:

```
HP ProBook 4540 
Intel(R) Core(TM) i3-3110M CPU @ 2.40GHz
4GiB SODIMM DDR3 Memory RAM
500GB Hitachi HTS72755 (HDD)
16GB Cruzer Fit (USB drive)
```

My recommendation: With the chip shortage and the high demand for Raspberry Pi's the prices are crazy. Instead you can use that old abandoned PC gathering dust for free!

# Linux Ubuntu Server 22.04 LTS

## Installation Steps:

1. Downloaded ISO file for Ubuntu Server 22.04 LTS from https://ubuntu.com/download/server while choosing the "Option 2 - Manual Server Installation".
2. Created a Bootable USB drive with Rufus (https://rufus.ie/en/) and with the Ubuntu Server ISO file.
3. Plugged the USB drive and started the machine while selecting to boot from the USB pen drive.
4. With the machine booted from the USB I've opened the Linux Shell and started checking the partitions with their file system information with `lsblk -f` and the output showed the HDD device name was 'sda2' and was formatted with the NTFS file system.
5. After confirming the device name I started formatting to the correct ext4 file system by running the command `sudo mkfs -t ext4 /dev/sda2`
6. Ran `lsblk -f` again just to make sure the format was successfully and that the HDD was formatted with the ext4 file system.
7. Proceeded with the Ubuntu Server 22.04 LTS installation in the HDD. I didn't select to install any snap, and only installed the OpenSSH server package.

## Boot Problem:

After smoothly performing the installation steps the Linux server completed the installation successfully and I rebooted the machine without the USB drive attached, then the machine, booting from the HDD, showed the first error message:

![Error Message - BootDevice Not Found](images/boot_error.jpg)

No panic, finally a bit of troubleshooting! Well, the troubleshooting definitely went longer than I thought and I run through all the troubleshooting steps below:

1. Performed a Hard Reset ✓
2. Re-formatted the HDD and re-installed Linux ✓
3. Restored BIOS to Default Settings ✓
4. Changed and tested several different BIOS settings (including changing the Boot Mode: Legacy, UEFI Hybrid and UEFI Native) ✓
5. Ran some HDD tests and Disk Sanitize to make sure the HDD was fine ✓
6. Manually selected the HDD to be booted from and updated the System boot priority ✓

Temporary workaround:

After hours of troubleshooting and with no resolution after performing the steps above I decided it was time to let this go for now and tried a different approach: I've found a 16GB USB Drive and on my first attempt to install the Linux Server on the USB drive it worked!

So this confirmed the issue is with the HDD, I still haven't found the exact cause of the issue, but I will! I have some next steps that I will be working on to try to figure this out at a later stage:

Some next steps I have in mind to find the Boot issue with the HDD:

1. Check if HDD has a partition or a section that is corrupted.
2. Check if there is an issue with the MBR.
3. Check for any SATA connections issues with the HDD and the machine.

## Setting up OpenSSH connection:
The OpenSSH came pre-installed (It shows an option to install OpenSSH while Linux Server is being installed.

Full documentation on the OpenSSH tool: https://www.openssh.com/

Setup steps:

After the server was booted up and I had logged in with my user and password:
1. To find out my server private IP address: `ip a`.
2. Ran `sudo lsof -P | grep LISTEN` to check for all the open ports and if the port 22 was opened.
3. Check if the OpenSSH is active and running with: `sudo service ssh status`
4. To connect to the local server: `ssh<USER>@<IP_ADDRESS>`

OpenSSH also allows connecting via SFTP to make it easier for file uploads to the server. 

### Setting up SSH Public and Private Key
To connect to my remote server without having to type a password everytime I have created a ssh key following the steps below:

1. On my local computer I ran `ssh-keygen -t rsa`
2. Still on my local computer I ran `ssh-copy-id <USER>@<SERVER_IP_ADDRESS>`

### Setting up server name on /etc/hosts
To avoid having to always use the server IP address to connect I have added a hostname to the `/etc/hosts` file:

<SERVER_IP_ADDRESS> <my_hostname>

## Users and Groups setup

Currently I am having one small issue while trying to upload a test image to the images/ folder. I am getting a permissions denied message.

The images/ folder has the following permissions: **drwxrwxr-x set as 0755** and I get the permissions denied when I try to upload an image via SFTP to that folder. But when I change the file permissions with `chmod 777 images/` to **drwxrwxrwx 0777** (not ideal but just for testing purposes) I am able to upload the image, so while connected via SFTP I am connected as a Public user. I will need to create a specific group for my username to allow these SFTP modifications.

To change the Files and Directories permissions to 777 (rwxrwxrwx) for testing I have used the chmod command:

`chmod 777 images/`

**Numeric Notation:**

```
r - read  - 4
w - write  - 2
x - execute  - 1
```

Also while trying to create/delete any file/directory I am getting a permission denied message.

### Fix:

To fix the file permissions error I updated the file User and Group owner to my own user and then I was able to create and delete anything without having to use sudo access. The Directory and the File permissions remained the same:

**Directory Permissions:** 755
**File Permissions:** 644

**/var/www/html contents before the fix:**

```
-rw-r--r-- 1 root root 10682 Jun 10 12:09 apache_default_index.html
drwxr-xr-x 4 root root  4096 Jun 10 12:51 assets/
-rw-r--r-- 1 root root  8648 Jun 10 12:23 index.html
-rw-r--r-- 1 root root 10545 Jun 10 12:55 why-donate.html
```

**/var/www/html contents after the fix:**

```
-rw-r--r-- 1 mateus mateus 10682 Jun 10 12:09 apache_default_index.html
drwxr-xr-x 4 mateus mateus  4096 Jun 10 12:51 assets/
-rw-r--r-- 1 mateus mateus  8648 Jun 10 12:23 index.html
-rw-r--r-- 1 mateus mateus 10545 Jun 10 12:55 why-donate.html
```

To change the Owner of the file: `chown <USER> file_name`

To change the Group of the file: `chgrp <USER> file_name`

In this case I ran these commands with a `*` instead of the `file_name` to change all the contents in the current directory I was in.

PS: While adding the username in the documentation I have covered my real username and used my_user just to be extra safe.

### Creating User

To stop using root everytime I was connected to the server I create a username by running `sudo adduser mateus`

And then to add my user to the sudo group I ran `usermod -aG sudo mateus`

And we can check that was succesfully by running `groups mateus`:

```
/home# groups mateus
mateus : mateus sudo
```

# Apache

## Apache Installation Steps

Full apache documentation here: https://apache.org/

1. Updated my repositories with `sudo apt-get update`
2. Installed apache2 running `sudo apt-get install apache2`
3. Ran `sudo lsof -i -P -n | grep LISTEN` to check if port 80 is open:

```
sshd       805            root    3u  IPv4  24422      0t0  TCP *:22 (LISTEN)
sshd       805            root    4u  IPv6  24424      0t0  TCP *:22 (LISTEN)
apache2    865            root    4u  IPv6  26387      0t0  TCP *:80 (LISTEN)
apache2   4615        www-data    4u  IPv6  26387      0t0  TCP *:80 (LISTEN)
apache2   4616        www-data    4u  IPv6  26387      0t0  TCP *:80 (LISTEN)
```
4. Opened a web browser and connected to my local server IP 192.168.1.x to confirm I could see the default Apache2 page:
    ![Apache web server](images/apache2_index.jpg)

5. That's it, now I have an Apache server running on Linux!

## Apache Management/Configuration

After Apache was installed I have added some custom HTML/CSS files that I have made to load a simple website:

![ONE%](images/one_p.jpg)

**Access Logs:**
To check for the apache Access logs we can ran `cat /var/log/apache2/access.log` for todays logs and `cat /var/log/apache2/access.log.1` or `zcat /var/log/apache2/access.log.2.gz` to check for past logs.

**Error Logs:**
To check for the apache Error logs we can ran `cat /var/log/apache2/error.log` for todays logs and `cat /var/log/apache2/error.log.1` or `zcat /var/log/apache2/error.log.2.gz` to check for past logs.

## Apache Commands

**To stop apache2:**
`sudo /etc/init.d/apache2 stop`

**To start apache2:**
`sudo /etc/init.d/apache2 start`

**To Restart apache2:**
`sudo /etc/init.d/apache2 restart`

# Docker

Docker container docs: https://docs.docker.com/engine/reference/commandline/container/

At this stage, and after getting more familiarized with Apache I started leaning towards Docker Containers and containerizing my applications going forward, so instead of running Apache and other applications directly on the server I installed docker container to run everything in containers. I also moved my local server to a remote one to make it easier to connect from anywhere while having a Public IP Address.

## Docker Installation Steps

Chose the convenience scripts installation method as this is just a personal project, but this installation method is only recommended for testing and development environments.

1. Ran sudo `sudo apt-get update`
2. `curl -fsSL https://get.docker.com -o get-docker.sh`
3. `sudo sh ./get-docker.sh`
4. Checked current docker version that was installed with `docker -v`:
```
Docker version 23.0.2, build 569dd73
```

To update docker: `sudo apt-get install --only-upgrade docker`

Installing docker compose: `apt install docker-compose`

# Stackstorm installation

1. Made sure I had the update version of Docker engine and docker-compose.
2. Cloned the st2-docker repo with `git clone https://github.com/stackstorm/st2-docker`
3. cd into the newly cloned dir and ran `docker-compose up -d`
4. Run `sudo docker ps` to show all the containers up and running.

## Setting up Stackstorm Web UI

As I am running stackstorm on a remote host inside a docker container I had to update the `docker-compose.yml` file and edit the line with the local ip address `127.0.0.1:80` below to match with the remote host IP address:

```
    ports:
      - "${ST2_EXPOSE_HTTP:-127.0.0.1:80}:80"
      # - "${ST2_EXPOSE_HTTPS:-127.0.0.1:443}:443"
      # more work would be needed with certificate generate to make https work.
```
I also updated the `config.js` file located on `/opt/stackstorm/static/webui/config.js` inside the docker container `stackstorm/st2web:latest`. To do that I had to connect to the st2web container with `docker exec -it <CONTAINER_ID> /bin/sh` and then edit the file to update it:

```
hosts: [{
  name: 'Express Deployment',
  url: 'http://<REMOTE_HOST>/api',
  auth: 'http://<REMOTE_HOST>/auth'
},{
  name: 'Development Environment',
  url: 'http://<REMOTE_HOST>:9101'
  auth: 'https://<REMOTE_HOST>:9100'
}]
```

After the file was updated I ran `docker-compose up -d` and was able to load ST2 web UI:


# Configuring CI/CD with Jenkins and Github

## Starting and running Jenkins container

I choose to install Jenkins in order to create a CI/CD Pipeline With Jenkins and GitHub. So everytime I made changes, commit and push any changes to my Github repo this should trigger a Jenkins job and push the changes to the server automatically.

Instead of storing all Jenkins data inside the container I choose to store the data on the host, doing this it should prevent the jenkins data to erase everytime the jenkins container is stopped and a new container is created.

1. Created a directory to store jenkins data with `mkdir jenkins-data`
2. In order to jenkins to have write permission to directory we also need to update the directory owner to jenkins user - uid 1000 with `chown 1000 jenkins-data`
3. Started jenkins container while setting the port mapping to match the port from the host with the container and also storing jenkins data on the directory I have created in the step above: `docker run -p 8080:8080 -v /root/jenkins-data:/var/jenkins_home jenkins/jenkins`

After jenkins image finished pulling and the jenkins container started running from that image I was able to access jenkins via port 8080, after a time waiting for jenkins to finish the setup I was able to view jenkins dashboard:

![jenkins_dashboard](images/jenkins_dashboard.jpg)

## Configuring Github

1. Clicked on the Settings of the LAMP Stack Server repo.
2. Created a webhook and updated the Payload URL to match the remote server IP with the port 8080.
3. Changed the Content type to "application/json"
4. Select just the PUSH event, toggled the "Active" option and clicked on "Add webhook"
5. With this configuration all my git pushes are being deployed via Jenkins to the server under `/root/jenkins-data/workspace/LAMP Stack Server/`

![jenkins_dashboard2](images/jenkins_dashboard2.jpg)
![jenkins_dashboard3](images/jenkins_dashboard3.jpg)

# Additional Tools and Packages

  ## Hardware Lister
  
  It is a tool that provides detailed information on the hardware configuration of the machine.
  
  Site and Documentation: https://ezix.org/project/wiki/HardwareLiSter
  
  **To install:**

  `sudo apt-get install lshw`
  
  **To check the hardware information:**

  `sudo lshw -short`

----

  ## htop

  htop is a colorful interactive process viewer! It shows information just like the `top` command but with more options and it's easier to view all the processes and what might be consuming too much RAM or CPU usage, it also has great additional options like Searching, Filtering and SortBy.

  Site and Documentation: https://htop.dev/

  **To install:**

  `sudo apt install htop`

  **To run:**

  Simply run `htop` to open the process viewer:
    ![htop](images/htop_process_viewer.jpg)
  To exit the process viewer you can hit F10 or CTRL+C.
