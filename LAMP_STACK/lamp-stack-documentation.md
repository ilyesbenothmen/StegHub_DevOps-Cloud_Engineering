## WEB STACK IMPLEMENTATION (LAMP STACK) IN AWS
### Introduction:
LAMP is an acronym for Linux, Apache, MySQL, and PHP. It is a widely used open‑source software stack for building and hosting dynamic websites and web applications on the internet.
In the following steps, we are going to introduce the deployment of a website from scratch.
### Step0 preparing prerequisites:
To spin up a virtual server in the cloud, we use a free‑tier AWS account. We choose a basic EC2 instance of the t3.micro family running Ubuntu 24.04. After that, we need to download the private key file; in this lab, it is called devops-lab1.pem.

![alt](images/2.png)

After that, note the instance’s public DNS name.
![alt](images/2-1.png)

to connect to the new created instance :

```bash
chmod 0400 .ssh/devops-lab1.pem
ssh -i .ssh/devops-lab1.pem ubuntu@ec2-98-93-227-71.compute-1.amazonaws.com
```
### Step1 install Apache and Update the Firewall
The first rule of thumb for any system administrator is to keep the operating system up to date.

>[!TIP]
>It is important to note that using the root account is not a good practice. Instead, you should use a sudo‑enabled user. In our case, ubuntu will be our privileged user.
```bash
sudo apt update
sudo apt install apache2
```
![alt](images/3.png)

To verify the status of the web server , run :
```bash
    sudo systemctl status apache2
```
![alt](images/4.png)
The service is up and running ,you can run the following command :
```bash
    curl http://localhost:80
```
![alt](images/5.png)
Right now, the service is up and reachable locally, but what if we want to publish it? Easy: we need to create an inbound rule to allow traffic from the internet. In our case, I checked my public IP address and authorized only that address.
![alt](images/6.png)
Now, to check that our service is reachable, we first need to determine the public IP. The easiest way is to get it from the instance details, but personally I prefer to use the DNS name.
![alt](images/7.png)
> [!NOTE]
> There is a reliable way to get the public IP address from the EC2 instance metadata service.
![alt](images/9.png)

