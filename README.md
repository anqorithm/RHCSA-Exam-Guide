# RHCSA Exam Guide - EX200

<p align="center">
  <img src="assets/red-hat.svg" width="400" alt="Red Hat Logo">
</p>

## What is this certification?

The RHCSA (Red Hat Certified System Administrator) certification is a strong reference for professionals working in the Linux operating system field. This exam provides a comprehensive assessment of an individual's skills in managing systems running Red Hat Enterprise Linux.

The certification is based on an individual's performance in a set of practical tasks, such as:
- File and directory management
- Network configuration
- User and permissions management
- Services and scheduled tasks management
- Security and access control
- And other essential system administration skills

Obtaining the RHCSA certification indicates technical skills in Linux system administration and reflects a commitment to professional development and seeking knowledge and experience in this evolving field.

The RHCSA exams require practical skills on computers, specifically on the Red Hat Enterprise Linux operating system. Individuals must prepare for the exam carefully, through studying educational materials and practical training on the tools and techniques used in the operating system.

## Exam Information

- The exam is fully on Linux system
- Duration: 3 hours (remote or in-person at testing centers)
- Approximately 22 questions
- Exam score: 300 points with 210 points required to pass

## Exam Environment

The exam environment will have these options:
- First option: Opens the questions page and instructions
- Second option: Opens to run the first or second node, with options like stop and start
- Third option: Opens a terminal for the workstation where you can connect via SSH to the first or second node for easier copy and paste

**Note:** For each service you configure, follow these steps:
```systemctl start [service]
systemctl enable [service]
[Configure the service]
systemctl restart [service]
systemctl enable [service]
```

As a precaution, check the service status at the beginning or end: `systemctl status [service]`

## Questions and Solutions

### Node 1

#### Q1: Network Configuration
**Task:**
- hostname = node1.lab.example.com
- ip-address = 172.25.250.10
- subnetmask = 255.255.255.0
- gateway = 172.25.250.254
- dns = 172.25.254.254

**Solution:**
```bash
# Check current connections
nmcli connection show

# Configure the connection
nmcli connection modify "Wired connection 1" ipv4.address 172.25.250.10/24 ipv4.gateway 172.25.250.254 ipv4.dns 172.25.254.254 ipv4.method manual connection.autoconnect yes

# Set hostname
hostnamectl set-hostname node1.lab.example.com

# Restart network manager
systemctl restart NetworkManager
systemctl enable NetworkManager

# Bring up the connection
nmcli connection up "Wired connection 1"
```

**Verify:**
```bash
reboot
hostname
cat /etc/resolv.conf
ip a
route -n
ping 172.25.250.10
ping 172.25.250.254
```

**Enable SSH access on Node 1 and Node 2:**
```bash
vim /etc/ssh/sshd_config
# Find second word, delete it and write "yes"
systemctl restart sshd
systemctl enable sshd
```

Now you can access nodes through terminal:
```bash
ssh root@node1
```

#### Q2: Configure Repository
**Task:** Configure YUM repository with the 2 given links (BaseOS and AppStream)
- BaseOS: http://content.example.com/rhel8.0/x86_64/dvd/BaseOS
- AppStream: http://content.example.com/rhel8.0/x86_64/dvd/AppStream

**Solution:**
```bash
# Check current repos
yum repolist

# Create repository configuration
vim /etc/yum.repos.d/local.repo
```

Add the following content to the file:
```
[BaseOS]
name=BaseOS
baseurl=http://content.example.com/rhel8.0/x86_64/dvd/BaseOS
enabled=1
gpgcheck=0

[AppStream]
name=Appstream
baseurl=http://content.example.com/rhel8.0/x86_64/dvd/AppStream
enabled=1
gpgcheck=0
```

Then update your package lists:
```bash
yum clean all
yum update
```

**Verify:**
```bash
yum repolist
yum install nmap -y
```

#### Q3: Debug SELinux
**Task:** Web server running on non-standard port "82" is having issues serving content. Debug and fix the issues.
- The web server can serve all the existing HTML files from '/var/www/html'. Don't make any changes to these files.
- Web service should automatically start at boot time.
- curl servera.lab.example.com:82

