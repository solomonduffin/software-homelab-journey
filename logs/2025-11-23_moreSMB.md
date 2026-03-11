So I've set up some awesome stuff so far, and learned a ton. Because it's just been working I haven't felt the need to do much in a while. I definitely have a lot more I want to do, but I also need to find out what more I can do. For now, I'm going to set up a simple fileshare between my laptop and phone for easy file transfer, which will be super helpful.

I'd like to set up a public file share. That would be super cool. Like something that the server is putting up onto the network for anyone in the network to access and pass around files. I think that'd be very useful. This might be super easy and not worthy of a log but we'll see.

It seems like it's pretty simple. Samba can do it, we just add this to `/etc/samba/smb.conf` after we create our directory.

```
[public]
   path = /mnt/tank/public
   browseable = yes
   public = yes
   guest ok = yes
   read only = no
   force user = nobody
```

But then, I want to broadcast it so it shows up in the network. This requires the WS-Discovery daemon, wsdd.

I had some issues starting it up. Samba works fine on an unprivileged container, but wsdd seems to have issues.

Okay wow. Nvm this has been a massive pain to set up for some reason. I need to come back with a clear head later.
