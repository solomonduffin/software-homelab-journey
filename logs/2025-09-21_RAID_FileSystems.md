<h3>RAID:</h3>

I have 3 6TB NAS Hard Drives. This currently only allows for a few RAID configurations. I think optimally I'd use RAID 10, which balances speed and redundancy, but that requires 4. I could also use RAID 1 in a three-way mirror, which also gives 2 drive redundancy, but that leaves me 6TB of storage out of the 18TB. For now, I decided to use RAID 5. Only one drive failure redundancy, but 12TB of usable storage. Also, because I'm using Ironwolf NAS drives, they have a lower chance of failure during rebuild which is great.

Of course, this isn't data safety, and I'll need another one or two external drives for data I care about. For now though, I'm just gonna go for this.

<h4>Thoughts about spending too much time on decisions</h4>
An early trap I can see myself falling into is treating this whole thing too seriously. I have a tendency to fret about options and do WAY too much research into each decision. This is good for learning, but ultimately shallow learning. I think it's good to be aware of the options, research pros and cons, and then make a decision and go for it. That's the way to learn and go deep. If I instead spend an hour on each decision, I'm inching forward very slowly and not getting down to the nitty grit of specific stacks of hardware and software that I can mess around with. And because this is a personal device built for fun and education, as opposed to some business or job I'm in, I can afford to mess up. I can rebuild my RAID array completely from the future multiple times if I want to, and I will as I expand my drive array.

<h4>RAID 5 specifics and File Systems</h4>
So I built my array on RAID 5 using mdadm. This took about 12 hours, maybe a bit less, and now it's a finished RAID array.

I had 3 drives, /dev/sda, /dev/sdb, /dev/sdc, and those have been "combined" into a single drive in Linux's eyes into /dev/md0. 

But it needs a filesystem, which places a structure for metadata and directories on top of the drive. There are some options here, Ext4, XFS, Btrfs, among others. I went with Ext4 since it doesn't matter too much for my system.

Once it's setup, you have to mount it to a file somewhere. If it's not mounted anywhere, it can't be navigated into. It exists, but there is no door/portal into it. You can mount a drive to multiple files too which is cool. It won't stay mounted after server reset though unless you edit it into `/etc/fstab`.