**Solution:**
```bash
systemctl start httpd
systemctl enable httpd

# Check SELinux ports
semanage port -l | grep http

# Add port 82 to http_port_t type
semanage port -a -t http_port_t -p tcp 82

# Verify port addition
semanage port -l | grep http

# Configure firewall
firewall-cmd --permanent --add-port=82/tcp
firewall-cmd --reload
```

**Verify:**
```bash
systemctl restart httpd
curl servera.lab.example.com:82
```

#### Q4: Cron Job
**Task:**
- Configure a cron job that runs every 1 minute and executes: logger "EX200 in progress" as the user natasha.
- Configure a cron job for user "natasha", cron must run daily at 2:23pm and execute the /usr/bin/echo "welcome"

**Solution:**
```bash
systemctl start crond
systemctl enable crond

# Edit crontab for natasha
crontab -eu natasha
```

Add the following lines to the crontab:
```
*/1 * * * * logger "EX200 in progress"
23 14 * * * /usr/bin/echo "welcome"
```

Then:
```bash
systemctl restart crond
systemctl enable crond
```

**Verify:**
```bash
crontab -lu natasha
tail -f /var/log/messages | grep "EX200"
```

#### Q5: Create User accounts with supplementary group
**Task:**
- group: sysadms
- users: natasha harry sarah (with nologin shell)
- natasha and harry should be the member of sysadms group.
- password for all users should be "trootent"

**Solution:**
```bash
groupadd sysadms
useradd -G sysadms natasha
useradd -G sysadms harry

# Find nologin path
which nologin

# Create user with nologin shell
useradd -s /usr/sbin/nologin sarah

# Set passwords
passwd natasha  # Enter "trootent"
passwd harry    # Enter "trootent"
passwd sarah    # Enter "trootent"
```

**Verify:**
```bash
cat /etc/group       # or cat /etc/group | grep sysadms
cat /etc/passwd
```

#### Q6: Create a collaborative DIR
**Task:**
- Create the Directory "/home/sysadms" with the following characteristics.
- Group ownership of "/home/sysadms" should go to "sysadms" group.
- The directory should have full permission for all members of "sysadms" group but not to the other users except "root".
- Files created in future under "/home/sysadms" should get the same group ownership.

**Solution:**
```bash
mkdir -p /home/sysadms
chgrp sysadms /home/sysadms
ls -ld /home/sysadms
chmod 770 /home/sysadms
chmod g+s /home/sysadms
```

**Verify:**
```bash
su - harry
touch /home/sysadms/test
ls -ld /home/sysadms/test
```

#### Q7: Configure NTP
**Task:** Configure NTP where chrony server is "classroom.example.com"

**Solution:**
```bash
systemctl start chronyd
systemctl enable chronyd
timedatectl
timedatectl set-ntp true
timedatectl

# Edit chrony configuration
vim /etc/chrony.conf
```

Add the server name inside the file:
```
server classroom.example.com iburst
```

Then:
```bash
systemctl restart chronyd
systemctl enable chronyd
```

**Verify:**
```bash
chronyc sources
```

#### Q8: Configure AutoFS
**Task:**
- NFS exports the /home/guests to your system.
- remoteuser6 home directory is classroom.example.com:/home/remoteuser6
- remoteuser6 home directory should be automounted locally beneath at /home/guests/remoteuser6
- while login with remoteuser6 then only home directory should be accessible from your system for remoteuser6 and password is "redhat"

**Solution:**
```bash
systemctl start autofs
systemctl enable autofs
getent passwd remoteuser6

# Edit master configuration
vim /etc/auto.master
```

Add this line:
```
/home /etc/auto.misc
```

Create/edit misc configuration:
```bash
vim /etc/auto.misc
```

Add this line:
```
remoteuser6 -rw,nfs,sync classroom.example.com:/home/remoteuser6
```

Then:
```bash
systemctl restart autofs
systemctl enable autofs
```

Optional:
```bash
chmod 777 /home/
```

**Verify:**
```bash
su - remoteuser6
pwd
```

#### Q9: Create user with specific UID
**Task:** Create user 'bob' with 2112 uid and set the password 'trootent'

**Solution:**
```bash
useradd -u 2112 bob
passwd bob  # Enter "trootent"
```

**Verify:**
```bash
id bob
```

#### Q10: Create an archive
**Task:** Create an archive '/root/test.tar.gz' of '/var/tmp' dir and compress it with gzip.

