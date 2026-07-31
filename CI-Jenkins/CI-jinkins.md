##  Tooling Website deployment automation with Continuous Integration .Introduction to Jenkins
### Introduction:

In the IT world, CI/CD is a core practice used to automate application builds and tests, enabling seamless software delivery from staging environments to production releases.
CI, or Continuous Integration, focuses on automatically validating code changes to ensure that the code developers are about to commit or merge does not introduce bugs or break existing functionality.
In this context, Jenkins—one of the most widely adopted open-source automation servers in the DevOps ecosystem—will be used to represent the CI component of the pipeline.
The artifacts produced by this CI pipeline will be consumed by downstream environments, such as shared storage and application servers, which rely on up‑to‑date, validated builds for deployment.

Below is the architecture that will be extended with a Continuous Integration (CI) pipeline.

1. Jenkins will be deployed on an Ubuntu-based EC2 instance and configured to orchestrate the CI workflow, including build, test, and validation stages.

2. The tooling source code hosted in GitHub will be automatically synchronized to the NFS share following each commit, ensuring that the latest changes are consistently available to downstream environments and processes


![alt](images/architecture.png)

### Step1 Install Jenkins Server:

Create an EC2 instance based on Ubuntu 24
![alt](images/0.png)
As a first step, update the Ubuntu package index to ensure the system has the latest metadata before installing any packages.
```bash
sudo apt update
```
![alt](images/1.png)
To obtain the latest installation guide, refer to the official Jenkins Debian repository at: **https://pkg.jenkins.io/debian-stable/**.

Install the default-jdk-headless package to provide a Java runtime suitable for running Jenkins.

```bash
sudo apt install default-jdk-headless
```

![alt](images/2.png)
first add the key to your system , Then add a Jenkins apt repository entry , Then install fontconfig
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
  
sudo apt-get install fontconfig

```
![alt](images/3.png)

![alt](images/4.png)
Check the java runtime version and then install jenkins
```bash
sudo apt-get install jenkins
```

![alt](images/5.png)
Make sure Jenkins is up and running

```bash
sudo systemctl status jenkins
```
![alt](images/6.png)

By default Jenkins run on port 8080 , for that reason add an inbound security rule in the EC2 Security Group

![alt](images/7.png)


Perform initial access to http://public-jenkins-ip:8080 and you will be prompted to the following page

![alt](images/8.png)

retrieve the password from /var/lib/jenkins/secrets/initialAdminPassword and introduce it to the input text  and press continue button.


![alt](images/9.png)

When prompted to select plugins during setup, choose the default option “Installer les plugins suggérés” (Install suggested plugins).  

>[!NOTE]
>The setup wizard automatically uses the language of your browser, which is why French is displayed in this example, but the selected language does not affect the configuration process.

Create the admin by filling the fields of the following form and continue 

![alt](images/10.png)
You will get this screen , in this lab we get http://jenkins-ip:80800/ ,just click on "save and continue"
![alt](images/11.png)

When the “Jenkins is ready” message appears, it indicates that the installation and initial configuration steps are complete and Jenkins is ready to accept connections and configure jobs !

![alt](images/12.png)



### Step2 Configure Jenkins to retrieve codes from GitHub using WebHooks:

Now, let’s introduce **webhooks** in a practical way. A webhook is essentially a callback mechanism that allows an external system, such as GitHub, to notify Jenkins when an event occurs (for example, a push to a repository). You can think of it as a small “service endpoint” exposed by Jenkins that GitHub calls automatically when changes happen.
In practice, this endpoint is identified by a URL, similar to how an ip:port combination identifies a process on a remote machine. When you configure a webhook in GitHub, you register this Jenkins URL so that GitHub knows exactly where to send event notifications.

In this part we will configure a simple Jenkins job . This job will be triggerd by GitHub webhook and will execute a 'build' tesk to retrieve codes from GiHub and store it locally on Jenkins server.




First of All go to Github and create a webhooks ,name it **http://public-jenkins:8080/github-webhook/** and submit.

![alt](images/14.png)

check after that , that the webhook is created as below.


![alt](images/15.png)
Now switch to jenkins and create anew **Item**
![alt](images/13.png)

Name it for example tooling_github and choose the free-style type
![alt](images/16.png)
Fill the form with the github repo and credentials and submit the job creation
![alt](images/17.png)
You will get the following screen 
![alt](images/18.png)
Click "Build Now" , if you have configured everything correctly ,the build will be successfully and you will see it under #1
![alt](images/19.png)
In our case, there is a problem, and the main challenge is to understand why it occurs. The “Waiting for next available executor” message shows that Jenkins cannot allocate an executor for the job, so this condition must be analyzed and troubleshot.

First, go to Nodes under Manage Jenkins to check for potential disk space allocation issues.
![alt](images/20.png)
Check the available disk space and attempt to increase the size allocated to /tmp.
![alt](images/21.png)

```bash
sudo systemctl edit tmp.mount
````
append this content at the end of the config file:
Options=mode=1777,strictatime,nosuid,nodev,size=2G

