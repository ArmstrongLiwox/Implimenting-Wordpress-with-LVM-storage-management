# Implimenting-Wordpress-with-LVM-storage-management - Armstrong

## Implimenting Wordpress website with LVM storage management

Three-tier Architecture

![3 tier architecture](<images/3 tier architecture.jpg>)

> The three layers are :
    1. The presentation layer which is the user interface.
    2. The business layer which is the backend.
    3. The data layer which is the computer data storage.

## Step 1. Prepare a web server



### 1. Launch instance and Create 3 volumes

> Launch an ec2 instance that will serve as a web server

![launch ec2](<images/launch ec2.jpg>)

> EBS means Elastic Bolck Volume

> Create 3 volumes , 10Gb each.
![create volume](<images/create volume.jpg>)
![volume](images/volume.jpg)
![volumes created](<images/volumes created.jpg>)
> 

### 2. Attach all three volumes the web server Ec2 instance
![attach volumes](<images/attach volume.jpg>)

### 3. Open up the Linux terminal to begin configuration

![logged in](<images/loged in.jpg>)
### 4. Use lsblk

```
lsblk
```
![lsblk](images/lsblk.jpg)

### 5. Use df -h
> to see all mount devices

```
df -h
```
![df -h](<images/df -h.jpg>)
### 6. Use gdisk
> to create single partition on each of the 3 disks
```
gdisk
```
![gdisk](images/gdisk.jpg)

```
sudo gdisk /dev/xvdf
```

![8e00](images/8e00.jpg)
![sudo gdisk](<images/sudo gdisk.jpg>)
![w](images/w.jpg)
![y](images/y.jpg)
![successful](<images/xvdf successful.jpg>)

### 7. use lsblk to view the newly configured partition

```
lsblk
```

![lsblk3](<images/lsblk 3.jpg>)

### 8. Install lvm2

```
sudo yum install lvm2 -y
```
![lvm2](images/lvm2.jpg)
```
sudo lvmdiskscan
```
![lvmdiskscan](images/lvmdiskscan.jpg)


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
![pvs](images/pvs.jpg)

### 9. Verify that your physical volume has been created successfully.

```
sudo pvs
```
![pvs](images/pvs.jpg)

### 10. Use vcreateto add all three PV to a volume grounp named VG webdata-vg

```
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
```
![vgs](images/vgs.jpg)

### Verify that the VG was creared successlly

```
sudo vgs
```
![vgs](images/vgs.jpg)

### 11. Create 2 logicall volumes

```
sudo lvcreate -n apps-lv -L 14G webdata-vg
```
```
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
![lvs](images/lvs.jpg)

### 12. Verify the logical volume has been created.

```
sudo lvs
```
![lvs](images/lvs.jpg)

### 13. Verify entire settup

```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
```
![display volumes](images/display.jpg)

```
sudo lsblk 
```
![lsblk 4](<images/lsblk 4.jpg>)

![lvs vgs pvs](<images/lvs vgs pvs.jpg>)

### 14. use mkfs.ext4 to format the logical volumes with ext4 filesystem

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
```
```
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
![mkfs](images/mkfs.jpg)

### 15. Create /var/www/html directory to store web files

```
sudo mkdir -p /var/www/html
```

![mkdir www](<images/mkdir www.jpg>)

### 16. Create to store backup of log data

```
sudo mkdir -p /home/recovery/logs
```

### 17. Mount /var/www/html on apps-lv logical volume

```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
![mount](images/mount.jpg)

### 18. Use rsync utility to backup all the files in the log directory /var/log into home/directory/logs

> this is required before mounting the file systems

```
sudo rsync -av /var/log/. /home/recovery/logs/
```
![rsync](images/rsync.jpg)

![ls -l var](<images/ls var.jpg>)

### 19. Mount /var/log on logs-lv logical volume

```
sudo mount /dev/webdata-vg/logs-lv /var/log
```
![mount](images/mount.jpg)


