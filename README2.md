  ### Add HardDrive Disk on Proxmox 4x/5x

**1ºstep**
Select correct **Proxmox Node** and click on **Disks**

![](https://www.hostfav.com/blog/wp-content/uploads/2017/08/Disks.jpg)

You Can See New Physical Hard Drive Is Showing **/Dev/Sdb**

**2ºstep** 
Open Proxmox VE Nodes ***Shell***

![](https://www.hostfav.com/blog/wp-content/uploads/2017/08/Disks.jpg)

**step 3º** (Creat Partition and Volume Group Name)

**step 3º.a** (Creat Partition)

Use **fdisk** command to create partition

    root@pve01:~# fdisk /dev/sdb
    Welcome to fdisk (util-linux 2.29.2).
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.
    
    Command (m for help):
Create new partition type **n** and hit enter
*n = new partition!*

    Command (m for help): n
    Partition type
     p primary (0 primary, 0 extended, 4 free)
     e extended (container for logical partitions)

Select partition type **p** for Primary and select default value for **Partition Number**, **First Sector** and **Last Sector**.

        Partition number (1-4, default 1):
    First sector (2048-16777215, default 2048):
    Last sector, +sectors or +size{K,M,G,T,P} (2048-16777215, default 16777215):
    
    Created a new partition 1 of type 'Linux' and of size 8 GiB.

Save partition – type **w** and hit enter.

*w = write partition!*

    Now new drive is ready. Using following commands we are going to create LVM Volume
  
 **step 3º.b** (Creat Volume)

Create Physical Volume

    root@pve01:~# pvcreate /dev/sdb1
     Physical volume "/dev/sdb1" successfully created.
     
Create Volume Group *

    root@pve01:~# vgcreate newdrive /dev/sdb1
    Volume group "newdrive" successfully created
*newdrive = exemple ! You can use something like HDD1,etc.*

 **step 4º** (Assign the storage to ProxMox on Logical Volume Manager)
 
Go to **Datacenter –> Storage –> LVM**
![Datacenter –> Storage –> LVM](https://www.hostfav.com/blog/wp-content/uploads/2017/08/addlvm.jpg)

Give a new name for the storage, select correct volume group from drop down and click on **Add.**

![](https://www.hostfav.com/blog/wp-content/uploads/2017/08/selectvg.jpg)
*newdrive = exemple ! You can use something like HDD1,etc.See **step 3º.b** to remmeber wich you have selected.

#### New drive has been added to Proxmox Successfully.
![enter image description here](https://www.hostfav.com/blog/wp-content/uploads/2017/08/ndrive.jpg)
