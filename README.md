# Project-with-Continous-Build-And-Continous-Deployment

A simple project exploring the CI/CD process integrating Ansible, Docker, Git, Jenkins, EC2. We will deploy a Flask Application using the mentioned tools. Here any event/commit taken place in the github will be automatically deployed in the test server using jenkins.

## CI/CD Process

CI and CD stand for continuous integration and continuous delivery/continuous deployment. In very simple terms, CI is a modern software development practice in which incremental code changes are made frequently and reliably. Automated build-and-test steps triggered by CI ensure that code changes being merged into the repository are reliable. The code is then delivered quickly and seamlessly as a part of the CD process. In the software world, the CI/CD pipeline refers to the automation that enables incremental code changes from developers’ desktops to be delivered quickly and reliably to production

![image](https://user-images.githubusercontent.com/94472101/150872961-d4e7c96a-6edd-4191-9f5d-01a0815b02de.png)

## Architecture

## Working

We will build a docker image of Flask Application in the build server and push it to the Docker hub. Then we will deploy the container using this image in the Test server. For that we have 3 ec2 instances say Deployment, Build and Test.

 - Deployment: An ansible playbook is created to build a docker image of a flask application in a build server and deploying it in a container in a Test server.                  Here the playbook is run from this server.
 
 - Build: Ansible in the deployment server will connect the build machine and builds the docker image. It then pushes the pre-built image to Dockerhub.
 
 - Test : Ansible in the deployment machine will then contact the Test instance and a container is created using the pre-built image from Dockerhub

## Prerequisites

- Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html) in all the 3 instances.

- Install jenkins and git in Deployment machine.

   ```
      $ yum install git -y
      $ amazon-linux-extras install epel -y
      $ wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
      $ rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
      $ yum -y install jenkins
      $ systemctl start jenkins
      $ systemctl enable jenkins
  ```
 - GitHub Account
 - Docker Hub Account

## Provisioning

 #### Clone the repo

     ```
     git clone git@github.com:jebincvarghese/Project-with-Continous-Build-And-Continous-Deployment.git
     ```
     
 #### Configuring jenkins
 
  1. Login to the jenkins using the url "http://ip:8080" and update the password
  
      ![jenkauth](https://user-images.githubusercontent.com/94472101/150881738-213b2b65-e995-4a2b-b379-d98522fe9e51.png)

  2. Install the required plugins and create admin user
  
      ![strt](https://user-images.githubusercontent.com/94472101/150882091-f5a86752-48a9-45ec-a764-b702a8b673a5.png)
      ![start1](https://user-images.githubusercontent.com/94472101/150882468-b750eef3-2aa3-4222-bcf0-5053fcec4d8b.png)

  3. Install the ansible from the 
     Dashboard >> manage jenkins >> manage plugins >> select ansible from available resource >> install without restart >> Restart Jenkins when installation is        complete and no jobs are running.
     
     ![jenk](https://user-images.githubusercontent.com/94472101/150884637-bcf6586c-a64a-4b42-a4dd-75b5076fddb5.png)
     Now jenkins can communicate with ansible.
 
  4. Configure Ansible plugin Dashboard >> manage jenkins >> Global tool configuration >> Add Ansible >> enter proper location of ansible executable file (/bin/)      >> apply and save
     
     ![ansiblecon](https://user-images.githubusercontent.com/94472101/150885391-045af588-eb66-4683-9707-df7dd9f4999e.png)
     
  5. Creating a new job in jenkins
     Dashboard >> New item >> Freestyle project >> source code management - Git >> Enter Repository URL >> Apply and Save
     
     ![job](https://user-images.githubusercontent.com/94472101/150886408-b0818a79-5979-4283-a031-3b4ea411819d.png)
     
     ![cicd](https://user-images.githubusercontent.com/94472101/150886279-9c48c40d-742d-48d0-af36-f5a0cb9744e1.png)
     
   6. Select "GitHub hook trigger for GITScm polling" under the section "Build Triggers" in order to use webhook and GitHub
   
      ![buildtrigger](https://user-images.githubusercontent.com/94472101/150887800-9c86fe4e-6576-4ba7-a27a-b2d01d1968b3.png)

   7. Choose the option "Invoke Ansible Playbook" under "Build" section and enter the absolute path of the playbook and hosts file. Also check the option "Disable       the host SSH key check" from "Advanced".
     
      ![invok](https://user-images.githubusercontent.com/94472101/150888726-c7c6e000-91bd-4781-8448-925f24609d88.png)
      ![adav](https://user-images.githubusercontent.com/94472101/150888764-177bb123-0a10-4472-a4c0-c610c8397cc7.png)  
      ![ssh](https://user-images.githubusercontent.com/94472101/150889130-d29b8433-7257-408b-a2ba-24e459374c28.png)
      
     >> Note: Usually jenkins will work withthe previlege of jenkins user by default. So change update the ownership of .pem file with jenkins user.
     
   8. Now configure webhooks to trigger the playbook by sending an alert to the webhook URL by GitHub. Based on this alert Jenkins will automate the playbook.

      Login to GitHub > navigate to the corressponding repository >> Settings >> Webhooks >> Enter the wbhook url (http://ip:8080/github-webhook/) under "Payload       URL" >> Update Webhook
      
      ![webhook](https://user-images.githubusercontent.com/94472101/150889971-a618bd9e-f579-437a-880f-f76cc2a25660.png)


## Result

     A Flask Application container is deployed now. And whenever a commit is done in github, then the change will be automatically deployed by jenkin server when  the webhook triggers.
 

