# <p style="text-align: center;">:electron: Born2beRoot</p>

```
You will create your first machine in VirtualBox (or UTM if you can’t use VirtualBox) under specific instructions. Then, at the end of this project, you will be able to set up your own operating system while implementing strict rules.
```

In this project I am using:
1. Apple M1 MacOs Ventura 13.0.1
2. UTM 4.0.9
3. Debian 11.6.0 amd64-netinst

## Virtual Machine

A Virtual Machine (VM) is a compute resource that uses software instead of a physical computer to run programs and deploy apps. Each virtual machine runs its own operating system and functions separately from the other VMs, even when they are all running on the same host. For example, you can run a virtual MacOS machine on a physical PC. 

VMs deployed to accomodate different levels of processing power needs, to run software that requires a different operating system, or to test applications in a safe, sandboxed environment. It is working through "Virtualization" technology. Virtualization uses software to simulate virtual hardware that allows 
VMs to run on a single host machine.

In this project I am using UTM since Virtial Box doesn't fully support Apple M1 chip yet. 

## Debian

Debian is a Linux-based Operating System for a wide range of devices including laptops, desktops and servers. It's easy to keep the Operating System up-to-date, whether you want to upgrade to a completely new release or just update a single package. 

In Debian-based OS distributions, the default package manager we can use is dpkg. This tool allows us to install, remove and manage programs on our operating system. However, in most cases, these programs come with a list of dependencies that must be installed for the main program to function properly. One option is to manually install these dependencies. However, APT (Advanced Package Tool), which is a tool that uses dpkg, can be used to install all the necessary dependencies when installing a program. So now we can install a useful program with a single command.

APT can work with different back-ends and fron-ends to make use of its services. One of them is apt-get, which allows us to install and remove packages. Along with apt get, there are also many tools like apt-cache to manage programs. In this case, apt get and apt-cache are used by apt. Thanks to apt we can install .deb programs easily and without worrying about dependencies. But in case we want to use a graphical interface, we will have to use aptitude. Aptitude also does better control of dependencies, allowing the user to choose between different dependencies when installing a program.


### *User Management*

#### Installing *sudo*
Switch to *root* and its environment via `su -`.
```
$ su -
Password:
#
# apt install sudo
```

### Adding User to *sudo* Group
```
# adduser <username> sudo
```
Verify whether user was successfully added to *sudo* group via `getent group sudo`.
```
$ getent group sudo
```
`reboot` for changes to take effect, then log in and verify *sudopowers* via `sudo -v`.
```
# reboot
<--->
Debian GNU/Linux 10 <hostname> tty1

<hostname> login: <username>
Password: <password>
<--->
$ sudo -v
[sudo] password for <username>: <password>
```


### Configuring *sudo*

```
$ sudo vi /etc/sudoers.d/sudoconfig
```

```
Defaults        passwd_tries=3
Defaults        badpass_message="incorrect password.."
Defaults        log_input,log_output
Defaults        iolog_dir="/var/log/sudo"
Defaults        requiretty
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
```
```
$ sudo mkdir /var/log/sudo
```
```
Defaults        logfile="/var/log/sudo/<filename>"
```

### Setting Up a Strong Password Policy

#### Password Age
```
$ sudo vi /etc/login.defs
```
set password expiration day
```
160 PASS_MAX_DAYS   99999 --> 160 PASS_MAX_DAYS   30
```
set minimum password change
```
161 PASS_MIN_DAYS   0 --> 161 PASS_MIN_DAYS   2
```
set warning message before password expiration date
```
162 PASS_WARN_AGE   7 
```

#### Password Strength
Secondly, to set up policies in relation to password strength, install the *libpam-pwquality* package.
```
$ sudo apt install libpam-pwquality
```
Configure password strength policy via `sudo vi /etc/pam.d/common-password`, specifically the below line:
```
$ sudo vi /etc/pam.d/common-password
<~~~>
25 password        requisite                       pam_pwquality.so retry=3
<~~~>
```
To set password minimum length to 10 characters, add below option to the above line.
```
minlen=10
```
To require password to contain at least an uppercase character and a numeric character:
```
ucredit=-1 dcredit=-1
```
To set a maximum of 3 consecutive identical characters:
```
maxrepeat=3
```
To reject the password if it contains `<username>` in some form:
```
reject_username
```
To set the number of changes required in the new password from the old password to 7:
```
difok=7
```
To implement the same policy on *root*:
```
enforce_for_root
```
Finally, it should look like the below:
```
password        requisite                       pam_pwquality.so retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root
```

