<h3>Ethernet Connection:</h3>
<p>
Okay. So, I've done a lot with trying to setup the ethernet connection for my server.
So first I connected my router to the wall jack in the laundry room. This lit up the connection on the panel downstairs, so it was definitely connected. Then, I had to go through a huge trial and error to find the wall jacks I wanted to get the server connected to the system. This turned out to be tricky, and I was unable to find the exact one I wanted.

Eventually, I moved the entire thing to another room where I did have a wall jack I could find and connect. But for some reason, this didn't work. I'm not sure if it has to do with the router not really sending signal through the system, but things were lighting up logically so I'm not 100% sure.

I ended up just moving the server into a small closet in my laundry room and moving other things around. This is the best case scenario to be honest because I got a direct 2.5Gbps connection from the router to my server. I'll need to drill some holes to make it look nice, but it's connected well.
</p>

<h3>Static IP assignment, and /etc/network/interfaces:</h3>

So next, I wanted to make a secure and easy connection between my laptop and the server. To do this, my server needs a static IP address. So first, my server's ethernet interface wasn't getting assigned an IPv4 address. I used 'sudo dhclient enp4s0' to assign it one. This address though is dynamically given, which means it can change over time. The router assigns it, but it may reassign it later. There are a few ways to force a static IP that doesn't change.

1. Assign IP through router settings
  I tried this route initially, but learned my Google Fiber router doesn't support it >:( So if I want more settings I need a third party router. But I might not need it. I can port forward which is all I need for now.
2. Assign IP in Debian network config
- https://idroot.us/debian-13-network-configuration/ 
- https://www.baeldung.com/linux/network-interface-configure 
<br>These are great resources for understanding some of this stuff. But Debian 13 has a few ways to do this config.
For now, I just used this "Traditional Method: /etc/network/interfaces Configuration". Which is pretty bare metal it seems. There are other methods that are more abstracted, but this is what I found first. Inside of /etc/network/interfaces lives all of the network interface behavior. There's the lo interface, which I think lets the server talk to itself even if nothing is connected? It's like a virtual server connection. localhost vibes. You can use it to ssh into the server from the server. Not 100% sure where this matters, but it's supposedly important.

Then there's the wifi interface, titled wlo1 on my machine. This was also autofilled, probably when I installed the distro and connected to wifi, because the wifi ssid name and passcode are in there in plaintext.

So here is where I made a new interface for the ethernet, which is enp4s0 on my machine.

<img width="1500" height="500" alt="216a943e-028c-453f-8741-56f51d21dc40" src="https://github.com/user-attachments/assets/91012b9b-5379-45ce-a8e1-7a2d32ffd2ea" />


<br>Some ChatGPT info about the IPv4 addresses:
<br>
>🔹 Private IP Ranges<br>
>Your 192.168.1.x address is part of the private IPv4 address space. These ranges are reserved for local networks only (they >never appear on the public internet):
>
>10.0.0.0 – 10.255.255.255 (10/8)
>172.16.0.0 – 172.31.255.255 (172.16/12)
>192.168.0.0 – 192.168.255.255 (192.168/16)
>
>So any home, office, or internal network can use addresses from those ranges.
>
>🔹 Why Your Router Uses 192.168.1.x<br>
>Most consumer routers default to:
>192.168.0.1 or 192.168.1.1 as the router/gateway
>and then hand out client IPs in that subnet (e.g. 192.168.1.100 – 192.168.1.254)

<br>That's why I chose 192.168.1.50, so it wouldn't hand out a conflicting IP address. So it shouldn't have any problems there.



<h3>ssh:</h3>

Okay, short section. So we want easy secure connection between my client and server. I generated a private/public keypair using

`ssh-keygen -t ed25519`

Then I stored the public key inside the servers \home\<server_username>\.ssh\authorized-keys. Now I can ssh in without entering a passcode, and I can disable passcode entry to only allow ssh key entry which is great. Now I can only login to my server locally, or from this laptop.

I also made a config file that lets me ssh in using `ssh debianlab` like this:

```
Host debianlab
    HostName 192.168.1.50
    User <server_username>
    IdentityFile C:\Users\<client_username>\.ssh\id_ed25519
```

<h3>Next Steps:</h3>

- Setup RAID for my hard drives
- Find a web server interface to see some of my server's stats if possible (temp, drives, etc..)
- Setup a VPN tunnel (Tailscale or Wireguard) to VPN into my server from anywhere



