Continuous Delivery pipeline for ECS, leveraging github, Jenkins and ECR

This project will help you set up and configure a Continuous delivery pipeline for AWS EC2(ECS) using github, Jenkins, ECR.
This pipeline builds Docker images from a github repo, pushes those images to an ECR registry, create a ECS task definition and then uses that task definition to create a service on the ECS cluster. We can use Jenkins for orchestration.

For this project you need these components installed on your machine.
•	Python
•	Pip
•	AWS cli
•	Homebrew

Installing Homebrew

	$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
.
.
.
==> Installation successful!

==> Homebrew has enabled anonymous aggregate formulae and cask analytics.
Read the analytics documentation (and how to opt-out) here:
https://docs.brew.sh/Analytics
No analytics data has been sent yet (or will be during this `install` run).

==> Homebrew is run entirely by unpaid volunteers. Please consider donating:
https://github.com/Homebrew/brew#donations

==> Next steps:
- Run `brew help` to get started
- Further documentation: 
https://docs.brew.sh

Verify homebrew installation
Brew –version


When you have installed Homebrew, insert the homebrew directory at the top of the PATH environment variable, you can do this by adding the following line at the bottom of your  ~/.profile file
PATH is an environment variable in linux and other Unix-like operating systems that tells the shell which directories to search for executable files in response to commands issued by a user

	export PATH=”usr/local/opt/python/libexec/bin:$PATH”

Install python3

	brew install python

Verify python is installed
	
	python –version

Some caveats:
You may set your default python as latest version by applying code

	$ unlink /usr/local/bin/python
	$ ln -s /usr/local/bin/python3.8 /usr/local/bin/python

Installing pip
	curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
	python get-pip.py

Verify pip installation
	pip -v
Install AWS CLI
Verify AWSCLI
	aws --version


1	Build an ECS cluster
1.	 Create an AWS access and secret key and opening a terminal and typing this
aws iam create-access-key  --user-name <username>

	<username> is an IAM user with AdministratorAccess

		aws iam create-access-key –user-name jaysalpatel
2.	Create an AWS profile 
	aws configure

3.	Create an SSH key in your closest region
Save the SSH pem file on your local machine

4.	Change the working directory to your Desktop directory and change permissions so only the current logged in user can read it
		cd Desktop
		chmod key.pem 400
	
5.	Verify file privilege
ls -lah key.pem

6.	Clone the git repository that contains the CloudFormation templates to create the infrastructure we will use to build our pipeline

git clone https://github.com/jicowan/hello-world

7.	Change the working directory to the directory that was created when you cloned the repository. At the command prompt, type of paste the following. Where <keyname> is the name of an SSH key in the region where you’re creating the ECS cluster.

aws cloudformation create-stack –template-body file://ecs-cluster.template –stack-name ECSClusterStack –capabilities CAPABILITY_IAM –tags Key=Name,Value=ECS –region us-east-1 –parameters ParameterKey=KeyName,ParameterValue=key ParameterKey=ECSCluster,ParameterValue=getting-started ParameterKey=AsgMaxSize,ParameterValue=2 

Note: Do not proceed to the next step until the Stack Status shows CREATE_COMPLETE. To get the status of the stack at a command prompt, type aws cloudformation describe-stacks --stack-name ECSClusterStack --query 'Stacks[*].[StackId, StackStatus]'

Step 2: Create a Jenkins server
	Jenkins is a popular server for implementing continuous integration and continuous delivery pipelines. In this project, you will use Jenkins to build a Docker image from a Dockerfile, push that image to the AWS ECR registry that you created earlier, and create a task definition for your container. Then you can deploy and update a service running on your ECS cluster. 

1.	Change the working directory to the root of the cloned repository, and then execute the following command
aws cloudformation create-stack –template-body file://ecs-jenkins-demo.template –stack-name JenkinsStack –capabilities CAPABILITY_IAM –tags Key=Name, Value=Jenkins –region us-east-1 –parameters ParameterKey=ECSStackName,ParameterValue=ECSClusterStack`
	
	Note: Do not proceed to the next step until the Stack Status shows CREATE_COMPLETE. To get the status of the stack type
	 	aws cloudformation describe-stacks –stack-name JenkinsStack –query ‘Stacks[*].[StackId, StackStatus]’

2.	Retrieve the public hostname of the Jenkins server. Open a terminal windows and type the command

aws ec2 describe-instances –filters “Name=tag-value”,”Values=JenkinsStack” | jq .Reservations[].Instances[].PublicDnsName

3.	Copy the hostname
4.	SSH into the instance, and then copy the Jenkins password from /var/lib/jenkins/secrets/initialAdminPassword

sudo cat /var/lib/jenkins/secrets/initialAdminPassword 

On OSX 
	ssh -i key.pem ec2-user@hostname

then logout of the instance
	logout

3. Create an ECR registry
	AWS ECR is a private docker container registry that you will use to store your container images. For this project, we will create a repository named hello-world in the us-east-1 region
	
1.	Create an ECR registry 
Aws ecr create-repository –repository-name hello-world –region us-east-1 
2.	Record the value of the URL of this repository

536510685689.dkr.ecr.us-east-1.amazonaws.com/hello-world


3.	Verify you can log in to the repo 
(aws ecr get-login)

4.	Install Jenkins on your EC2 instance

ssh -I key.pem ec2-user@2130.213.123

install software packages

sudo yum update -y

Add the Jenkins repo 

sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo

5.	Import a key file from Jenkins-CI to enable installation from the package

sudo rpm –import https://pkg.jenkins.io/redhat/jenkins.io.key


6.	Install Jenkins
sudo yum install jenkins -y 

7.	Start Jenkins as a service

sudo service jenkins start


step 5. Configure Jenkins

1.	Paste the public hostname of the Jenkins server

2.	Paste the password into the field

3.	Install plugins 

4.	Create Admin user and password

5.	Instance config

6.	Jenkins interface

7.	Install Jenkins plugins

1.	Click manage Jenkins
2.	Choose manage plugins tab
3.	Find CloudBees Docker build and publish plugin and the AWS ECR plugin
4.	Download now and install after restart
Step 6. Create and import SSH keys form Github
1.	Open a terminal window and paste this command into the window

ssh-keygen -t rsa -b 4069 -C jaysalpatel.aws@gmail.com

2.	Accept file location and type a passphrase
3.	Ensure ssh-agent is enabled
eval “$(ssh-agent -s)”

4.	Add SSH key to agent
ssh-add ~/.ssh/id_rsa

5.	Copy the contents of the id_rsa.pub file to the clipboard. You are going to use that for your github SSH keys
pbcopy < ~/.ssh/id_rsa.pub

6.	Login into github

1.	Go to SSH and GPG keys 
2.	Choose SSH key and Add SSH key
3.	sudo cat /.ssh/id_rsa
4.	Paste your key in the field

Steps 7. Create a github rpeo
1.	Create repo
2.	Push code to repo

cd hello-world/
3.	Delete the hidden .git directory if you are running OSX, 
Rm -fR .git

4.	Initialize the repo
5.	




	