### Creating a New User
```
$ sudo adduser <username>
```
Verify whether user was successfully created via `getent passwd <username>`.
```
$ getent passwd <username>
```
Verify newly-created user's password expiry information via `sudo chage -l <username>`.
```
$ sudo chage -l <username>
Last password change					: <last-password-change-date>
Password expires					: <last-password-change-date + PASS_MAX_DAYS>
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: <PASS_MIN_DAYS>
Maximum number of days between password change		: <PASS_MAX_DAYS>
Number of days of warning before password expires	: <PASS_WARN_AGE>
```

### Creating a New Group
Create new *user42* group.
```
$ sudo addgroup user42
$ sudo adduser <username> user42
$ getent group user42
```

### SSH
SSH or **Secure Shell** is a **remote administration protocol that allows users to control and modify their servers** over the Internet thanks to an authentication mechanism. Provides a mechanism to authenticate a user remotely, transfer data from the client to the host, and return a response to the request made by the client.<br>

### Installing & Configuring SSH
```
$ sudo apt install openssh-server
$ sudo vi /etc/ssh/sshd_config
```
To set up SSH using Port 4242, replace below line:
```
13 #Port 22
```
with:
```
13 Port 4242
```
To disable SSH login as *root* irregardless of authentication mechanism, replace below line
```
32 #PermitRootLogin prohibit-password
```
with:
```
32 PermitRootLogin no
```
Check SSH status via `sudo service ssh status`.
```
$ sudo service ssh status
```
>Alternatively, check SSH status via `systemctl status ssh`.
>```
>$ systemctl status ssh
>```
<br>

### UFW with SSH
**UFW (Uncomplicated Firewall)** is a software application responsible for ensuring that the system administrator can **manage iptables in a simple way**. Since it is very difficult to work with iptables, UFW provides us with an interface to modify the firewall of our device **(netfilter)** without compromising security. Once we have UFW installed, we can choose which ports we want to allow connections, and which ports we want to close. This will also be very useful with SSH, greatly improving all security related to communications between devices. 

### Installing & Configuring UFW

```
$ sudo apt install ufw
$ sudo ufw enable
```
Allow incoming connections using Port 4242 via `sudo ufw allow 4242`.
```
$ sudo ufw allow 4242
$ sudo ufw status
```

### Connecting to Server via SSH
```
$ ssh <username>@<ip-address> -p 4242
```
Terminate SSH session at any time via `logout`.
```
$ logout
```

### cron and wall?</a>
There are two commands that will be very helpful in case of being system administrators. These commands are:
- **Cron:** Linux task manager that allows us to execute commands at a certain time. We can automate some tasks just by telling cron what command we want to run at a specific time. 
- **Wall:** command used by the root user to send a message to all users currently connected to the server. 

### Setting Up a *cron* Job
Configure *cron* as *root*.
```
$ sudo crontab -u root -e
```
To schedule a shell script to run every 10 minutes, replace below line
```
23 # m h  dom mon dow   command
```
with:
```
23 */10 * * * * sh /path/to/script
```
Check *root*'s scheduled *cron* jobs via `sudo crontab -u root -l`.
```
$ sudo crontab -u root -l
```

### AppArmor
AppArmor provides **Mandatory Access Control (MAC) security**. In fact, **AppAmor allows the system administrator to restrict the actions that processes can perform**. If a vulnerability occurs (some of the restricted tasks are performed), AppArmor blocks the application so that the damage does not spread to the rest of the system.<br>

In this project, you have to create a simple script called `monitoring.sh` It must be developed in bash.
At server startup, the script will display some information (listed below) on all ter- minals every 10 minutes (take a look at wall). The banner is optional. No error must be visible.
Your script must always be able to display the following information:<br/>
• The architecture of your operating system and its kernel version.<br/>
• The number of physical processors.<br/>
• The number of virtual processors.<br/>
• The current available RAM on your server and its utilization rate as a percentage.<br/> 
• The current available memory on your server and its utilization rate as a percentage.<br/>
• The current utilization rate of your processors as a percentage.<br/>
• The date and time of the last reboot.<br/>
• Whether LVM is active or not.<br/>
• The number of active connections.<br/>
• The number of users using the server.<br/>
• The IPv4 address of your server and its MAC (Media Access Control) address.<br/>
• The number of commands executed with the sudo program.


