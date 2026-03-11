So I just finished building my home server. It's definitely a rabbit hole. Once you get one thing going, there're bottlenecks to fix, and so many possible paths to go down.

I decided on using Debian with CLI for now. I want to have the flexibility to do lots of things without too much abstraction like I would have if I used something like OpenMediaLibrary, TrueNAS, or something similar.

Server Part List:
- CPU: i3-12100
- Motherboard: Asus PRIME B760M-A AX Micro ATX
- HDDs: Seagate IronWolf NAS 6TB - x3
- SDD: Samsung 990 EVO 1TB
- RAM: Corsair Vengeance 32GB (2x 16GB)
- PSU: MSI A650GLS
- Case: Fractal Design Define R5
- 900VA UPS

Home Networking & Ethernet:
So I built the server but I need to wire it to ethernet. The motherboard I got supports 2.5Gbps, and I'd love to max that out. But, I also have to get my server connected. The motherboard works over wifi, but I want it to be isolated somewhere in the house connected to ethernet for sure. So I've been learning about home networks. 

We currently have Google Fiber, at I think 3Gpbs speed. There's a box in the laundry room that Google installed that converts the light signal from the Optic cables into ethernet signal. That links into the router which sends the connection wirelessly over the house. We also have some WiFi meshes which boost the signal.

The problem is, how do I extend the ethernet connection from the router around the house? Or at least to my server box which I need in a place far from the router? So there are a number of ways to do this, but this house was built with ethernet in the walls. It is Cat 5e, which only supports 1Gbps speeds. But it's a start for sure. Basically there's a bunch of unused wall jacks that all come together at a patch panel in our basement. This used to be used with our old wifi, but no one living here knew what it was or why it mattered, and we never used ethernet with any devices. Maybe some were hardwired by an electrician but we never utilized it ourselves. So now, the Google Fiber isn't using that at all.

  Possible Solutions:
  1. Connect the router to the patch panel
      This would allow me to use any wall jack in the house and get 1Gbps speeds. Not maxed out speeds, but great for convenience.
      I still need to figure out how to do this but I think it's pretty simple as long as the old wires and everything are still        working.
  2. Use a Powerline Adapter.
      You get 2 of these, then plug one in next to the router, and the other next to the server. Ethernet cable connects between        adapter->router, and adapter->server. This just gives a strong wireless signal directly from one adapter to the other.            These supposedly will give pretty good speeds, maybe 2.5/Gbps. But it's wireless and I'm not sure about the range.
  3. Just use wifi
     So I can always just use the wifi, but that's no fun. Plus, I want to see if I can actually access and use the speeds we're       paying for, just to learn how to optimize that in general.


Next Ideas/Steps:
- I need to setup SSH so I can get in from my laptop. I'll need to learn basic security practices here to make sure ports are        shored up.
- Configure RAID 5. Find a good self-hosted app for keeping track of my data, health of my drives, partitions, etc.. through         maybe a web server.
- Find a good external hard drive that I can use to backup important information every so often. I want this to be a homelab that    can be messed with for sure, but I also want to store and host data. I'm hoping that those two things don't clash too hard.
- Then I can start doing a bunch of stuff. I want to learn Docker containers, I want to host a website, do home automation, host     media servers, etc... And just generally become comfortable with the Linux environment and a headless OS.
