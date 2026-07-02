# Linux RAID 5 Lab Note 
### Creating, Breaking, and Repairing a Redundant Storage Array


A hands-on lab simulating enterprise storage resilience using Linux loop devices. 

> 💡 **Problem-Solving Note:** I initially started this lab on a locked Google Chromebook, but system restrictions prevented the low-level operations. I pivoted to **Google Cloud Platform (GCP)**, which provided a secure, flexible, and unrestricted sandbox environment.


Here is a breakdown of how I built, intentionally broke, and successfully repaired a RAID 5 array using mdadm and loop devices. 

 

**Part 1: Preparing the Virtual Disks**

To simulate real hardware without buying physical drives, I created “fake” hard drives out of files using loop devices: 

**a. Create 3 empty 100MB disk images**

```bash
fallocate –l 100M disk1.img disk2.img disk3.img
```

**b. Attach the images to the system as loop devices**
```bash
sudo losetup -fP disk1.img
sudo losetup -fP disk2.img
sudo losetup -fP disk3.img
```

**c. Confirm the device names (usually loop0, loop1, loop2)**
```bash
losetup -a
```



**Part 2: Building the RAID5 Array** 

With the virtual disks ready, I combined them into a fault-tolerant RAID 5 configuration and mounted the filesystem: 

**a. Install the RAID tool** 

```bash
sudo apt update && sudo apt install mdadm –y
```

**b. Create the RAID 5 Array**

```bash
sudo mdadm  --create  --verbose /dev/md0 --level=5 –raid-devices=3 /dev/loop0 /dev/loop1 /dev/loop2
```

**c. Format the Array**

```bash
sudo mkfs.ext4  /dev/md0
```

**d. Create a folder to hold the files**

```bash
sudo mkdir –p /mnt/raid_lab
```

**f. Plug in the RAID to that folder**

```bash
sudo mount /dev/md0 /mnt/raid_lab
```

 

**Part 3: Testing and Resilience (The ‘Break’ Test)** 

RAID 5 can survive the loss of a single drive. I put this to the test by simulating a hard drive failure: 

**a. Create a secret file on the RAID**

```bash
echo “RAID is my secret data!” | sudo tee /mnt/raid_lab/secret.txt
```


**b. Simulate a disk failure**

```bash
sudo mdadm  /dev/md0  --fail /dev/loop1
```

**c. Verify the data still exists**

```bash
cat  /mnt/raid_lab/secret.txt
``` 

**d. Check the ‘injured’ status**

```bash
cat  /proc/mdstat
``` 

 

**Part 4: The Repair (Hot-Swap)**  

To restore the server to full health, I simulated a hot-swap by clearing out the failed “hardware” and re-introducing it as a fresh drive: 

**a. Remove the broken disk**

```bash
sudo mdadm  /dev/md0 --remove /dev/loop1
``` 

**b. Wipe the disk**

```bash
sudo wipefs –a /dev/loop1
``` 

**c. Add the new disk back to the array**

```bash
sudo mdadm  /dev/md0 --add /dev/loop1
``` 

**d. Final health check**

```bash
sudo mdadm --detail  /dev/md0
``` 

 
 

# Lessons Learned 

**Adaptability**: When local hardware restrictions stopped the project, cloud infrastructure provided an immediate, secure solution. 

**Storage Resilience**: Experienced firsthand how RAID 5 calculates parity mathematically to keep data alive and readable even during an active hardware failure. 

**System Administration**: Mastered core sysadmin tools, including mdadm, losetup, and filesystem mounting procedures. 


