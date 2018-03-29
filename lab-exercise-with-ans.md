# Lab 3
Perform the following steps before power on the virtual machine.  
1. Add 2 5GB hard disks to the virtual machine.  
2. Change the “Network Connection” of the network adapter from NAT to Host-only.  
3. Save the settings and create a duplicate copy of the virtual machine to Desktop.

## Part 1: User Management

### Tasks:
1. Create a file named “User_guide” in /etc/skel directory.
```
cd /etc/skel
touch User_guide.txt
```
*Self-check: What is the purpose of the /etc/skel directory?*
```
When new user is created, their files will be copy from /etc/skel to their home directory.
```

2. Create a new user with the following information.
Username	: stadm
Default password	: st12345
```
sudo useradd -d /home/stadm -m stadm
sudo passwd stadm
# System will prompt for password
```

Self-check:
How to check the existence of an user?
```
cat /etc/passwd | grep <user_name>
```

3. Create a shell script that contains the following commands.
```
useradd –m $1
echo $1:depw12345 | chpasswd
```

Self-check:
What is $1?   **First argument passed to the script.**
Explain the operation of the second line.  
```
To set a user with name specify in $1 to have the password of 'depw12345'
```
How to display the permission of a file?  
```
ls -l <file_name>
```
What is the command to make the script executable?  
```
chmod u+x <script_name>
```

4. Create a new user (user name: stuser) using the script created in (3).
```
sudo ./the_script stuser
```

5. Execute the command that will force stuser to change password when first login.
```
sudo passwd stuser -e
```

6. Create two new user groups:
stgroup
supadmin
```
sudo addgroup stgroup
sudo addgroup supadmin
```

Self-check:
How to confirm that the groups have been created in the system?
```
cat /etc/group | grep <group_name> | wc -l
# If results is 1 means the group exists
# If results is 0 means the group does not exist
```

7. Assign stadm to supadmin group and stuser to stgroup.
```
sudo usermod -g supadmin stadm
sudo usermod -g stgroup stuser
```

Self-check:
How to know the groups that an user assigned to?
```
groups <user_name>
```
How to remove an user from a group?
```
sudo deluser <username> <group_name>
```
How to remove a group from system?
```
sudo delgroup <group_name>
```

8. Remove user stuser from system but keep the home directory.
```
sudo deluser stuser
```


Part 2: Storage Management - LVM

1. Configure the 2 additional hard disks as physical volumes.
```
This one must use the VirtualBox or VMWare UI to do this.
```

Self-check:
How to display a list of all physical volume created?
```
sudo fdisk -l
```
How to check the space in physical volume?
```
ncdu
```

2. Create a volume group named st_group.
```
# Before this, you need to install lvm2
sudo apt install lvm2
sudo vgcreate st_group /dev/sdb
```

Self-check:
How to display a list of all volume group created?
```
sudo vgdisplay
```

3. In the first additional hard disk, create a logical volume of size 2GB, named as st_data for st_group.
```
sudo lvcreate -n st_data -L 2G st_group
```

Self-check:
How to display a list of logical volume created?
```
sudo lvdisplay
```

4. Format the logical volume as ext4 filesystem.
```
sudo mkfs.ext4 /dev/mapper/st_group-st_data
```
5. Create a mount point, /mnt/lvm/st_lvm, to mount the logical volume.
```
cd /mnt
sudo mkdir lvm
cd lvm
sudo mkdir st_lvm
```
6. Mount the logical volume.
```
sudo mount /dev/mapper/st_group-st_data /mnt/lvm/st_lvm
```
7. Add the second additional hard disk to st_group.
```
vgextend st_group /dev/sdc
```
8. Extend the logical volume size by 2GB from the first physical volume, 3GB from the second physical volume.
```
sudo lvextend -n /dev/mapper/st_group-st_data -L 2G
```



Part 3: SSH

1. Power on the second virtual machine.
2. Configure both virtual machines to use static IP in the network 192.168.49.0, change the second machine’s host name to server2.
```
# Must set both VMs to use Host-only adapter
# We must configure the IP address from 100 onwards, because 0-100 is reserved.
# E.g. 192.168.49.0 means the network
# E.g. 192.168.49.1 means the gateway
```


Make changes to the appropriate files after the host name has been changed.
Restart the network service after configuration.
Make sure the machines can communicate with each other, e.g. use ping command.

3. Configure SSH server to listen to port 10022.
```
sudo sed -i 's/Port 22/Port 10022/g' /etc/ssh/sshd_config
sudo service ssh restart
```

How to connect to a different port?
```
# Use ssh -p <port_number> <destination_ip>
ssh -p 10022 10.0.0.10
```

Self-check:
How to display the port that SSH server listen to?
```
sudo netstat -tulpn | grep ssh
```

4. Create a new user (user name = st_mgr) in both machine, assigned that user to st_group.
```
# Copy the following command on both server1 and server2
sudo useradd -d /home/st_mgr -m st_mgr
sudo addgroup st_group
sudo usermod -g st_group st_mgr
sudo passwd st_mgr
# I'll use 123 as password for this example
```
5. Perform necessary configuration to allow st_mgr to login to server1 via SSH with key authentication.
```
# In server2
su st_mgr
# Type in 123

ssh-keygen
# Press enter for every prompt

# Note that 10.0.0.10 is the IP of server1
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 10022 10.0.0.10 

# Then type in the password which is 123

# Now you can login to server1 using the following command
ssh -p 10022 10.0.0.10
```

6. Configure SSH server to allow only user st_mgr and user group st_adm to login via SSH.
```
# Switch back to root user, in this case my root user is vagrant, and password is 'vagrant'
su vagrant

sudo bash -c 'echo "" >> /etc/ssh/sshd_config'
sudo bash -c 'echo "AllowUsers st_mgr" >> /etc/ssh/sshd_config'
sudo bash -c 'echo "AllowGroups st_adm" >> /etc/ssh/sshd_config'

# Restart the ssh server
sudo service ssh restart
```