![alt](images/22.png)
then reload and remount /tmp
```bash
sudo mount -o remount,size=2G /tmp
```
![alt](images/23.png)
Now resize swap tp 2G and make the change permenant.

![alt](images/25.png)
![alt](images/26.png)

![alt](images/27.png)
Reviewing the console log indicates that the error remains uncorrected.
![alt](images/28.png)
The build triggered an error again, and the new /tmp partition size is not effective.
![alt](images/29.png)
It turns out the problem wasn’t where we first looked. After some research, we discovered that the branch name was configured as **master** instead of **main**. We’ll now update the branch reference to main to resolve the issue.
![alt](images/30.png)
After applying the correction, the job immediately turns green, confirming that the pipeline is now functioning as expected.
![alt](images/31.png)
Let us check the Build History
![alt](images/32.png)

Chech also the Console Output
![alt](images/33.png)

Now configure the job to trigger from the github webhook
![alt](images/34.png)
In the post-build   Actions choose to archive all the files - files resulted from a build are called artifacts. In my config I didn't configured files to archive and it will create a problem later that we will try to fix
![alt](images/35.png)
change the README.MD and push the changes to master branch.

![alt](images/36.png)
As expected a new build is triggerd but with error !

![alt](images/37.png)
First let us check the console Output
The error is : No Artifacts are configured for archiving ...
After Setting files to archive to "**" , the webhook is triggerd and the problem is fixed  
![alt](images/38.png)
![alt](images/39.png)

We have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as 'push' because the changes are being 'pushed' and files transfer is initiated by GitHub). There are also other methods: trigger one job (**downstream**) from another (**upstream**), poll GitHub periodically and others.

>[!NOTE]
>By default , the artifacts are stored on Jenkins server locally
```bash
ls /var/lib/jenkins/jobs/tooling_github/builds/#/archive
```
![alt](images/40.png)

### Step3 Configure Jenkins to copy files to NFS server via SSH

Now that the build artifacts are saved locally on the Jenkins server, the next step is to copy them to the /mnt/apps NFS share.

Jenkins is extensible through plugins, with an ecosystem of over 1,400 available plugins. In this section, you will install the Publish Over SSH plugin.

![alt](images/41.png)
After choosing theplugin and press on Install Button , jenkins will restart.
![alt](images/42.png)
Go to SSH Servers and fill the required field as below , test Configuration and save the configuration.

![alt](images/43.png)

Navigate to ssh build artifacts over SSH and configure the fields as detailed below:

![alt](images/44.png)

Commit a minor change to README.MD and push it to the remote repository to verify the automated webhook build. Check the Console Output.

![alt](images/45.png)

Go back to the tooling_github job and check the status and Builds

![alt](images/46.png)

Verify the modif directly on the file on our /mnt/shares.

![alt](images/47.png)

Create a file named testfile, add text content to it, and commit the changes to the respository.

![alt](images/48.png)

The console output displays the repository changes and confirms the file transfer to the shared directory.

![alt](images/49.png)

Now navigate to /mnt/apps and check the craetion of testfile as expected.

![alt](images/50.png)

### Conclusion:

At this stage, we have established a foundational Jenkins CI pipeline and integrated it with the existing components, as illustrated in the architecture for this lab.