**Solution:**
```bash
tar -zcvf /root/test.tar.gz /var/tmp
```

**Verify:**
```bash
ls /root/test.tar.gz
```

#### Q11: Find and copy files
**Task:** Locate all files owned by sarah and copy them under /root/find_user

**Solution:**
```bash
mkdir /root/find_user
find / -user sarah -type f
find / -user sarah -type f -exec cp -rfvp {} /root/find_user \;
```

**Verify:**
```bash
ls /root/find_user
```

#### Q12: Find string and save to file
**Task:** Find a string 'home' from '/etc/passwd' and put it into '/root/search.txt' file

**Solution:**
```bash
grep "home" /etc/passwd
grep "home" /etc/passwd > /root/search.txt
```

**Verify:**
```bash
cat /root/search.txt
```

#### Q13: Create a container image
**Task:** Create a container image named monitor with the given link. Do not edit the Container file.
Login with user "student" to perform all the questions related to container.

**Solution:**
Open two terminal windows, one for root and one for the student user:

As student:
```bash
ssh student@node1
podman login [link from instruction]
# Enter username and password
wget [link from question]
podman build -t monitor .  # Note: the period is part of the command
podman images
```

#### Q14: Container configuration
**Task:**
1. Create a Container name asciipdf
2. Use monitor image for asciipdf which you previously created
3. Create a systemd service named container-asciipdf for "student" user only
4. Service will automatically start across reboot with no manual intervention
5. Local host Directory /opt/files attach to Container directory /opt/incoming
6. Local host Directory /opt/processed attach to container host directory /opt/outgoing

**Solution:**
As root:
```bash
ssh root@node1
mkdir /opt/files /opt/processed
chown student:student /opt/files /opt/processed
chmod 777 /opt/files /opt/processed
loginctl enable-linger student
```

As student:
```bash
podman run -d --name ascii2pdf -v /opt/files/:/opt/incoming/:Z -v /opt/processed/:/opt/outgoing/:Z monitor
podman ps
```

To verify the container works:
```bash
touch /opt/files/data
ls /opt/processed
file /opt/processed/data
```

Create systemd service files:
```bash
mkdir -p .config/systemd/user
cd .config/systemd/user
podman generate systemd --name ascii2pdf --new --files
ls
systemctl --user daemon-reload
systemctl --user start container-ascii2pdf.service
systemctl --user enable container-ascii2pdf.service
```

As root:
```bash
reboot
```

As student:
```bash
ssh student@node1
podman ps
```

### Node 2

#### Q15: Break node 2 password
**Task:** Reset the root password

**Solution:**
1. Choose the option to restart the node, then select grub2 or the option with an arrow
2. Press Ctrl + e
3. Write `(rd.break)` at the end of the linux line
4. Execute the following:
```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root  # Set new password
touch /.autorelabel
exit
exit
```
The node will reboot automatically.

#### Q16: Configure yum repository
Same as question 2 but with different links.

#### Q17: Swap partition
**Task:** Add a swap partition of 512MB and mount it permanently.

**Solution:**
```bash
free -m
lsblk
fdisk /dev/vdb
```

In fdisk:
- n (new partition)
- p (primary)
- Press Enter (default partition number)
- Press Enter (default first sector)
- +512MiB (partition size)
- t (change partition type)
- Press Enter
- L (list partition types) and select the number for "swap" (or directly enter "swap")
- w (write changes)

Then:
```bash
partprobe /dev/vdb
mkswap /dev/vdb1
```

Copy the UUID that is displayed and add it to fstab:
```bash
vim /etc/fstab
```

Add this line (with your UUID):
```
UUID=219c38e0309480923849823894038 swap swap defaults 0 0
```

Then:
```bash
free -m
swapon -a
free -m
```

**Verify:**
```bash
reboot
lsblk
```

#### Q18: Create a Logical Volume and mount it permanently
**Task:**
- Create a logical volume with the name "lvname" by using 50 extents from the volume group "groupname".
- Consider size of volume group as "8MB".
- Mount it on '/mnt/lvname' with file system ext3.

**Solution:**
```bash
lsblk
fdisk /dev/vdb
```

