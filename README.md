# Born2beRoot

Here I created my first machine in VirtualBox under specific instructions.

### Table of Contents
* [Installation](#installation)
* [sudo](#sudo)
* [SSH](#ssh)
* [Password Policy](#user-management)
* [cron](#cron)
* [Lighttpd & MariaDB & PHP](#lighttpd)
* [FTP](#ftp)
* [Useful Commands](#commands)

<a name="installation"/>

## Installation

The latest stable version of Debian is Debian 11. You can download it via this [link](https://www.debian.org/download).

<a name="sudo"/>

## sudo

### 1. Installing sudo
Switch to root environment:
```
$>su
$>Password:
```
Install *sudo*:
```
$>apt install sudo
```
Verify if installation was successful:
```
$>dpkg -l | grep sudo
```

### 2. Adding User to *sudo* Group
Add user to *sudo* group:
```
$>adduser <username> sudo
```
Verify if installation was successful:
```
$>getent group sudo
```
`reboot`, then log in again and verify `sudopowers` via `sudo -v`. From now on, you can run root-privileged commands via prefix `sudo`.

### 3. Configuring sudo
Create a log directory via `sudo mkdir /var/log/sudo` and editing the **sudoers.tmp** file via `$>sudo visudo`:
* `Defaults passwd_tries=3` - sets number of password attempts to 3.
* `Defaults badpass_message="Wrong password"` - sets error message.
* `Defaults logfile="/var/log/sudo/sudo.log"` - creates and sets log file.
* `Defaults log_input,log_output` - creates logs of inputs and outputs when using `sudo`.
* `Defaults iolog_dir="/var/log/sudo"` - the directory where logs are stored.
* `Defaults requiretty"` - enables TTY mode.
* `Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"` - resctricts path that can be used by `sudo`.

<a name="ssh"/>

## SSH

### 1. Configuring SSH
Install the OpenSSH Service to use the SSH network protocol:
```
$>sudo apt install openssh-server
```
Verify if installation was successful:
```
$>dpkg -l | grep ssh
```
Configure SSH via `sudo vi /etc/ssh/sshd_config`:
| Replace | with | to |
| :---: | :---: | :---: |
| `#Port 22` (line 15) | `Port 4242` | set up SSH using Port 4242. |
| `#PermitRootLogin prohibit-password` (line 34) | `line34 PermitRootLogin no` |  disable SSH login as root. |

Check SSH status:
```
$>sudo service ssh status
```

### 2. Configuring UFW
Install UFW:
```
$>sudo apt install ufw
```
Verify if installation was successful:
```
$>dpkg -l | grep ufw
```
Enable Firewall and allow connections through Port 4242:
```
$>sudo ufw enable
$>sudo ufw allow 4242
```
Check UFW status:
```
$>sudo ufw status
```

### 3. Connecting to Server via SSH
Open your terminal and SSH into vm:
```
$>ssh <username>@<ip-address> -p 4242
```
To terminate SSH session either use `logout` or `exit`.

<a name="user-management"/>

## Password Policy

Configure password age via `$>sudo vi /etc/login.defs`:
* `PASS_MAX_DAYS 30` - makes passwords expire after 30 days.
* `PASS_MIN_DAYS 2` - sets minimum number of days allowed before password modification to 2.
* `PASS_WARN_AGE 7` - displays users warning message 7 days before their passwords expire.

Install the *libpam-pwquality* package, which provides common functions for password quality checking:
```
$>sudo apt install libpam-pwquality
```
Verify if installation was successful:
```
$>dpkg -l | grep libpam-pwquality
```
Configure password strength policy via `sudo vi /etc/pam.d/common-password`:
* `minlen=10` - password must be at least 10 characters long.
* `ucredit=-1` - password must contain an uppercase letter.
* `dcredit=-1` - password must contain a number.
* `maxrepeat=3` - password must not contain more than 3 consecutive identical characters.
* `reject_username` - password must not include the name of the user.
* `difok=7` - does not apply to the root password: password must have at least 7 characters that are not part of the former password.
* `enforce_for_root` - root password has to comply with this policy.

Resulting line: `password requisite pam_pwqiality.so difok=7 enforce_for_root retry=3 minlen=10 ucredit=-1 dcredit=-1 maxrepeat=3 reject_username`

<a name="cron"/>

## cron

A crontab file contains a list of commands meant to be run at specified times. For this purpose I created a [monitoring.sh](https://github.com/zangelis/Born2beRoot/blob/master/monitoring.sh) file under `/usr/local/bin/` and configured *cron* via `sudo crontab -u root -e`. To display some information defined in the script on all ter- minals every 10 minutes, replace line 23 with following line:
```
*/10 * * * * bash /path/to/script | wall
```
Last check your cron jobs via `sudo crontab -u root -l`.

<a name="lighttpd"/>

## Lighttpd & MariaDB & PHP

### 1. Installing Lighttpd
Install Lighttpd:
```
&>sudo apt install lighttpd
```
Verify if installation was successful:
```
$>dpkg -l | grep lighttpd
```
Allow connections through Port 80:
```
$>sudo ufw allow 80
```

### 2. Configuring MariaDB
Install MariaDB:
```
$>sudo apt install mariadb-server
```
Verify if installation was successful:
```
$>dpkg -l | grep mariadb-server
```
Remove insecure default settings via `sudo mysql_secure_installation`:
```
$>sudo mysql_secure_installation
Enter current password for root (enter for none): #Just press Enter (do not confuse database root with system root)
Set root password? [Y/n] n
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```
Log in to MariaDB console via `sudo mariadb` and follow [this great tutorial](https://www.daniloaz.com/en/how-to-create-a-user-in-mysql-mariadb-and-grant-permissions-on-a-specific-database/).

Create a new database:
```
MariaDB [(none)]> CREATE DATABASE <database-name>;
```
Create a new database user:
```
MariaDB [(none)]> CREATE USER '<db-username>'@localhost IDENTIFIED BY '<password>';;
```
Grant user full privileges on the newly-created database:
```
MariaDB [(none)]> GRANT ALL privileges ON <database-name>.* TO '<db-username>'@localhost;
```
Flush the privileges:
```
MariaDB [(none)]> FLUSH PRIVILEGES;
```
Exit the MariaDB shell:
```
MariaDB [(none)]> exit
```
Verify if database user was successfully created by logging in to the MariaDB:
```
$ mariadb -u <db-username> -p
Enter password: <password>
MariaDB [(none)]>
```
Confirm if database user has access to the database:
```
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| <database-name>    |
| information_schema |
+--------------------+
```
Exit the MariaDB shell:
```
MariaDB [(none)]> exit
```

### 3. Installing PHP
Install php-cgi & php-mysql:
```
$>sudo apt install php-cgi php-mysql
```
Verify if installation was successful:
```
$>dpkg -l | grep php
```

### 4. Configuring WordPress
Install *wget*:
```
$>sudo apt install wget
```
Download WordPress to `/var/www/html`, extract downloaded content and remove tarball:
```
$>sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html
$>sudo tar -xzvf /var/www/html/latest.tar.gz
$>sudo rm /var/www/html/latest.tar.gz
```
Copy content of `/var/www/html/wordpress` to `/var/www/html` and remove empty directory:
```
$>sudo cp -r /var/www/html/wordpress/* /var/www/html
$>sudo rm -rf /var/www/html/wordpress
```
Create WordPress configuration file via `sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php` and configure WP to reference MariaDB database & user via `$>sudo vi /var/www/html/wp-config.php`:
```
define( 'DB_NAME', '<database-name>' );
define( 'DB_USER', '<db-username>' );
define( 'DB_PASSWORD', '<password>' );
```

### 5. Configuring Lighttpd
Last but not least enable following modules:
```
$>sudo lighty-enable-mod fastcgi
$>sudo lighty-enable-mod fastcgi-php
$>sudo service lighttpd force-reload
```

<a name="ftp"/>

## FTP

Install FTP server:
```
$>sudo apt install vsftpd
```
Verify if installation was successful:
```
$>dpkg -l | grep vsftpd
```
Install FTP client:
```
$>sudo apt install ftp
```
Verify if installation was successful:
```
$>dpkg -l | grep ftp
```
Allow connections through Port 21:
```
$>sudo ufw allow 21
```
Configure vsftpd via `sudo vi /etc/vsftpd.conf` and uncomment line 31 to enable any form of FTP write command:
```
#write_enable=YES
```
Set root folder for FTP-connected user to `/home/<username>/ftp`:
```
$>sudo mkdir /home/<username>/ftp
$>sudo mkdir /home/<username>/ftp/files
$>sudo chown nobody:nogroup /home/<username>/ftp
$>sudo chmod a-w /home/<username>/ftp
```
Add following to the top of vsftpd.conf via `sudo vi /etc/vsftpd.conf`:
```
user_sub_token=$USER
local_root=/home/$USER/ftp
```
Uncomment line 114 to prevent user from accessing files or using commands outside the directory tree:
```
#chroot_local_user=YES
```
Whitelist FTP:
```
$>sudo vi /etc/vsftpd.userlist
```
Save and leave the file and add username via `echo <username> | sudo tee -a /etc/vsftpd.userlist`. Then add following lines via `sudo vi /etc/vsftpd.userlist`:
```
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
```
FTP into your vm:
```
$>ftp <ip-address>
```

<a name="commands"/>

## Useful Commands

### Setup
* `chage -l <username>` - Checks if passwords rules are working on the user.
* `sudo ufw status` - Checks the ufw status.
* `sudo service ssh status` - Checks the status of the ssh.
* `uname -a` - Checks the operating system.
* `head -n 2 /etc/os-release` - Checks the operating system.

### User
* `getent group` - Checks the groups created.
* `getent group <groupname>` - Checks the users under a group.
* `sudo adduser <username>` - Creates a new user.
* `sudo /etc/pam.d/common-password` - to set the length of the password, the characters, etc.
* `sudo nano /etc/login.defs` - To set password expiration date, etc.
* `sudo addgroup <groupname>` - Creates a new group.
* `sudo adduser <username> <groupname>` - Adds a user to a group.

### Hostname and Partitions
* `hostname` - Checks current hostname.
* `sudo hostnamectl set-hostname <newhostname>` - Changes the hostname.
* `sudo nano /etc/hosts` - Changes the old hostname for the new one.
* `lsblk` - See the partitions.

### Sudo
* `command -v sudo` - Checks, if sudo is installed.
* `sudo adduser <username> sudo` - Adds the user to the sudo group.
* `sudo visudo` - Access sudoers file to edit the configuration of sudo.

### UFW
* `command -v ufw` -  Checks, if ufw is installed.
* `sudo ufw status` - Checks the ufw status.
* `sudo ufw allow <port_number>` - Allows the port indicated.
* `sudo ufw status numbered` - Checks the ufw status with numbers on the left of each rule.
* `sudo ufw delete <number>` - Deletes the rule that corresponds to that number.

### SSH
* `command -v ssh` - Checks, if ssh is installed.
* `sudo service ssh status` - Checks the status of the ssh.
* `sudo nano /etc/ssh/ssh_config` - Checks the port used by ssh.
* `ip a` - Checks the IP to use for connect from the host terminal.
* `ssh <username>@<ip> -p 4242` - (In the host terminal) to connect to the VM using ssh.

### monitoring.sh
* `sudo crontab -e` - Edit the crontab file.
* `sudo nano /usr/local/bin/monitoring.sh` - Edit the monitoring.sh.

### Etc.
* `cut -d: -f1 /etc/passwd` -  Checks all the users in the machine.
* `groups` - Checks which groups my user belongs to.
* `/usr/sbin/aa-status` - Checks the status of AppArmor.
* `ss -tulp` - Shows the net status, ports etc.
