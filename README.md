# Implimenting-Wordpress-with-LVM-storage-management

## Implimenting Wordpress website with LVM storage management

Three-tier Architecture

![3 tier architecture](<images/3 tier architecture.jpg>)

> The three layers are :
    1. The presentation layer which is the user interface.
    2. The business layer which is the backend.
    3. The data layer which is the computer data storage.

## Step 1. Prepare a web server



### 1. Prepare a web server

> Launch an ec2 instance that will serve as a web server

> EBS means Elastic Bolck Volume

> Create 3 volumes , 10Gb each.

> 

### 2. Attach all three volumes the web server Ec2 instance

### 3. Open up the Linux terminal to begin configuration

### 4. Use lsblk

```
lsblk
```

### 5. Use df -h

```
df -h
```

### 6. Use gdisk

```
gdisk
```
> 

```
sudo gdisk /dev/xvdf
```

### 7. use lsblk to view the newly configured partition

```
lsblk
```

### 8. Install lvm2

```
sudo yum install lvm2
```

```
sudo lvmdiskscan
```
> Run pvcreate to mark each of the three diska as physical volumes.

```
sudo pvcreate /dev/xvdf1
```
```
sudo pvcreate /dev/xvdg1
```
```
sudo pvcreate /dev/xvdh1
```
### 9. Verify that your physical volume has been created successfully.

```
sudo pvs
```
### 10. Use vcreateto add all three PV to a volume grounp named VG webdata-vg

```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
### Verify that the VG was creared successlly

```
sudo vgs
```

### 11. Create 2 logicall volumes

```
sudo lvcreate -n apps-lv -L 14G webdata-vg
```
```
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
### 12. Verify the logical volume has been created.

```
sudo lvs
```
### 13. Verify entire settup

```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
```
```
sudo lsblk 
```

### 14. use mkfs.ext4 to format the logical volumes with ext4 filesystem

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
```
```
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
### 15. Create /var/www/html directory to store web files

```
sudo mkdir -p /var/www/html
```

### 16. Create to store backup of log data

```
sudo mkdir -p /home/directory/logs
```

### 17. Mount /var/www/html on apps-lv logical volume

```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```

### 18. Use rsync utility to backup all the files in the log directory /var/log into home/directory/logs

> this is required before mounting the file systems

```
sudo rsync -av /var/log/. /home/recovery/logs/
```
### 19. Mount /var/log on logs-lv logical volume

```
sudo mount /dev/webdata-vg/logs-lv /var/log
```

### 20. Restore files back into /var/log directory

```
sudo rsync -av /home/recovery/logs/log/. /var/log
```
### 21. Update file /etc/fstab so that the mount configuration will persist after restart of the server.

```
sudo blkid
```
```
sudo vi /etc/fstab/
```
### 22. Test the configuration and reload the daemon

```
sudo mount -a
```
```
sudo systemctl daemon-reload
```

### 23. Verify the setup by running df -h

```
df -h
```















## Step 2. Prepare the Data base server
