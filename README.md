![image](https://github.com/user-attachments/assets/38d7173c-f77c-4b5b-884e-be85aa00e59c)# Tooling-Website-deployment-automation-with-Continuous-Integration.-Introduction-to-Jenkins
Tooling Website deployment automation with Continuous Integration. Introduction to Jenkins

In previous Project, we introduced horizontal scalability concept, which allow us to add new Web Servers to our Tooling Website and we have successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. If it is just two or three servers - it is not a big deal to configure them manually. Imagine that you would need to repeat the same task over and over again adding dozens or even hundreds of servers.

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.

In this project we are going to start automating part of our routine tasks with a free and open source automation server - Jenkins. It is one of the mostl popular CI/CD tools, it was created by a former Sun Microsystems developer Kohsuke Kawaguchi and the project originally had a named "Hudson".

Acording to Circle CI, Continuous integration (CI) is a software development strategy that increases the speed of development while ensuring the quality of the code that teams deploy. Developers continually commit code in small increments (at least daily, or even several times a day), which is then automatically built and tested before it is merged with the shared repository.

In our project we are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/<yourname>/tooling will be automatically be updated to the Tooling Website.

## Task
Enhance the architecture prepared in Project 8 by adding a Jenkins server, configure a job to automatically deploy source codes changes from Git to NFS server.

#### Here is how your updated architecture will look upon competion of this project:

![image](https://github.com/user-attachments/assets/76a1ede9-a4bb-42c8-ba13-3f27dd4a3a9c)


## Step 1 - Install Jenkins server
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
![image](https://github.com/user-attachments/assets/4bb0dd0b-3715-40d7-83ee-375510f28528)

2. Install JDK (since Jenkins is a Java-based application).

```
sudo apt update
```
![image](https://github.com/user-attachments/assets/6fbaa279-50d0-4b69-9824-3c353d47c562)


```
sudo apt install default-jdk-headless
```
![image](https://github.com/user-attachments/assets/e85f4225-3675-41cc-9a61-2c058d3795af)

3. Install Jenkins

```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

Make sure Jenkins is up and running
```
sudo systemctl status jenkins
```
![image](https://github.com/user-attachments/assets/53c67534-3b96-4f61-b3c4-7c360aef58d8)


4. By default Jenkins server uses TCP port 8080 - open it by creating a new Inbound Rule in your EC2 Security Group.
![image](https://github.com/user-attachments/assets/51166252-af46-4cca-bd42-b8c45cc88d3b)

5. Perform initial Jenkins setup.

From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

![image](https://github.com/user-attachments/assets/c1147bf7-f29e-456f-8972-e86c8209fad1)

We will be prompted to provide a default admin password

Retrieve it from your server:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
![image](https://github.com/user-attachments/assets/e3093b8b-38d3-44ee-b936-ecd88708ce75)

Then you will be asked which plugings to install - choose suggested plugins.
![image](https://github.com/user-attachments/assets/4b934f87-0bf0-42c0-b909-88ae12414803)

![image](https://github.com/user-attachments/assets/7987a70a-a5f1-4032-a09d-cd1ef2799712)


## Step 2 - Configure Jenkins to retrieve source codes from GitHub using Webhooks.

In this part, we will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will  be triggered by GitHub webhooks and will execute a 'build' task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in we GitHub repository settings.

Go to Forked Repo Tooling In own github and Inside the tooling repo need to do this.

![image](https://github.com/user-attachments/assets/d7f146ef-0051-44ba-bdf1-c87b219fada1)

After Saving It Going to show:
![image](https://github.com/user-attachments/assets/89aa6a21-86d8-4089-9490-21ebc8fbabd4)

2. Go to Jenkins web console, click "New Item" and create a "Freestyle project"

![image](https://github.com/user-attachments/assets/b2bd8adf-f718-4dcb-a736-736d068ca12d)


To connect GitHub repository, We will need to provide its URL, we can copy from the repository itself
![image](https://github.com/user-attachments/assets/d3f811d5-4bc4-4444-a390-1fefdb4d0da7)

In configuration of Jenkins freestyle project choose Git repository, provide there the link to Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![image](https://github.com/user-attachments/assets/ed96aa41-41a2-452d-a30e-6cf5ac935d0f)
Save the configuration and let us try to run the build.


Got Error while building the jobs 
![image](https://github.com/user-attachments/assets/c1ef8d2e-0bf7-4106-ad62-a2f4df86d85a)
It is because of miss-configuration we need to edit configuration and have to change master to main branch. Need to select to trigger the pipeline.

![image](https://github.com/user-attachments/assets/5b308107-47ed-4424-8312-81aaf0ab6bfb)
Save the configuration and let us try to run the again build.

![image](https://github.com/user-attachments/assets/0d578ec6-02ba-4268-8d83-4a2f60a427b6)
It run successfully.

But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

3. Click "Configure" your job/project and add these two configurations
In Build trigger
![image](https://github.com/user-attachments/assets/d9cff3ea-b5bf-4035-b3e6-249f155c4d9f)
Save the configuration


4. Configure "Post-build Actions" to archive all the files - files resulted from a build are called "artifacts"
![image](https://github.com/user-attachments/assets/75ee0ae8-f1b9-4afb-aa89-3d3c8c3af4ec)

Under the form field for Files to archive, use ** to select all .
![image](https://github.com/user-attachments/assets/c136a336-5848-4ada-b049-6f76388d9176)

Save the configuration.

5. Now, go ahead and make some change in any file in we GitHub repository (e.g. README.MD file) and push the changes to the master branch. We will see that a new build has been launched automatically (by webhook) and you can see its results - artifacts, saved on Jenkins server.


6. 