## Submission & Peer Evaluation
Turn in a signature.txt file. To get this signature, you
first have to open the default installation folder (it is the folder where your VMs are
saved):
- Windows: %HOMEDRIVE%%HOMEPATH%\VirtualBox VMs\
- Linux: ~/VirtualBox VMs/
- MacM1: ~/Library/Containers/com.utmapp.UTM/Data/Documents/
- MacOS: ~/VirtualBox VMs/

Then, retrieve the signature from the ".vdi" file (or ".qcow2 for UTM’users) of your
virtual machine in sha1 format. Below are 4 command examples for a rocky_serv.vdi
file:
+ Windows: certUtil -hashfile rocky_serv.vdi sha1
+ Linux: sha1sum rocky_serv.vdi
+ For Mac M1: shasum rocky.utm/Images/disk-0.qcow2
+ MacOS: shasum rocky_serv.vdi

This is an example of what kind of output you will get:

d33a9eb73cd04c47de30070a1e6bd4688dc692a5




### <a name="What-is-LVM?">What is LVM?</a>
**LVM (Logical Volume Manager)** is an **abstraction layer between a storage device and a file system**. We get many advantages from using LVM, but the main advantage is that we have much more flexibility when it comes to managing partitions. Suppose we create four partitions on our storage disk. If for any reason we need to expand the storage of the first three partitions, we will not be able to because there is no space available next to them. In case we want to extend the last partition, we will always have the limit imposed by the disk. In other words, we will not be able to manipulate partitions in a friendly way. Thanks to LVM, all these problems are solved.<br>
By using LVM, **we can expand the storage of any partition** (now known as a logical volume) whenever we want without worrying about the contiguous space available on each logical volume. We can do this with available storage located on different physical disks (which we cannot do with traditional partitions). We can also move different logical volumes between physical devices. Of course, **services and processes will work the same way they always have**. But to understand all this, we have to know:
<ul>
 <li><b>Physical Volume (PV):</b> physical storage device. It can be a hard disk, an SD card, a floppy disk, etc. This device provides us with storage available to use.</li>
   <li><b>Volume Group (VG):</b> to use the space provided by a PV, it must be allocated in a volume group. It is like a virtual storage disk that will be used by logical volumes. VGs can grow over time by adding new VPs.</li>
   <li><b>Logical volume (LV):</b> these devices will be the ones we will use to create file systems, swaps, virtual machines, etc. If the VG is the storage disk, the LV are the partitions that are made on this disk.</li>
</ul>









## Bonus

### #1: Installation
Watch *bonus* installation walkthrough *(no audio)* [here](https://youtu.be/2w-2MX5QrQw).

### #2: Linux Lighttpd MariaDB PHP *(LLMP)* Stack

#### Step 1: Installing Lighttpd
Install *lighttpd* via `sudo apt install lighttpd`.
```
$ sudo apt install lighttpd
```
Verify whether *lighttpd* was successfully installed via `dpkg -l | grep lighttpd`.
```
$ dpkg -l | grep lighttpd
```
Allow incoming connections using Port 80 via `sudo ufw allow 80`.
```
$ sudo ufw allow 80
```

#### Step 2: Installing & Configuring MariaDB
Install *mariadb-server* via `sudo apt install mariadb-server`.
```
$ sudo apt install mariadb-server
```
Verify whether *mariadb-server* was successfully installed via `dpkg -l | grep mariadb-server`.
```
$ dpkg -l | grep mariadb-server
```
Start interactive script to remove insecure default settings via `sudo mysql_secure_installation`.
```
$ sudo mysql_secure_installation
Enter current password for root (enter for none): #Just press Enter (do not confuse database root with system root)
Set root password? [Y/n] n
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```
Log in to the MariaDB console via `sudo mariadb`.
```
$ sudo mariadb
MariaDB [(none)]>
```
Create new database via `CREATE DATABASE <database-name>;`.
```
MariaDB [(none)]> CREATE DATABASE <database-name>;
```
Create new database user and grant them full privileges on the newly-created database via `GRANT ALL ON <database-name>.* TO '<username-2>'@'localhost' IDENTIFIED BY '<password-2>' WITH GRANT OPTION;`.
```
MariaDB [(none)]> GRANT ALL ON <database-name>.* TO '<username-2>'@'localhost' IDENTIFIED BY '<password-2>' WITH GRANT OPTION;
```
Flush the privileges via `FLUSH PRIVILEGES;`.
```
MariaDB [(none)]> FLUSH PRIVILEGES;
```
Exit the MariaDB shell via `exit`.
```
MariaDB [(none)]> exit
```
Verify whether database user was successfully created by logging in to the MariaDB console via `mariadb -u <username-2> -p`.
```
$ mariadb -u <username-2> -p
Enter password: <password-2>
MariaDB [(none)]>
```
Confirm whether database user has access to the database via `SHOW DATABASES;`.
```
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| <database-name>    |
| information_schema |
+--------------------+
```
Exit the MariaDB shell via `exit`.
```
MariaDB [(none)]> exit
```

#### Step 3: Installing PHP
Install *php-cgi* & *php-mysql* via `sudo apt install php-cgi php-mysql`.
```
$ sudo apt install php-cgi php-mysql
```
Verify whether *php-cgi* & *php-mysql* was successfully installed via `dpkg -l | grep php`.
```
$ dpkg -l | grep php
```

#### Step 4: Downloading & Configuring WordPress
Install *wget* via `sudo apt install wget`.
```
$ sudo apt install wget
```
Download WordPress to `/var/www/html` via `sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html`.
```
$ sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
```
Extract downloaded content via `sudo tar -xzvf /var/www/html/latest.tar.gz`.
```
$ sudo tar -xzvf /var/www/html/latest.tar.gz
```
Remove tarball via `sudo rm /var/www/html/latest.tar.gz`.
```
$ sudo rm /var/www/html/latest.tar.gz
```
Copy content of `/var/www/html/wordpress` to `/var/www/html` via `sudo cp -r /var/www/html/wordpress/* /var/www/html`.
```
$ sudo cp -r /var/www/html/wordpress/* /var/www/html
```
Remove `/var/www/html/wordpress` via `sudo rm -rf /var/www/html/wordpress`
```
$ sudo rm -rf /var/www/html/wordpress
```
Create WordPress configuration file from its sample via `sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php`.
```
$ sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
```
Configure WordPress to reference previously-created MariaDB database & user via `sudo vi /var/www/html/wp-config.php`.
```
$ sudo vi /var/www/html/wp-config.php
```
Replace the below
```
23 define( 'DB_NAME', 'database_name_here' );^M
26 define( 'DB_USER', 'username_here' );^M
29 define( 'DB_PASSWORD', 'password_here' );^M
```
with:
```
23 define( 'DB_NAME', '<database-name>' );^M
26 define( 'DB_USER', '<username-2>' );^M
29 define( 'DB_PASSWORD', '<password-2>' );^M
```

#### Step 5: Configuring Lighttpd
Enable below modules via `sudo lighty-enable-mod fastcgi; sudo lighty-enable-mod fastcgi-php; sudo service lighttpd force-reload`.
```
$ sudo lighty-enable-mod fastcgi
$ sudo lighty-enable-mod fastcgi-php
$ sudo service lighttpd force-reload
```

### #3: File Transfer Protocol *(FTP)*

#### Step 1: Installing & Configuring FTP
Install FTP via `sudo apt install vsftpd`.
```
$ sudo apt install vsftpd
```
Verify whether *vsftpd* was successfully installed via `dpkg -l | grep vsftpd`.
```
$ dpkg -l | grep vsftpd
```
Allow incoming connections using Port 21 via `sudo ufw allow 21`.
```
$ sudo ufw allow 21
```
Configure *vsftpd* via `sudo vi /etc/vsftpd.conf`.
```
$ sudo vi /etc/vsftpd.conf
```
To enable any form of FTP write command, uncomment below line:
```
31 #write_enable=YES
```
To set root folder for FTP-connected user to `/home/<username>/ftp`, add below lines:
```
$ sudo mkdir /home/<username>/ftp
$ sudo mkdir /home/<username>/ftp/files
$ sudo chown nobody:nogroup /home/<username>/ftp
$ sudo chmod a-w /home/<username>/ftp
<~~~>
user_sub_token=$USER
local_root=/home/$USER/ftp
<~~~>
```
To prevent user from accessing files or using commands outside the directory tree, uncomment below line:
```
114 #chroot_local_user=YES
```
To whitelist FTP, add below lines:
```
$ sudo touch /etc/vsftpd.userlist
$ echo <user name> | sudo tee -a /etc/vsftpd.userlist
<~~~>
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
<~~~>
```

#### Step 2: Connecting to Server via FTP
FTP into your virtual machine via `ftp <ip-address>`.
```
$ ftp <ip-address>
```
Terminate FTP session at any time via `CTRL + D`.
