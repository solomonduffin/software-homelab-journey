<h3>VPNs</h3>
<h4>Wireguard setup</h4>

Okay, I'm not gonna lie I relied on ChatGPT's help heavily with setting up Wireguard. However, it's all because of a tiny error I didn't catch that took me forever to find. So I followed this guide: https://www.tech2geek.net/how-to-set-up-a-wireguard-vpn-on-linux-step-by-step-guide/ which was great. It would have worked out of the box no question, except for one thing.

I had tried to setup Wireguard a few days ago, and must have had a typo or something somewhere, because it wasn't working. In my wg0.conf file, I had put my server's private key, and saveConfig = True. Unbenownst to me, this meant that anytime I changed this file, it would reset it to the saved state when I reset wireguard, meaning it used the old private key that I got rid of in my quest to start from scratch with a new guide.

```
[Interface]
Address = ...
saveConfig = True :(
PostUp = ...
PostDown = ...
ListenPort = ...
PrivateKey = <server's private key>
```

Anyway, once I figured that out, it all worked great! I now can turn on a VPN on my phone or laptop, and access my server from anywhere in the world. It feels surreal and insanely cool. There's something about this that feels so much more grounded and meaningful than any random project I've done in CS. I'm sure I'd feel differently working on a real project in a job or internship, but this is a cool feeling.


---


<h3>Disk Mounting Intuition</h3>

Okay, so initially I was thinking of disk filesystems and mounting like the following. So I configure a disk with a certain filesystem. That dictates the metadata structure of how files work and move and how data should be cached and stored. Then, you can mount it to a folder in the OS filesystem, which opens up a portal you can hop through to now be inside the new disks filesystem. They're separate and cleanly connected. However, this is not quite right. I was researching virtualization through OS's like Proxmox, and I was wondering how disk mounting would work on each of the virtualized OS's. With the above intuition, each OS could mount the disk, and could add and remove data. However this would not work. When an OS mounts a disk, it assumes it has executive control over that disk. It can read and write to free blocks knowing that they can see the whole picture and will have no conflicts. When two OS's mount a disk, they both think they can do this, and will conflict read/write operations. It's similar to CPU cores, which have to communicate on the BUS in order to not conflict cache read/writes. So a better metaphor than the portal is that an OS kernel builds a control room opening to a disk when mounting, which it uses to observe and control the disk. Multiple control rooms would create chaos. There are "cluster filesystems" though which can allow for mounting across multiple kernels.

Something like Proxmox creates virtual disk partitions for each OS, and each OS formats its partition to a filesystem. There are ways to share data between VMs still though.











