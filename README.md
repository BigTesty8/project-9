# TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS (project9)
#### TASKS
-----

Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

Here is how your updated architecture will look like upon competition of this project:

![](https://github.com/BigTesty8/project-9/assets/137091610/6fed6f04-7f49-4fdf-8d06-fc67ef6afb31)

#### Step 1 – Install Jenkins server
First off you need to setup your ubuntu server and name it JENKINS SERVER and ssh into it and on your terminal run the following commands to install jenkins and start it up.
#### Install JDK
- 'sudo apt update'
- 'sudo apt install default-jdk-headless'
#### Install jenkins
- 'wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins'
#### To make sure jenkin is running run
- 'sudo systemctl status jenkins'
By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group and then from your browser access your jenkins with your jenkins server public IP address with the :8080 attached at the end.
#### Jenkins account setup
Input username and password,password can be found by running this on your terminal 'sudo cat /var/lib/jenkins/secrets/initialAdminPassword' you can now install suggested plugins and startup jenkins you should see a dashboard like mine below:

![](https://github.com/BigTesty8/project-9/assets/137091610/e346686f-bb45-4bd9-bc52-c3d048d72a51)

#### Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks
Go to the git repository /settings/select webhook/select add and copy your jenkins url and paste in webhook with /git-hub webhook attached/

![](https://github.com/BigTesty8/project-9/assets/137091610/fb8cb310-8874-4ca4-bd64-1955d2c514a0)

In jenkins locate the source code management and select /GIT/
Go to Code in git repository and copy your repo url and paste,also create a credential and save.
Now select build now,go to the console output to check if code was a success!

![](https://github.com/BigTesty8/project-9/assets/137091610/20965dee-57b6-428e-97e0-d488cb508043)

#### But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

Click “Configure” your job/project and add these two configurations
- Configure triggering the job from GitHub webhook:
- Configure “Post-build Actions” to archive all the files – files resulted from a build are called “artifacts”.
- 
![](https://github.com/BigTesty8/project-9/assets/137091610/9b466b4d-0d52-4d92-8367-c5e2b6ee1b59)

- Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.
You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.
By default, the artifacts are stored on Jenkins server locally 'ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/'

#### Step 3 – Configure Jenkins to copy files to NFS server via SSH
- Install “Publish Over SSH” plugin.
- On main dashboard select “Manage Jenkins” and choose “Manage Plugins” menu item,
- On “Available” tab search for “Publish Over SSH” plugin and install it
- On main dashboard select “Manage Jenkins” and choose “Configure System” menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:
- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
Arbitrary name
- Hostname – can be private IP address of your NFS server
- Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
- 
![](https://github.com/BigTesty8/project-9/assets/137091610/87263402-046c-450c-ad4f-a1eb9686325e)

Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.
- Save the configuration, open your Jenkins job/project configuration page and add another one “Post-build Action”
- Configure it to send all files produced by the build into our previously define remote directory. In our case we want to copy all files and directories – so we use **.
If you want to apply some particular pattern to define which files to send – use this syntax.

- Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.
Webhook will trigger a new job and in the “Console Output” of the job you will find something like this:

![](https://github.com/BigTesty8/project-9/assets/137091610/3783b145-5b55-4f15-ab4e-ba72cf1b4465)

To make sure that the files in /mnt/apps have been updated – connect via SSH/Putty to your NFS server and check README.MD file

cat /mnt/apps/README.md
![](https://github.com/BigTesty8/project-9/assets/137091610/5bdeba25-df9c-4266-943c-3cb005bef96b)

