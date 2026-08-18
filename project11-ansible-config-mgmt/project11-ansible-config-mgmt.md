## Ansible Configuration Management (Automate Project 7 to 10)
### Introduction:
When designing infrastructure for a real-world enterprise environment, it is common practice to deploy what is known as a jump host (or bastion host). This server acts as a controlled entry point for administrative access to the internal infrastructure.

By centralizing external access through a dedicated, hardened system, we reduce the overall attack surface and limit the exposure of internal systems to potential threats.

The architecture below depicts the target architecture that we will implement.
 
![alt](images/architecture.png)

This lab consists of two parts:
1. Install and configure Ansible client to act as a Jump Server/Bastion Host
2. Create a simple ansible playbook to automate servers configuration
### Step 1 - install and Configure Ansible on EC2 Instance:

1. In previous labs, we used an AWS instance named steghub-jenkins. For the purpose of this lab, we will rename this instance to steghub-jenkins-ansible and configure it to act as our Jump Server (Bastion Host).

In the context of AWS, instance names are typically managed through Tags. Therefore, we will locate the Name tag associated with the instance in the AWS Management Console and update it accordingly.

![alt](images/1.png)

2. In the Github account we created a new repository and named it **ansible-config-mgt**.

![alt](images/2.png)

3. Install **Ansible** : 
[]  Install with pip
[x] Install with apt manager
In this case type the following system commands :
```bash
sudo apt update
sudo apt install ansible
```
Check that ansible is installed by running :
```bash
sudo ansible --version
```
![alt](images/3.png)

>[!TIP]
>



![alt](images/4.png)

![alt](images/5.png)

![alt](images/6.png)

![alt](images/7.png)
> [!NOTE]
> 
![alt](images/9.png)

### Step2 Installing Mysql

![alt](images/10.png)


![alt](images/11.png)

![alt](images/13.png)



### Step3 Installing PHP


![alt](images/14.png)

![alt](images/15.png)


>[!Note]
>

### Step4 Creating a virtual host for your website using Apache



![alt](images/19.png)





![alt](images/16.png)



![alt](images/20.png)


![alt](images/17.png)

### Step 5 Enable PHP on the website:
![alt](images/21.png)


![alt](images/18.png)

### Conclusion:

