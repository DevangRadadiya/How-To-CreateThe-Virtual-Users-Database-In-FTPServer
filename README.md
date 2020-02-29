# How-To-CreateThe-Virtual-Users-Database-In-FTPServer
Avirtualuserisauserloginwhichdoesnotexistasarealloginonthesystemin  /etc/passwdand/etc/shadowfile. Virtualuserscanthereforebemoresecurethanrealusers,becauseacompromised accountcanonlyusetheFTPserverbutcannotlogintosystemtouseotherservices suchasSSHorSMTP.

1. Create The Virtual Users Database In FTP Server
 
A virtual user is a user login which does not exist as a real login on the system in /etc/passwd and /etc/shadow file. 
 
Virtual users can therefore be more secure than real users, because a compromised account can only use the FTP server but cannot login to system to use other services such as SSH or SMTP.
2. Step-By-Step Guide
 
2.1. Create The Virtual Users Database
First create a plain text files with the usernames and password on alternating lines.
For e.g. create user called "Devang" with password called "Dev" and “NSS” with password "NSS2123":
# mkdir /etc/vsftpd # if necessary
# cd /etc/vsftpd
# sudo vim vusers.txt

2.2. Sample Output:
Devang
Dev
NSS
NSS2123
Next, create the actual database file like this (may require the db_util package to be installed first):
# db_load -T -t hash -f vusers.txt vsftpd-virtual-user.db
# chmod 600 vsftpd-virtual-user.db # make it not global readable
# rm vusers.txt

2.3. Configure VSFTPD For Virtual User
Edit the vsftpd configuration file (/etc/vsftpd.conf). Add or correct the following configuration options, depending on if they're already listed somewhere in the file or not (or just add these all to the bottom):
anonymous_enable=NO
local_enable=YES
# Virtual users will use the same privileges as local users.
# It will grant write access to virtual users. Virtual users will use the
# same privileges as anonymous users, which tends to be more restrictive
# (especially in terms of write access).
virtual_use_local_privs=YES
write_enable=YES
# Set the name of the PAM service vsftpd will use
pam_service_name=vsftpd.virtual
# Activates virtual users
guest_enable=YES
# Automatically generate a home directory for each virtual user, based on a template.
# For example, if the home directory of the real user specified via guest_username is
# /home/virtual/$USER, and user_sub_token is set to $USER, then when virtual user devang
# logs in, he will end up (usually chroot()'ed) in the directory /home/virtual/devang
# This option also takes affect if local_root contains user_sub_token.
user_sub_token=$USER
# Usually this is mapped to Apache virtual hosting docroot, so that
# Users can upload files
local_root=/home/vftp/$USER
# Chroot user and lock down to their home dirs
chroot_local_user=YES
# Hide ids from user
hide_ids=YES
#
allow_writeable_chroot=YES
Save and close the file.

2.4. Create A PAM File Which Uses Your New Database
The following PAM is used to authenticate users using your new database. Create /etc/pam.d/vsftpd.virtual: # sudo vim /etc/pam.d/vsftpd.virtual

2.5. Append (Or Create With) The Following:
#%PAM-1.0
auth       required     pam_userdb.so db=/etc/vsftpd/vsftpd-virtual-user
account    required     pam_userdb.so db=/etc/vsftpd/vsftpd-virtual-user
session    required     pam_loginuid.so
Create The Location Of The Files
You need to set up the location of the files / dirs for the virtual users. Type the following command: # mkdir /home/vftp
# mkdir -p /home/vftp/{devang,NSS}
# chown -R ftp:ftp /home/vftp

2.6. Restart The FTP Server
Type the following command:
# systemctl restart vsftpd
 
 
 
Test Your Setup
Open another shell session and type:
$ ftp localhost
[root@devang vsftpd]# ftp localhost
Trying ::1...
Connected to localhost (::1).
220 (vsFTPd 3.0.2)
Name (localhost:root): devang
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||37060|).
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Feb 29 04:38 Devang
-rw-r--r--    1 ftp      ftp             0 Feb 29 04:35 devang.txt
226 Directory send OK.
ftp>
 
