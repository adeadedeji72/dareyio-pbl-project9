# **INSTALL AND CONFIGURE JENKINS SERVER** #

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.

In this project we are going to start automating part of our routine tasks with a free and open source automation server – Jenkins. It is one of the mostl popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named "Hudson".

In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/yourname/tooling will be automatically be updated to the Tooling Website.

## **Task** ##
Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how your updated architecture will look like upon competion of this project:

![](add_jenkins.png)


## **INSTALL AND CONFIGURE JENKINS SERVER** ##

### **Step 1** – Install Jenkins server ###
Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

Install JDK (since Jenkins is a Java-based application)
~~~
sudo apt update
sudo apt install default-jdk-headless
~~~
Install Jenkins
~~~
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
~~~

Check Jenkins status
~~~
sudo systemctl status jenkins
~~~

Open port 8080 for Jenkins in the Inbound rule for the Instance

![](open-8080.jpg)

Access the newly installed Jenkins from
~~~
http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
~~~

Obtain the required password form
~~~
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
~~~
Copy and paste to the browser

![](jenkins-login.jpg)

![](jenkins-initialpass.jpg)

![](jenkins-loggedin.jpg)

Select **Install Suggested Plugins**

Once plugins installation is done – create an admin user and you will get your Jenkins server address.

![](jenkins-admin.jpg)

The installation is completed!

![](jenkins-ready.jpg)

### **Step 2** – Configure Jenkins to retrieve source codes from GitHub using Webhooks ###

In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable Webhook in the Github settings

![](webhook-setting.gif)

If you encounter an error about the webhook not reachable, **allow ICMP traffic on Inbound traffic on the Jenkins EC2 instance.**

2. Go to Jenkins web console, click "New Item" and create a "Freestyle project"

![](new-jenkins-prj.jpg)

3. To connect your GitHub repository, you will need to provide its URL, copy from the repository itself.

4. In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![](jenkins-repo-login.jpg)

5. Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under

![](manual-build.jpg)

Click "Configure" your job/project and add these two configurations
Configure triggering the job from GitHub webhook:

![](jenkins_trigger.png)

![](archive_artifacts.gif)

Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

![](auto-build.jpg)

We have configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstream) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally
~~~
ls /var/lib/jenkins/jobs/<project_name>/builds/<build_number>/archive/
~~~


## **CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH** ##

### **Step 3** – Configure Jenkins to copy files to NFS server via SSH ###
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

1. Install "Publish Over SSH" plugin.
On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it

![](jenkins-publish-over-ssh.jpg)

2. Configure the job/project to copy artifacts over to NFS server.
On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

    A. Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
    B. Arbitrary name
    C. Hostname – can be private IP address of your NFS server
    D. Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
    E. Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

![](jenkins-push-over-ssh.jpg)

Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"

![image](build-over-ssh.jpg)

![](bos-source-file.jpg)

Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

![](console-output.jpg)

If you are getting error about **'permission denied'** in the Console Output, check and set to ownership and permissions on the /mnt directory as
~~~
sudo chown nobody:nobody /mnt
sudo chmod -R 777 /mnt
~~~
To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file
~~~
cat /mnt/apps/README.md
~~~
If you see the changes you had previously made in your GitHub – the job works as expected.
