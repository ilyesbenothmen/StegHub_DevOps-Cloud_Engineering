## Web Solution with wordpress
### Introduction:
WordPress is a well‑known CMS (Content Management System) that is widely used on the internet to publish news, create blogs, and build many other types of websites.

In this lab, we will deploy WordPress from scratch and configure it on two Amazon EC2 servers on AWS.
### Step1 - Prepare a Web Server 
1. **Launch the creation of a RHEL based EC2 instance to serve as a web server**

![alt](./images/0.png)
than download the pem file to connect later to the EC2 instance

![alt](./images/1.png)
After connecting get the storage layout with lsblk command
```bash
lsblk
```

![alt](./images/2.png)


![alt](./images/3.png)
In general Linux administrators use fdisk or gpated to partition disks
We still use fdisk for tiny and medium size storage VMs, but gparted is more suitable for huge storage and large number of partitions.

![alt](./images/4.png)
In our case initially we use fdisk just to render partitions that are build with a fresh deployed VM.

Now come the creation of storage disks in the same AZ (Availability Zone) as the VM itself, which is a required condition to map disks to host.


![alt](./images/6.png)
After that attach the volume to VM identified by its tag previously



![alt](./images/8.png)


repeat the same step 3 times after that run lsblk again to check that volumes appear at the os level with nvme1n1 ,nvme2n1 and nvme3n1


![alt](./images/9.png)


We can check partitions with fdisk -l

![alt](./images/10.png)



Since the VM is RHEL based we need to activate it to be allowed to  download packages and update from redhat.

![alt](./images/11.png)

![alt](./images/12.png)



After that use subscription-manager to register to main stream channel of redhat





2. **Create a partition for each volume ***

```bash
sudo gpared /dev/nvme1n1
(parted)mklabel gpt
(parted)mkpart primary ext4 1MiB 100%
(parted)quit
```

![alt](./images/13.png)

Repeat the creation of partition to have nvm1n1p1, nvme2n1p1 and nvme3n1p1

![alt](./images/14.png)
3. **Use pvcreate utility to maek each of 3 disks as Physical volume (PVs) to be use as Logical Volume (LV)**
To list all PV , use pvs utility

```bash
pvs
```
![alt](./images/15.png)

4. **Install lvm2 package**
```bah

sudo yum install lvm2
```
Run lvmdiskscan to verify available partitons.
```bah
sudo lvmdiskscan
```


![alt](./images/16.png)
5. **Create Volume group webdata-vg with vgcreate utility**

```bash
vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1 
```

![alt](./images/17.png)

6. **Create Logical Volume (LVs) with lvcreate utility**
```bash
lvcreate -n apps-lv -L 14G webdata-vg
lvcreate -n logs-lv -L 14G webdata-vg
```
![alt](./images/18.png)

Check the LV creattion with lvs utility

![alt](./images/19.png)
Check vg properties with vgdisplay command

```bash
sudo lvs
sudo vgdisplay
```
![alt](./images/20.png)
Now check the final disk layout with lsblk
7. **Use mkfs.ext4 to format the logical volumes with ext4 filesystem**

```bash
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

```

![alt](./images/21.png)

8. **Create mount-points for apps-lv and log-lv**
So create /var/www/html and /home/recovery/logs as intermediate folder
after that mount apps-lv to /var/www/html and save the content of /var/log to that temporary folder /home/recovery/logs with rsync tool to preserve files identity , owner and permissions 

![alt](./images/22.png)
Achieve the same for logs-lv
![alt](./images/23.png)

![alt](./images/24.png)
9. **Get UUID  for each partition to mount it automatically**
![alt](./images/25.png)
append new partitions to /etc/fstab
![alt](./images/26.png)

mount the newly added partitions and reload systemct daemon
![alt](./images/27.png)


![alt](./images/28.png)
Now we are done with EC2 web server