### 20. Restore files back into /var/log directory

```
sudo rsync -av /home/recovery/logs/log/. /var/log
```

![restore](images/restore.jpg)

### 21. Update file /etc/fstab so that the mount configuration will persist after restart of the server.

```
sudo blkid
```
```
sudo vi /etc/fstab
```
```
UUID=0134cdfc-Ocfd-4018-8b2e-bdf6802e91c0   /var/www/html  ext4  defaults  0 0
UUID=eb227b42-1387-4813-a2c4-77cf95bc7c68   /var/log       ext4  defaults  0 0
```
![blkid](images/blkid.jpg)

![fstab](images/fstab.jpg)

### 22. Test the configuration and reload the daemon

```
sudo mount -a
```
```
sudo systemctl daemon-reload
```
![reload](images/reload.jpg)

### 23. Verify the setup by running df -h

```
df -h
```

![df -h check](<images/df -h check.jpg>)




# Installing wordpress and configuriing to use MYSQL Database

## Step 2. Prepare the Data base server

> Launch a second redhat EC2 instance that will have a role -DB server

![launch ec2 db server](<images/launch db ec2.jpg>)

![db volumes](<images/volumes db.jpg>)

![attach db](<images/attach db.jpg>)

![attach volumes db](<images/attach db volumes.jpg>)

![connect db](<images/connect db.jpg>)

> Repeat steps of webserver

![configure db](<images/connect db.jpg>)

![install lvm2](<images/install lvm2.jpg>)

![pv create](images/pvcreate.jpg)

![vgs db](<images/vgs db.jpg>)


> create db-lv (instead of apps-lv)

```
sudo lvcreate -n db-lv -L 20G vg-db
```
![db-lv](images/db-lv.jpg)

```
sudo mkfs.ext4 /dev/vg-db/db-lv
```
![mkfs.ext4](images/mkfs.ext4.jpg)


> mount it to /db (instead of var/www/html)

```
sudo mount /dev/vg-db/db-lv /db
```
![mount db](<images/mount db.jpg>)

```
sudo blkid
```
![blkid db](<images/blkid db.jpg>)

```
sudo vi /etc/fstab
```

![fstab db](<images/fstab db.jpg>)

```
sudo mount -a
```
```
sudo systemctl daemon-reload
```
![reload db](<images/reload db.jpg>)

![df -h db](<images/df -h db.jpg>)

## Step 3. Install wordpress on the webserver EC2

```
sudo yum update -y
```

### 1. Update repository

```
sudo yum -y update
```
![yum update](<images/yum update.jpg>)

![update complete](<images/update complete.jpg>)

### 2. Install wget, Apache and its dependencies

```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json 
```
 ![wget](images/wget.jpg)


### 3. Start Apache

```
sudo systemctl enable httpd
```
```
sudo systemctl start httpd
```
![start apache](<images/start apache.jpg>)


### 4. Install Apache and its dependencies

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```
```
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
```
```
sudo yum module list php
```
![list](images/list.jpg)

```
sudo yum module reset php
```
```
sudo yum module enable php:remi-7.4
```
```
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
```
![php install](<images/install php.jpg>)

![php version](<images/php version.jpg>)

```
sudo systemctl start php-fpm
```
```
sudo systemctl enable php-fpm
```
```
sudo systemctl status php-fpm
```
![status](<images/php status.jpg>)

```
sudo setsebool -P httpd_execmem 1
```
![setsebool](images/setsebool.jpg)

### 5. Restart Apache

```
sudo systemctl restart httpd
```
### 6. Download wordpress and copy to var/www/html/

```
mkdir wordpress
```
```
cd   wordpress
```
```
sudo wget http://wordpress.org/latest.tar.gz
```
![wordpress](<images/wordpress download.jpg>)

```
sudo tar xzvf latest.tar.gz
```
![extract](images/extract.jpg)

```
sudo rm -rf latest.tar.gz
```
```
sudo cp -R wp-config-sample.php wp-config.php
```
![config.php](images/config.php.jpg)

```
# cp wordpress/wp-config-sample.php wordpress/wp-config.php
```
```
sudo cp -R wordpress/. /var/www/html/
```
![copy wordpress](<images/copy wordpress.jpg>)

### 7. Configure SE linux policies
```
 sudo chown -R apache:apache /var/www/html/wordpress
 ```
 ```
 sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
 ```
 ```
 sudo setsebool -P httpd_can_network_connect=1
