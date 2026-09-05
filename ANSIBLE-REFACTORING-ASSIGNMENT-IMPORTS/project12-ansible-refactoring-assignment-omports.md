## Ansible Refactoring & Static Assignments (Imports and Roles)
### Introduction:

Refactoring is primarily about improving code readability and maintainability by leveraging the built-in capabilities of programming frameworks and tools. It can also extend to the design layer: well-known design patterns used by software developers are, in a sense, a form of design-level refactoring. Since Ansible is a configuration management language, applying refactoring practices is especially valuable for producing cleaner, more reusable, and easier-to-maintain automation code.
For better understanding or Ansible artifacts re-use read this article on[playbook-reuse](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse.html).
 
![alt](images/architecture.png)

This lab consists of two parts:
1. Ansible code Refactoring 
2. Running playbook against UAT environment
### Step 1 - Jenkins Job enhancement:

1. Go to ansible-jenkins server and create a new directory called ansible-config-artifact
![alt](images/1.png)

2. Change the permissions to /home/ubuntu/ansible-config-artifact so that Jenkins will save files there.

![alt](images/2.png)

3. Go to jenkins web console -> Manage Jenkins -> Manage Plugins on Available to search for **Copy Artifact** and install the plugin

![alt](images/3.png)


4. Create a freestyle project and name it  **save_artifacts**

![alt](images/6.png)


5. This project will be triggered by completion of the existing ansible project.


- you can keep only the 2 last build to preserve space
![alt](images/77.png)
- Configure a trigger that will be initiated after ansible project is build
![alt](images/8.png)
6. Create a buils step and choose **Copy artifacts from other project** , specify **ansible** as a source project and **/home/ubuntu/ansible-config-atifact** as a target directory
![alt](images/9.png)
7. Now it is time to test the pipeline setup , by making some changes to README.MD

![alt](images/10.png)
There is a problem of permission ,let us fix it. 
If we add ubuntu to jenkins group and restart jenkins service , hope that works!

![alt](images/11.png)

*****, you can verify that the artifact was archived and that the build finished with **SUCCESS**.
Let us make some change to README.MD and check again the Console Output 
![alt](images/12.png)
looks like the solution doens't worked !
![alt](images/13.png)
Why not jenkins will be a member of ubuntu group ! and restart jenkins by running:
```bash
sudo systemctl restart jenkins
```
![alt](images/14.png)

The change permission solution works fine !

![alt](images/15.png)

![alt](images/16.png)

### Step 2 - Refactor Ansible code by importing other playbooks into site.

We need to create a new branch named refactor
```git
git pull origin main
git checkout -b refactor
```
![alt](images/17.png)
1. site.yml will be the entrypoint into the entire infrastructure configuration.
2. Create a folder and name it static-assignments 
3. Move common.yml file into static-assignments folder
4. Insite site.yml file , import common.yml playbook
```yaml
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
![alt](images/18.png)

the structure of the project looks like the following:
![alt](images/181.png)
5. create a playbook under static-assignments and name it common-del.yml ,since wireshark is already installed this playbook will uninstall it !
![alt](images/19.png)

Update site.yml with :

```yaml
- hosts: all
- import_playbook: ../static-assignments/common-del.yml
```
commit the changes and issue a pull request

![alt](images/20.png)

![alt](images/21.png)

![alt](images/22.png)

![alt](images/23.png)

Check that changes are applied to ansible-config-artifact and first check which hosts are meant by common-del.yml playbook
![alt](images/24.png)
Run the playbook against dev servers and check the expected results
![alt](images/25.png)

Now we have learned how to use import_playbooks module and we have a ready solution to install/delete packages on multiple servers with just one command.

### Step 3 - Configure UAT Webservers with a role 'Webserver' :

Let us put aside our dev environment and create a fresh UAT (User Acceptance Test) environment

1. Launch 2 new instance named Web1-UAT and Web2-UAT running RHE8.
![alt](images/26.png)
2. Create a role inside ansible-config-mgt folder and move inside it
issue the command :
```bash
cd roles
ansible-galaxy init webserver
```

![alt](images/27.png)

Remove files ,tests and vars

![alt](images/29.png)

3. Update your inventory ansible-config-mgt/inventory/uat.yml with IP addresses of the 2 new UAT Web servers
4. create an ansible.cfg file in which set **roles_path** to roles , So ansible could know where to find configured roles.

![alt](images/31.png)
you can check the roles_path effective running configuration by running:
```bash
ansible-config dump |grep DEFAULT_ROLES_PATH
```
![alt](images/32.png)
5. Now it is time to configure our playbook to:
- install and configure apache
- Clone tooling website from Github
- Copy the code to /var/www/html 
- Start apache

Our main.yml playbook consist of the following tasks:

![alt](images/33.png)

### Step 4 - Reference 'Webserver' role
Within the static-assignments folder , create a new assignment for uat-webservers uat-webservers.yml.
![alt](images/34.png)
Now we need to refer our uat-webservers.yml role inside site.yml
We should have this in site.yml
![alt](images/35.png)

### Step 5 - Commit & Test

Now commit the changes to the refactor branhc and trigger a pull request to the remote main branch

![alt](images/37.png)

Check the changes are done and the pipeline works fine 
![alt](images/38.png)
To run the playbook against the uat environment with , we need either manually add the public key of github.com to known_hosts file of each OS or automate the task.
For the seek of this lab , we developed a playbook named ssh-key-known-add.yml  and we run it against uat environment

![alt](images/39.png)
We should have this in ssh-key-known-add.yml 
![alt](images/40.png)
In Git we need to authorize our hosts to download code by adding their public key into SSH and GPG keys
![alt](images/42.png)

![alt](images/41.png)
After that , run site.yml against uat.yml inventory

![alt](images/43.png)

![alt](images/44.png)

We need to see both of our UAT Web Servers configred and we can try to reach them from the browser

![alt](images/45.png)

![alt](images/46.png)

![alt](images/47.png)

### Conclusion:

In this lab, we used static assignment to refactor our Ansible automation code. In the next lab, we will use dynamic assignment to improve code reuse.


