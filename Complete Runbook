# I use the default "pi" user for this
#ls /dev/disk/by-uuid
fdisk -l | grep '^Disk'
#Find the new HDD and /dev/sd*
#7412602F-F69B-4D7C-9A99-EEB6C68E2D3E
sudo fdisk /dev/sda
#Press "n", if Oartition number is default 2 or higher - do nothing | otherwise 1 - first sector (ENTER)- -> last sector (ENTER) -> then "w"
sudo mkfs.ext4 /dev/sda1
# Press ENTER 2x
# Copy UUID d1ed4057-3bcb-4092-9d0b-20a3f11d51b2
sudo mkdir /backup-disk
sudo chown -c pi /backup-disk/
sudo mount /dev/sda1 /backup-disk/
#check if disk is listed
df -h
sudo nano /etc/fstab
UUID=*partition UUID* /backup-disk ext4 defaults
sudo reboot
# check if HDD will be active after reboot
df -h

#Change pi password or change user
passwd
#Generate ssh key + 3x enter
ssh-keygen -b 4096
cat /home/pi/.ssh/id_rsa.pub
# or
ssh-keygen -t ed25519
cat /home/pi/.ssh/id_ed25519.pub
#add your public key to the system
# if you publish it directly to the internet - set "PasswordAuthentication no" you can find it with nano STRG+W password
sudo nano /etc/ssh/sshd_config
# If you tunnel your system trough to your internal and / or use a Synology NAS - you need to keep it at yes
#Copy public key from cat /home/pi/.ssh/id_ed25519.pub

# Choose your type of Jumphost
# In the web or at home, @home it only works, if you are able to forward / open port to your internal system.
# If you want everything running over a connection broker, have an internal jumphost, thats connecting to the external one.
# So you could only use Keys in the web and map the ssh port from the backupPI with PW only to a port on your internal jumphost
# You can follow Martin Sauter / Heurekus guide or just go ahead https://blog.wirelessmoves.com/2019/06/how-to-run-a-server-at-home-without-an-ipv4-address.html
# or even watch his talk about it (german only) https://media.ccc.de/v/gpn19-76-einen-server-daheim-ohne-ffentliche-ipv4-adresse
# I let the SSH run on 22 but you can change it, if you like

# on my web based jumphost I've modify the ssh config by adding this (Yes I know, that I'm running this as the default master user (root)
# If you have a "normal" user, you are limited to ports over 1024
Webjumphost:
sudo nano /etc/ssh/sshd_config
# Add the following
GatewayPorts yes
ClientAliveInterval 15
ClientAliveCountMax=2
# Import your public key first, then set "PasswordAuthentication no" you can find it with nano STRG+W password the Webjumphost should not need it
# If you want so see if the mapping works you can run 
watch -n 0.5 "netstat -tulpn"

Homejumphost:
sudo apt install autossh
crontab -e
# add the following
# I mapp the port to 2049 on the external jumphost
@reboot autossh -M 0 -f -o ConnectTimeout=2 -o ServerAliveInterval=15 -o ServerAliveCountMax=2 -p 22 -N -R 2049:localhost:22 USER@WEBJUMPHOST
# ssh 1x to the Webjumphost so your system knows the devicekey
ssh USER@WEBJUMPHOST -p YOUR_SSH_PORT
sudo reboot
# add public key from BackupPI to .ssh/authorized_keys
nano .ssh/authorized_keys
# you can also run the watch if the mapping of the backuppi is working
watch -n 0.5 "netstat -tulpn"

BackupPI:
sudo apt install autossh
@reboot autossh -M 0 -f -o ConnectTimeout=2 -o ServerAliveInterval=15 -o ServerAliveCountMax=2 -p 22 -N -R 2051:localhost:22 USER@HOMEJUMPHOST
# ssh 1x to the Webjumphost so your system knows the devicekey
ssh USER@WEBJUMPHOST -p YOUR_SSH_PORT
sudo reboot

Synology Hyper Backup:
+ -> New -> rsync
- rync kompatibel
- Jumphost IP or DNS name
- Security on
- Port ****
- User pi
- Password = your PW
- Backupmodul: /backup-disk
- $BACKUP_NAME
