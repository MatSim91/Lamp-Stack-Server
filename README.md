<h1 align="center">LAMP Stack Server</h1>

**Overview:** The goal of this project is to configure and run an Apache HTTP server in a Linux OS Server, set up Git for version control, create a Relational Database, run containerized Node.js applications with Docker, use Kubernetes as container orchestration, and create CI/CD pipelines and Python automation.

I will be developing from scratch a customized LAMP stack first locally and then eventually connecting the entire application to the internet and will be documenting all my learning paths with the steps, errors, and cool solutions that I have found.

# Table Of Contents

1. [Finding a Machine](#finding-a-machine)
2. [Linux Ubuntu Server 22.04 LTS Installation](#linux-ubuntu-server-2204-lts-installation)
    - [Installation Steps Performed](#installation-steps-performed)
    - [Boot Problem](#boot-problem)
    - [Users and Groups Setup](#users-and-groups-setup)
3. [Apache Installation](#apache-installation)
4. [Additional Tools and Packages](#additional-tools-and-packages)
    - [Hardware Lister](#hardware-lister)
5. [Technologies Used](#technologies-used)
6.  [Languages Used](#languages-used)
7.  [Frameworks, Libraries & Programs Used](#frameworks-libraries-and-programs-used)
8.  [Testing](#testing)
    - [](#)
9. [Deploy](#deploy)
10. [Credits](#credits)
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

# Linux Ubuntu Server 22.04 LTS Installation

## Installation Steps Performed:

1. Downloaded ISO file for Ubuntu Server 22.04 LTS from https://ubuntu.com/download/server while choosing the "Option 2 - Manual Server Installation".
2. Created a Bootable USB drive with Rufus (https://rufus.ie/en/) and with the Ubuntu Server ISO file.
3. Plugged the USB drive and started the machine while selecting to boot from the USB pen drive.
4. With the machine booted from the USB I've opened the Linux Shell and started checking the partitions with their file system information with `lsblk -f` and the output showed the HDD device name was 'sda2' and was formatted with the NTFS file system.
5. After confirming the device name I started formatting to the correct ext4 file system by running the command `sudo mkfs -t ext4 /dev/sda2`
6. Ran `lsblk -f` again just to make sure the format was successfully and that the HDD was formatted with the ext4 file system.
7. Proceeded with the Ubuntu Server 22.04 LTS installation in the HDD. I didn't select to install any snap, and only installed the OpenSSH server package.

## Boot Problem:

After smoothly performing the installation steps the Linux server completed the installation successfully and I rebooted the machine without the USB drive attached, then the machine, booting from the HDD, showed the first error message:

![Error Message - BootDevice Not Found]()

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
-rw-r--r-- 1 my_user my_user 10682 Jun 10 12:09 apache_default_index.html
drwxr-xr-x 4 my_user my_user  4096 Jun 10 12:51 assets/
-rw-r--r-- 1 my_user my_user  8648 Jun 10 12:23 index.html
-rw-r--r-- 1 my_user my_user 10545 Jun 10 12:55 why-donate.html
```

To change the Owner of the file: `chown my_user file_name`

To change the Group of the file: `chgrp my_user file_name`

In this case I ran these commands with a `*` instead of the `file_name` to change all the contents in the current directory I was in.

PS: While adding the username in the documentation I have covered my real username and used my_user just to be extra safe.

# Apache Installation



# Additional Tools and Packages

  ## Hardware Lister
  
  It is a tool that provides detailed information on the hardware configuration of the machine.
  
  Site and Documentation: https://ezix.org/project/wiki/HardwareLiSter
  
  **To install:**

  `sudo apt-get install lshw`
  
  **To check the hardware information:**

  `sudo lshw -short`
