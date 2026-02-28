# Install VSFTPD

> *Last Updated: Feb, 28, 2026 - Saturday*
>
>*Created by: Christian*

**Note:** Assume that your Linux is already installed and ready to install from the start.

**Note:** This is for personal preference, guide and shows some of examples, you can create your own structure and setup for your server.

## 1. Getting Started: Install VSFTPD

Go to your terminal by pressing **CTRL + ALT + T** and type the following commands:

```bash
# Important to update the apt first
sudo apt update

# Then install the VSFTPD Server
sudo apt install vsftpd

# Enable to vsftpd to make sure that it automatically start up when the Linux boots up
sudo systemctl enable vsftpd

# Start the vsftpd server
sudo systemctl start vsftpd

# Check if the vsftpd is running
systemctl status vsftpd
```

## 2. Configure the Firewall (This may be optional)

In case that you didn't have install Firewall in your Linux yet, go to **Terminal** and enter the following commands:

```bash
# Install UFW
sudo apt install ufw

# Check if ufw is enabled
sudo systemctl is-enabled ufw

# Enable the ufw to make sure that it automatically start up when the Linux boots up
sudo systemctl enable ufw

# Start the ufw
sudo systemctl start ufw

# Check if the ufw is running
systemctl status ufw
```

### 2.1. Allow ports

```bash
# For this Example, we will use these ports, but you can allow any ports
sudo ufw allow 20/tcp
sudo ufw allow 21/tcp
sudo ufw allow 22/tcp
sudo ufw allow 10000:10100/tcp
sudo ufw allow OpenSSH

# Then check the status if enabled
sudo ufw status

# In case that the firewall isn't enabled yet
sudo ufw enable
```

## 3. Configure the VSFTPD Config

To locate the `config` of `vsftpd`, enter this command:

```bash
sudo nano /etc/vsftpd.conf
```

Then press **Enter.**

`nano` - This is the name of a simple, easy-to-use command-line text editor. It is a popular alternative to more complex editors like Vim or Emacs because it displays common shortcuts on the screen and is more intuitive for beginners.

You will go to the configuation of `vsftpd` some of the codes required input it manually and some of codes are already there just remove the comment `#`.

Here are the following line of codes that needed for this setup:

```bash
listen=NO

listen_ipv6=YES

# This prevents anonymous accessing FTP
anomymous_enable=NO

# Uncomment to allow local users to Log in
local_enable=YES

# Uncomment to enable any form of FTP write Command
write_enable=YES

# Enable chroot_list by uncomment this
chroot_local_user=YES

# Then add this to enable write chroot
allow_writeable_chroot=YES

# Direct the FTP User(s) to the main root of the server
local_root=/srv/ftp

# Passive mode, set the minimum and maximum range of ports
pasv_enable=YES
pasv_min_port=10000
pasv_max_port=10100

# Every file instered will automatically set the permission to 640
file_open_mode=0640
local_umask=027
```

After finish writing the codes, press **CTRL + O then Enter** to save the configuation and press **CTRL + W** to exit the config and back to your terminal.

After back, restart the `vsftpd` server.

```bash
sudo systemctl restart vsftpd
```

Then try to access the FTP (File Transfer Protocol) Server by getting the **IP Address** of the Linux, this can get through this command.

```bash
ip a
```

In accessing FTP Server, make sure that:
- Same having a network connection
- Same Gateway

## 4. Creating Folder structure

This is an example on how you create your folder inside your FTP server.

First much better if you go inside the FTP server folder already to save code lines.

```bash
# CD - Change Directory 
cd /srv
```

Let's say I make a folder for `Admin`, here's how to proceed:

```bash
# Create a folder
sudo mkdir Admin

# OR, In case you aren't inside your /srv directory
sudo mkdir /srv/ftp/Admin

# Note: mkdir - Make Directory
```

Then if you want to remove the folder

```bash
# Remove a single file
sudo rm sample.txt

# Remove the directory along with its contents and subdirectories
sudo rm -r Admin

# Notes:
# rm - Remove
# -r - Recursive
```

Use this as your reference to build your own folder structure.

## 5. Adding Users for the Server

### 5.1. Creating Users

To create a user for your FTP Server, take this example and do the following command:

```bash
sudo adduser --home /srv/ftp/Admin ftp_admin

# adduser - Adds the user
# --home - Sets as their home folder
# /srv/ftp/Admin - their path
# ftp_admin - Name of the User Itself
```

Then press **Enter.** Ensure that you created already the folder and that folder exists.

At this point you required to create a password for that user, after creating a password, enter some **Full Name** but the next information will optional.

Then after creating information you'll confirm it and you'll good to go. Now you keep repeat the process until you created all the users you want.

