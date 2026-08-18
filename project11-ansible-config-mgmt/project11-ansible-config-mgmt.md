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
>When installing Ansible with apt or yum, you get the latest Ansible version available in your distribution’s repositories, which is often older than the current upstream release. In contrast, installing Ansible with pip usually gives you the latest stable version published on PyPI, unless you explicitly pin a version.


4. Create a new Jenkins Item named ansible that archive the code change

- Create a freestyle project named **ansible**

![alt](images/4.png)
- Fill in the **Repository URL** and **Credentials**
![alt](images/5.png)
- Create a webhook by filling the **Payload URL** , for the moment "Disable SSL verification
![alt](images/6.png)
- Set the webhook to trigger ansible build
![alt](images/7.png)
- Configure a Post-build action that archive all files(**).
![alt](images/8.png)
> [!NOTE] pay attention to use (**) as Files to Archive not (*)
> 
You will see the ansible item checked in green after the first build integration.
![alt](images/9.png)

Let’s check the status of the ansible item; it is indicated by #1, showing that it completed successfully.

![alt](images/10.png)

By checking the Console Output, you can verify that the artifact was archived and that the build finished with **SUCCESS**.
![alt](images/11.png)
5. You can check /var/lib/jenkins/jobs/ansible/builds/1/archive
![alt](images/12.png)
In order to keep a fixed public IP address, we used an Elastic IP.
![alt](images/13.png)
### Step 2 - Prepare your development environment using Visual Studio Code:

1. To improve productivity, we use an integrated development environment

 (IDE). For this bootcamp, our IDE of choice is Visual Studio Code because it:

- Integrates seamlessly with GitHub.

- Works directly with our Debian WSL environment.

- Connects to the jump host over SSH.

- Lets us browse remote files graphically as if they were local.

- Provides an integrated terminal.

- Includes a built-in debugger and interpreter support.

- Allows us to invoke an AI chatbot from within the editor.



2. We need to allow the jenkins-ansible host to push code to GitHub over SSH. To do this, we add its SSH public key to our GitHub account under Settings → SSH and GPG keys.

![alt](images/14.png)
3. Clone the ansible-config-mgt.git to the jumphost VM with
```git
git clone git@github.com:ilyesbenothmen/ansible-config-mgt.git
```
As shown in the image below, we are connected to the jumphost shell through the integrated Terminal.
>[!Note]
>the README.md is from the remote repository on Github
![alt](images/15.png)

The bottom of the following image indicates that we are successfully connected to the jumphost over SSH

![alt](images/16.png)
### Step 3 - Begin Ansible Development :
1. Following GitFlow best practices, we classify branches by purpose: feature/, bugfix/, release/, hotfix/, and so on.


2. In this example, we are working on a new feature, so we create and use a feature branch:
```git
git checkout -b feature/prj11-ansible
git fetch
git checkout feature/prj11-ansible
```
At this stage we are working locally on our local repository on the newly created feature/prj11-ansible branch.

![alt](images/17.png)

3. Create a directory and name it playbooks 

4. Create a directory and name it inventory

5. Within playbooks directory , create the first playbook and name it common.yml

6. Winthin the inventory folder , create an inventory file for each environment :  dev ,prod,uat and staging.

![alt](images/18.png)


>[!Note]
>
fffffffffffffffffffffffffffffffffffffffff


![alt](images/19.png)





![alt](images/16.png)



![alt](images/20.png)


![alt](images/17.png)

### Step 5 Enable PHP on the website:
![alt](images/21.png)


![alt](images/18.png)

### Conclusion:

