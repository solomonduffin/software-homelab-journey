Out of town for a few weeks. While I was gone I was doing research on different things. One thing I realized is 
that for what I want to do with this lab I need Proxmox, or some other hypervisor. They're more flexible in every way, 
and good practice for lots of difficult/useful things that I'm sure will pop up. But anyway I'm glad I did what I did 
with Debian, even though I'm going to completely restart now. I think having the simple environment to dip my toes in 
was a good call, and even though I didn't do that much, I feel more confident overall in the whole process. Now, switching 
to a hypervisor will allow me to really have fun with different Linux distros, and have even more containerization between 
my services. It should also be good for dipping into what system admins do? Although I'm not completely sure about that.


So first I did a few things to prepare my server. I honestly didn't care about anything there. I could have saved some config files,
ssh keys, etc..., but I'd rather set it all up again to get the practice. So really all I did was save some network
information for the Proxmox setup, and wipe the hard drives and RAID config because I'll be setting up a new storage system in
Proxmox.

The Promox install was easy, and now I can access the graphical interface through the local port https://192.168.1.50:8006.

## Understanding Proxmox
Reading the Proxmox docs https://pve.proxmox.com/pve-docs/, it's clear that this can be as complex as I want it to get.
It seems like this is definitely made for both personal and enterprise use. If I wanted to run multiple servers and cluster
them together, and I'm still iffy on how that works, I could do that.


### What I want to do next:
- Look at firewall settings and do basic hardening
- Setup a ZFS raid system
- Setup some containers and VMs and mess around with them
- See how Backblaze integrates with this system and pay for cloud storage once I start really putting important data on this
- Get back to what I had with Debian, setup some hosted servers

### The main things I want to understand:
- Linux Containers vs VMs
- The current storage setup on the host SSD (two partitions, one is EXT-4 and the other is LVM) why is it split and what are they doing?
- This important looking page: https://pve.proxmox.com/wiki/Host_System_Administration
- How to setup ZFS, and what a ZFS Pool is
- Passthrough
- How to create a VM/container, and how those access and store data
- Firewall rules and hardening
- User creation, permissions for hardening, create admin and disable root?
- Authentication Realms (Linux PAM vs built-in Proxmox authentication)
- Network bridges