In fdisk:
- n (new partition)
- p (primary)
- Press Enter (default partition number)
- Press Enter (default first sector)
- +1G (partition size)
- t (change partition type)
- Press Enter
- L (list partition types) and select the number for "Linux LVM" (or directly enter "lvm")
- w (write changes)

Then:
```bash
partprobe /dev/vdb
pvcreate /dev/vdb2
pvs
vgcreate -s 8M groupname /dev/vdb2
vgs
lvcreate -l 50 -n lvname groupname
mkfs.ext3 /dev/groupname/lvname
mkdir /mnt/lvname
vim /etc/fstab
```

Add this line to fstab:
```
/dev/groupname/lvname /mnt/lvname ext3 defaults 0 0
```

Then:
```bash
mount -a
lsblk
```

**Verify:**
```bash
reboot
lsblk
```

#### Q19: Resize the logical volume
**Task:** Resize the logical volume "lvname" to be 230MiB. After reboot, size should be between 200MB to 300MB

**Solution:**
```bash
lvextend -r -L 320MiB /dev/groupname/lvname
mount -a
resize2fs /dev/groupname/lvname
```
Note: If the file system was xfs, the command would be `xfs_growfs /dev/groupname/lvname`

**Verify:**
```bash
reboot
lsblk
```

#### Q20: Configure System Tuning
**Task:** Choose the recommended 'tuned' profile for your system and set it as the default

**Solution:**
```bash
systemctl start tuned
systemctl enable tuned
tuned-adm active
tuned-adm recommend
tuned-adm profile [output from previous command]
tuned-adm active
systemctl restart tuned
systemctl enable tuned
```

### Extra Questions
One of these will appear in the exam:

#### Extra Q1: Make a Simple script
**Task:**
- Create a mysearch script to locate all files under /usr/share having size 10M.
- After executing mysearch script, files should be copied under /root/myfiles

**Solution:**
```bash
mkdir /root/myfiles
vim mysearch.sh
```

Add the following content:
```bash
#!/bin/bash
find /usr/share -type f -size -10M -exec cp -prvf {} /root/myfiles \;
```

Then:
```bash
chmod u+x mysearch.sh
./mysearch.sh
```

**Verify:**
```bash
ls -a /root/myfiles
```

#### Extra Q2: Expire password
**Task:** Make all local users present in the system have passwords expire in 30 days

**Solution:**
For a single user (harry):
```bash
chage -M 30 harry
```

For all users, modify the configuration file:
```bash
vim /etc/login.defs
```

Find and change:
```
PASS_MAX_DAYS 30
```

**Verify:**
```bash
cat /etc/shadow
```

#### Extra Q3: Configure application
**Task:** Configure the application RHCSA as harry user; when logging in, it will show the message "welcome to advantage pro"

**Solution:**
```bash
su - harry
vim .bash_profile
```

Add the following content:
```bash
RHCSA="welcome to advantage pro"
export RHCSA
echo $RHCSA
```

Then:
```bash
source .bash_profile
```

**Verify:**
```bash
# Logout and log back in
su - harry
# Should see: Welcome to advantage pro
```

#### Extra Q4: Sudo privileges
**Task:** Configure sudo for group 'elite' so that its members should have sudo access with no password

**Solution:**
```bash
visudo
```

Add the following line:
```
%elite ALL=(ALL) NOPASSWD: ALL
```

**Verify:**
Switch to a user from the elite group (create one if needed and add to elite group):
```bash
su - username
sudo useradd test
sudo cat /etc/passwd
```

#### Extra Q5: ACL
**Task:** Copy "/etc/fstab" file to "/var/tmp", then configure "/var/tmp/fstab" file permissions with ACL:
- The file /var/tmp/fstab should be owned by "root"
- The file /var/tmp/fstab should belong to the group "root"
- The file /var/tmp/fstab shouldn't be executable by anyone
- The user "sarah" should be able to read and write to the file
- The user "harry" can neither read nor write to the file
- Other users (future and current) should be able to read /var/tmp/fstab

**Solution:**
```bash
cp /etc/fstab /var/tmp
chown root:root /var/tmp/fstab
chmod ugo-x /var/tmp/fstab
setfacl -m u:sarah:rw /var/tmp/fstab
setfacl -m u:harry:0 /var/tmp/fstab
chmod o=r /var/tmp/fstab
```