```

## Step 4. Install MySQl on the DB server EC2

```
sudo yum update
```
```
sudo yum install mysql-server
```
![mysql](<images/install mysql.jpg>)


> restart and enable the service

```
sudo systemctl restart mysqld
```
```
sudo systemctl enable mysqld
```

> verify that the service is up and running
```
sudo systemctl status mysqld
```

![mysql running](<images/mysql running.jpg>)

## Step 5. Configure DB to work with wordpress

```
sudo mysql_secure_installation
```
![secure](images/secure.jpg)

```
sudo mysql -u root -p
```
![mysql](images/mysql.jpg)

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`172.31.47.114` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'172.31.47.114';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

```
create database wordpress;
```
```
show databases
```
![database](images/database.jpg)

```
CREATE USER 'armstrong'@'172.31.47.114' IDENTIFIED WITH mysql_native_password BY 'happy';
```
```
GRANT ALL PRIVILEGES ON *.* TO 'armstrong'@'172.31.47.114' WITH GRANT OPTION;
```
```
FLUSH PRIVILEGES;
```
```
SHOW DATABASES;
```
```
select user, host from mysql.user;
```
```
exit
```

![database created](<images/database created.jpg>)


```
sudo vi /etc/my.cnf
```
```
[mysqld]
bind-address=0.0.0.0
```

```
[mysqld]
bind-address=172.31.47.114
```
![bind address](<images/bind address.jpg>)

```
sudo systemctl restart mysqld
```
> error encountered 
```
sudo systemctl stop mysqld
sudo yum remove mysql-server
sudo rm -rf /var/lib/mysql
sudo rm -rf /etc/my.cnf
sudo yum autoremove
rpm -qa | grep mysql
sudo userdel mysql
sudo groupdel mysql
sudo reboot
```
![status](images/status.jpg)

> edit sudo wp-config.php file in web server

```
sudo vi wp-config.php
```
![wp config](<images/wp config.jpg>)

![edit config](<images/edit config.jpg>)

```
sudo systemctl restart httpd
```
> disable the default page of apache

```
sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_backup
```


## Step 6. Configure wordpress to connet to remote database

> open Mysql port 3306 on DB server EC2

> in the inbound rule configure source as /32

![inbound rule](<images/inbound rule.jpg>)

> install mysql client and test that you can connect from your webserver to the Data Base server EC2

```
sudo mysql -h 172.31.42.213 -u myuser -p
```
![communicate database](<images/communicate database.jpg>)

## Step 7. Configure SE Linux policies

```
 sudo chown -R apache:apache /var/www/html/
 ```
 ```
 sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R
 ```
 ```
 sudo setsebool -P httpd_can_network_connect=1
```
![se linux policies](<images/SE Linux policies.jpg>)


```
sudo yum install mysql
```
```
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```
> verify you can execute SHOW DATABSES;

> change permissions and configurations so Apache could use wordpress.

> enable TCP port 80 in inbound rules configuration for Web server

> try to access from your browser link to the wordpress

```
http://<web-server-public-ip-address>/wordpress/
```
![wordpress working](<images/wordpress working.jpg>)

> fill out the DB credentials

![info](images/info.jpg)

![login](images/login.jpg)

![loginn](images/login1.jpg)

![success](images/success.jpg)

![website](images/website.jpg)

# SUCCESS

# CONGRATULATIONS
















