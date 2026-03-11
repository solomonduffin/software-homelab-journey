# Storage

Okay so I'm struggling with how much detail I should go into here. The problem is I'm only understanding certain things at a surface level and I don't care to necessarily go so much deeper right now. But at a later date I would maybe come back to this section and fill in details, which maybe defeats the purpose of the logs? I'm not sure. I think this is for me and my own understanding and storage of concepts to come back to so that's probably most useful for myself.

**Great resource for understanding + command reference:**<br>
https://blog.victormendonca.com/2020/11/03/zfs-for-dummies/

**Official terms and definitions for a bunch of this:**<br>
https://docs.oracle.com/cd/E18752_01/html/819-5461/ftyue.html


### Volume Managers, ZFS, Pools, LVMs
Anyway, I'm trying to understand how ZFS pools, or storage pools work. And then how Datasets are on top of that. And then the question is, is that a Proxmox concept, a ZFS concept, or a general concept? Does it transcend the layer I'm working in and is relevant in many layers of storage?

So I think it does. Basically a pool is a connection of multiple storage drives, which can now be managed together as a single drive. But it's different from say RAID which actually combines the disks logically for redundancy. So pools don't provide redundancy necessarily, they just allow you to "combine" storage into one controllable and mountable unit, essentially it's just virtual storage. The systems that create pools, or logical volumes, are called Volume Managers, and each have their own system and features.

So ZFS is a volume manager (zpool is the name of the volume manager aspect) that can both apply a RAID configuration over a set of physical drives, and then manage the "single" volume. But, it's also a filesystem, which is what makes ZFS unique compared to some other volume managers. It also manages the metadata of the disk, directories, files, permissions, snapshots, etc.. It decides where to put things and how to maintain parity using the RAID scheme. 
So it replaces the common Linux stack of mdadm + LVM + ext4.

The term Pool specifically is the volume managed by ZFS. A logical volume (LV) is the term for the virtual storage managed by LVM. LVM is not a filesystem, it's just a volume manager. It can sit on top of a RAID array, or a normal drive, and does not need to understand how that block of storage functions, it just manages it. Then something like ext4 comes in to organize how files will be stored on the drive, and actively handle directories and navigation and such.

https://en.wikipedia.org/wiki/Logical_volume_management

<img width="1800" height="643" alt="image" src="https://github.com/user-attachments/assets/77fe3922-52ba-400e-8976-5ddfe8de569e" />




### Datasets
Datasets are another ZFS specific term. They're like separated directories in the pool. NOT partitions. They can have their own settings, sharing settings, mount points, snapshots, etc.. They share free space from the pool dynamically. They're like sub-filesystems. They can be set up to hold any category of things. 

So a zpool will have multiple **datasets**, and then also **Volumes (zvol)** which are blocks holding things like VMs, **snapshots** of datasets - readonly, and **clones** which are writable copies of snapshots.

What's nice about these datasets (specifically the filesystem/directory kind) is you can mount them into multiple containers at once (technically I think any filesystem can as well, so this is more about containers than datasets, but this is part of how I'll use datasets). They can't be mounted into multiple VMs because that would still break. But containers are run by the same Linux kernel so it can be shared. EXCEPT there does seem to be a way to mount into a VM using something called virtiofs, which is a middleman that sets up the VM as a sort of guest host.

#### The following is directly from https://avidandrew.com/understanding-zfs-datasets.html:
<details><summary>Basic but good summary of what ZFS datasets are</summary>

  ZFS datasets are a powerful and flexible organizational tool that let you easily and quickly structure your data, monitor size over time, and take backups.

A traditional filesystem like ext4 runs inside one disk partition. This is a simple structure, but inflexible if the growth of your data changes over time and you later want to rearrange your data. For example, imagine you created 2 partitions, a 10GB one for vegetables and a 5GB one for fruit. Perhaps you initially expect your vegetable data to grow much more quickly, but what if the reverse happens? You may end up with 5GB of fruit data, and now the partition has to be resized in order to make room. While this operation is possible, it will probably involve moving other data on the disk somewhere else and require the fruit partition to be unmounted temporarily. Using ZFS datasets, you can dynamically move the space where it is needed, no resizing or shuffling of data required.

Think of a zpool as a collection of dynamic filesystems (aka datasets) that can be nested within each other, resized, snapshotted, etc. For example, a zpool of food could look like this:
<img width="1000" height="320" alt="image" src="https://github.com/user-attachments/assets/496cf3e8-42c8-4e05-be76-a90bc011847c" />
If you wanted to manage your apples, you can cd /food/fruit/apples and similar for each of the other foods. If you later decide that tomatoes are really a fruit, you can move them simply with zfs rename food/vegetables/tomatoes food/fruit/tomatoes.

#### How do you configure a dataset?

Each dataset has a number of properties that you can get or set on it using zfs get and zfs set respectively. Some examples of what you can do with properties:

- Compression: Save space on the data stored in your dataset by enabling compression; there are a number of algorithms to choose from, and you can recursively enable compression on child datasets by simply setting it on the parent dataset. Note that changing compression settings doesn't affect existing data in the dataset, only new data written from then on. For an example, let's enable compression for all vegetable data:

<img width="1000" height="320" alt="image" src="https://github.com/user-attachments/assets/1efe7342-27f3-4087-ba07-51db4e8c610b" />

- Encryption: You can enable ZFS encryption on a dataset easily using a password
- Quotas: If you want to prevent a particular dataset from growing larger than a certain size, you can use a quota.
- Mountpoint: You can configure where a dataset is mounted, even in a directory completely unrelated to other datasets in the pool, using zfs set mountpoint=/path/to/the/new/mountpoint the/dataset/name.
- Custom Information: You can even store custom information about a dataset in User Properties. For example, to store the color of the apples:

<img width="1650" height="176" alt="image" src="https://github.com/user-attachments/assets/7e107328-e911-40b4-8bbd-67e3855ed2d5" />

 #### How do you backup datasets?

Even more powerful than any of the above features is ZFS's ability to take, send, and delete snapshots. While you can do this directly with ZFS commands, I highly recommend the combination of sanoid and syncoid to fully automate this process.
</details>


<img width="700" height="421" alt="image" src="https://github.com/user-attachments/assets/f210994e-cc92-4393-8db9-3e1306def1c1" />

