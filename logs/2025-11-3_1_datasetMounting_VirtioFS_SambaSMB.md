Okay I did a lot of things yesterday.

I setup multiple containers and VMs for different purposes. 

I created a dataset called /tank/media, which has a bunch of subdirectories (initially each subdirectory was its own 
dataset, but I decided I don't need different rules and snapshots for each, just one for the whole media storage block)

The structure is:
```
/tank/media
|_ audiobooks
|_ books
|_ tv
|_ movies
|_ other
```

<h3>Container Bind-Mounting</h3>
This dataset is "bind-mounted" within my containers that need to access it (i.e. audiobookshelf). To do this, in the host you edit the file: 

`/etc/pve/lxc/<CTID>.conf` (proxmox specific system here), where <CTID> is the ID of 
the container (100, 101, 102, ...). Inside this file, add a line with:<br>
  
`mpX: <host path>,mp=<container path>,[options]`

For example:<br>
`mp0: /tank/media,mp=/mnt/media,ro=1,backup=0` (read only = true, do not backup)

Now, within the container I can access everything inside of the /tank/media dataset inside of /mnt/media in my container.

<h3>VM Mounting w/ VirtioFS</h3>
For VMs, it's a bit different. There are a few ways to go about it, but I used VirtioFS which is a pretty simple system.
Here's a good tutorial: woshub.com/proxmox-shared-host-directory/
But basically you add the dataset to the VM as a VirtioFS 'hardware' thing in the Hardware section within the 
Proxmox GUI or in the CLI, and it will have a tag, i.e. 'media'. Then, inside the VM you create a mount point, 
i.e. /mnt/media, and mount media in that spot: 

`sudo mount -t virtiofs media /mnt/media`

Then, to mount it on boot permanently, edit `/etc/fstab` with a new line:
`media  /mnt/media  virtiofs  defaults,_netdev  0  0`

<h3>Samba SMB share</h3>
The other thing that I did was setup a Samba SMB share. I wanted to put some data into this dataset from my Windows machine,
and the best way to do that is with an SMB, which is also super easy to set up.

https://ubuntu.com/tutorials/install-and-configure-samba#3-setting-up-samba

On the host, install Samba `apt install Samba`.
Then, in the conf file in `/etc/samba/smb.conf`, you create a new share like this:

```
[media]
path = /tank/media
browseable = yes
read only = no

guest ok = no
valid users = solomon
force user = root
create mask = 0664
directory mask = 0775
```

With other options available, but only the top ones necessary.
Then create a password `smbpasswd -a yourusername`.

Now, after reset, I can access the dataset from windows by running Win+R, `\\[proxmox_host_IP]\media`. With the proxmox host
IP at the start, and the tag afterward. It's pretty quick and I can move files back and forth easily.