### 5.2. Assign the Ownership of the folder to the user itself

In this example, we set the `ftp_admin` to be the owner of `/srv/ftp/Admin` path.

```bash
sudo chown ftp_admin:ftp_admin /srv/ftp/Admin

# chown = Change Ownership
```

### 5.3. Set the permissions on their folder

Let's set the permissions to their folder by doing this command:

```bash
sudo chmod 750 /srv/ftp/Admin

# chmod - Change Mode
```

In this example, that number `750` is the permission itself, represent as:

- **7** - Allow read,write and execute for the user / owner.
- **5** - Allow read and execute for the groups.
- **0** - Other users doesn't allow anything.

**Note:** This is really important to learn the Linux permission about how it really works. I recommend to checkout this and take your time to learn Linux permissions by [Clicking this link.](https://www.digitalocean.com/community/tutorials/how-to-set-permissions-linux)

**But here's the problem**

Although other users can't access the Admin folder, since they're the owners of their own folder, they can change permissions of the folder and able to change who can access which ruined the FTP Setup and having an access error due to modified permissions.

**Unfortunately**

There's no way to prevent them to modify permissions on their own folder since they are the **OWNER** of the folder.

**So here's the solution**

If they can't prevent because they're the owner, then they shouldn't let be the **OWNER** itself. To achieve this those FTP users must move as groups, so follow carefully the next steps.

## 6. Create groups

In your terminal, type this command to create one groups.

```bash
sudo groupadd ftpusers

# sudo groupadd <name of the group>
```

Then after creating groups it's time to add the users to your group.

```bash
sudo usermod -aG ftpusers ftp_admin
```

You may repeat the same command on the other users just change the FTP user.

## 7. Access Control List (ACL) Masking

This is where we masks the permissions to the FTP Users so they can still access the FTP server but cannot modify the permissions.

In case you haven't installed the `ACL`:

```bash
sudo apt install acl
```

After installing, follow the steps on what commands we write and enter to mask the user permission on it's folders.

```bash
# 1. Set the base permissions to the root
sudo chmod 700 /srv/ftp/Admin
sudo chown root:ftpusers /srv/ftp/Admin

# 2. Remove group access
sudo setfacl -m g::--- /srv/ftp/Admin
sudo setfacl -d -m g::--- /srv/ftp/Admin

# 3. Give the User the full access
sudo setfacl -m u:ftp_admin:rwx /srv/ftp/Admin
sudo setfacl -d -m u:ftp_admin:rwx /srv/ftp/Admin

# 4. Fix the Mask
sudo setfacl -m m:rwx /srv/ftp/Admin
sudo setfacl -d -m m:rwx /srv/ftp/Admin

# (Optional) Verify the Folder
getfacl /srv/ftp/Admin
```

You can repeat the entire process to the other folders but make sure look carefully at the FTP **User** and the **Folder path** before enter to avoid any conflicts.

After done, you may check the folder by accessing the FTP Server then try to change permissions on the folder. If they can't change the access then it works perfectly and solved the problem that **OWNERS** can change permissions.

## 8. Backup via rsync

Having a backup in FTP Server is important. In this example, we can achieve near live backup using `rsync`.

First, how `rsync` works? It is a small sudo package that able to sync the folders and files in one command.

Now, let's check if the `rsync` is installed.
```
rsync --version
```
It should print the version of the `rsync`. But if not, then install it.

```bash
sudo apt install rsync
```

After installing, let's create a folder outside **/ftp** where we put the backup folder for the FTP server.

For this example, I'll create a folder outside but you can select whatever file path you want.

```bash
sudo mkdir -p /srv/ftp_mirror

# Set permissions so that everyone cannot access this except the owner
sudo chmod 700 /srv/ftp_mirror
```
Now that **/ftp_mirror** serves as a backup folder of the FTP Server, but again you can choose your own folder where you place your FTP Backup.

Let's try having a dry run of file mirroring, you can achieve this in one command:

```bash
# Dry run syncing
sudo rsync -aAX --delete --dry-run /srv/ftp/ /srv/ftp_mirror/
#sudo rsync <modes> /<Source Folder> /<Destination Folder>

# -a - Archive Mode (Permissions, Owner, Timestamps)
# -A - Preserve ACLs
# -X - Preserve extended attributes.
# -v - Verbose, put that next to X and it prints all the files if copied (Optional)
# --delete -  Removes the files in destination that no longer exists in source

# Example
sudo rsync -aAX -v --delete --dry-run /srv/ftp/ /srv/ftp_mirror/
```

Pay attention to the folders.

```bash
/srv/ftp(/) <-- This part

/srv/ftp_mirror(/) <-- This part
```

Putting a slash **(/)** after the folder name makes to get / place the content **INSIDE** the folder, not the folder itself.

Anyways here's the Real Sync

```bash
# Real Syncing
sudo rsync -aAX --delete /srv/ftp/ /srv/ftp_mirror/
```

After entering that command, it may seems nothing happens but checkout the folder if it really sync. If yes, then it works successfully.

**Tips:** It's best to have a preserved folder, different from your backup.

**Important:** Before you do this command, make sure that the folder **/ftp_mirror** contents is safe and have no any malicious files to prevent the **/ftp_master** get infected with files.

```bash
# Create master directory
sudo mkdir -p /srv/ftp_master/

# Then Sync from Mirror to Master
sudo rsync -aAX --delete /srv/ftp_mirror/ /srv/ftp_master/
```

So the folders would be:

> **/ftp** - The Live FTP Server where everyone accessing.
> 
> **/ftp_mirror** - The backup mirror sync folder and it keep syncing from live server.
> 
> **/ftp_master** - The preserved folder that more stable and also serve as the "Main Backup" that no one can touch, but can be updated too. 

## 9. Automation

This is where the backup sync get automated, using `crontab` we can achieve near real-time syncing.

The main purpose of `crontab` is to allow to make a scheduled commands or scripts run automatically, so we can use it.

In this example, we set the `crontab` in sync the FTP server every hour. To begin, first access the `crontab` by writing this command:

```bash
sudo crontab -e

# Note: Require "sudo" because it's for admin
```

Then write the following code:

```bash
0 * * * * rsync -aAX --delete /srv/ftp/ /srv/ftp_mirror/
```

Then save it, now it will trigger every hour to sync.

But if you want to customize the schedule, you can learn how to set the schedule by [clicking this link.](https://www.geeksforgeeks.org/linux-unix/crontab-in-linux-with-examples/)

## 10. Recovery

Now let say you accidentally deleted a file and you need to recover it, the command are simply:

```bash
# Example
sudo rsync -aAX --delete /srv/ftp_mirror/Admin/ /srv/ftp/Admin/

# Return the permissions
sudo chomod 770 /srv/ftp/Admin
```

It's important to select one folder **ONLY** to avoid recover old files and loss the new files that uploaded on other folder.

## 11. Logs

It's important to trace the logs of the FTP Server to track all the authentication, change directories and any actions or operations made inside the FTP Server for each users.

To see the sample logs so far, you can type this simple command:

```bash
sudo cat /var/log/vsftpd.log
```

Then it immediately prints all the logs.

In case you want to reset the log, simply type this command:

```bash
sudo truncate -s 0 /var/log/vsftpd.log

# truncate - Shrink  or extend the size of a file and it changes the file without deleting the file
# -s - set size
# 0 set the file to zero bytes 
```

That way, it resets all the logs from the `vsftpd.log`.

However, you can watch the live logs while the FTP Server is running. You can watch the live logs through this command:

```bash
sudo tail -f /var/log/vsftpd.log

# tail - Displays the end (last part) of a file
# -f - Follow
```

It shows the last 10 lines of the `vsftpd.log`, you can stop the action by pressing **CTRL + C**.

### Logrotate

So what is `logrotate`? It's a Linux utility that manages the automatic rotation and compression of log files. It helps prevent log files from consuming all available disk space by periodically renaming, compressing, and generating fresh log files.

So what we gonna do, is to make a `logrotate` for the FTP logs that resets the log every week so that the file won't consume large space and can separate logs.

We can achieve that by following this steps. First, let's open a file:

```bash
sudo nano /etc/logrotate.d/vsftpd
```

Upon open you will see the some code already written but you can replace it with this code:

```bash
/var/log/vsftpd.log {
    weekly
    rotate 4
    missingok
    notifempty
    compress
    create
    delaycompress
    create 640 root adm
    su root adm
}

# weekly - rotate every week
# rotate 4 - keep 4 old logs (≈ 1 month)
# compress - old logs become .gz
# create - creates a fresh empty log (this is your “reset”)
# missingok - no error if log doesn’t exist
# notifempty - don’t rotate empty logs
# delaycompress - Delays compression until the next roation cycle
```

This will auto-reset your logs every week, keeping the previous backup compressed.

### Force Log rotation

This may be optional but here's how you test the `logrotate`,

```bash
# Safe Mode / Dry-run
sudo logrotate -d /etc/logrotate.d/vsftpd

# Real force logrotate
sudo logrotate -f /etc/logrotate.d/vsftpd
```

This command will trigger the `logrotate` and do the action based on codes inside the file.