### Step2 - Prepare a Web Server
Create a new EC2 Server
Repeat the same procedure to create partitions and bind them to the EC2 oinstance
![alt](./images/29.png)
create dbdata-vg and corresponding LVs : db-lv and logs-lv
![alt](./images/31.png)
Check the created LVs with lvs command and vgdisplay
![alt](./images/32.png)
Again , have a last check to note partitions 
![alt](./images/33.png)

create ext4 filesystems on db-lv and logs-lv

![alt](./images/34.png)
Mount ext4 filesystems on /db and /var/log
![alt](./images/35.png)
restore the content of /home/recovery/logs on /var/log
![alt](./images/36.png)

Note UUID of partitions and update /etc/fstab
![alt](./images/37.png)

Reload Systemct daemon

![alt](./images/38.png)


### Step3 - Install Wordpress on the EC2 web server :

1. **Update the OS meta-data**

![alt](./images/39.png)
2. **install wget,Apache and dependencies**

```bash
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```
![alt](./images/40.png)

3. **Enable and start httpd**


![alt](./images/41.png)

4. **Install PHP and it's dependencies**
```bash
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-80.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd execmem 1
```

![alt](./images/42.png)

![alt](./images/43.png)
5. **Restart Apache httpd server**
After that create wordpress directory and download the last packages
![alt](./images/44.png)
6. **Download wordpress and copy wordpress**
![alt](./images/45.png)

7. **Configure Selinux Context**

![alt](./images/46.png)

### Step4 - Install MYSQL on your DB Server EC2
Update the OS meta-data and install mysql-server
![alt](./images/47.png)
For RHEL 10 it seems that the package mysql-server isn't available !
![alt](./images/48.png)
> [!NOTE]
>To get the exact name of the package use : sudo dnf provides mysql*-server
After that run the install of the exact version of mysql-server
```bash
sudo dnf install mysql8.4-server
```
![alt](./images/49.png)
Enable and start mysql-server by typing:
```bash
sudo systemctl status mysqld
sudo systemctl enable mysqld
sudo systemctl start mysqld
```
![alt](./images/50.png)
### Step5 - Configure DB to work with Wordpress

Connect to mysql locally by issuing:
```bash
sudo mysql
```
Create a database named wordpress and create a user name myuser@172.31.29.33 which is the ip address of the webserver 
```sql
create database wordpress;
create user `myuser`@`172.31.29.33` identified by 'mypass';
grant all on wordpress.* to 'myuser'@'172.31.29.33';
flush privileges;
```
![alt](./images/51.png)
Allow EC2 webserver to connect to the database server.

![alt](./images/52.png)

Now it is time to configure security group to allow port 3306 on the EC2 database server.

![alt](./images/53.png)
### Step6 - Configure Wordpress to connect to remote database
Query the version of mysql-client and issue:
```bash
sudo dnf install mysql8.4
```
![alt](./images/54.png)
Try to connect remotly to the database server with:

```bash
sudo mysql -u myuser -p -h 172.31.22.104
```
![alt](./images/55.png)

After obtaining the mysql prompt , query authorized database by show databases command.


![alt](./images/56.png)

Use your favorite browser and query http://public-ip/wordpress , it shouldn't work because I have missed to configure wp-config.php file. 

![alt](./images/57.png)

The result was expected , unless we didn't configred wp-config.php file
![alt](./images/58.png)
Just replace databse name, user , password and db host with correct values.
![alt](./images/59.png)
Now the call to the default page will redirect to installation page. 
Choose English as the language
![alt](./images/60.png)
Fill in the formula page with the corresponding value

![alt](./images/61.png)
Press the "Install WordPress" Button
![alt](./images/62.png)
and the success page should be displayed
Now try to reconnect againto the default page and enter login and password
![alt](./images/63.png)
The landing page is displayed and the application is running as expected
![alt](./images/64.png)

Finally it works fine !

### Conclusion:
In this lab, we manually installed and configured WordPress on our servers. As we progress through this bootcamp, we will see how Infrastructure as Code (IaC) can make deploying WordPress and other applications much easier and more automated.