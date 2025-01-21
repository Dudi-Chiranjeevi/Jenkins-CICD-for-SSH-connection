# Jenkins-CICD-for-SSH-connection

To achieve the process of SSH Connection for remote servers from one master server

1. I have created 4 AWS Ec2 Instances, 1 is for Main server with T2-medium and other 3 servers with T2-micro.

2. I have implemented Jenkins CI/CD pipeline to Automate the process of SSH Connection to the Remote Servers, so I have installed Jenkins on my main server using this commands, to install and start the Jenkins.

sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo apt-get update
sudo apt-get install jenkins

3. As we are created all the servers in AWS with same keypair, so copy the content of that file which is downloaded in our local and create the .pem file in this location "/var/lib/jenkins/.ssh/keypair.pem" in the main server and paste the copied content.

4. We have a file like "/var/lib/jenkins/server_list.conf" that contains remote server IP's.

5. After we done with Jenkins pipeline execution, go to main server and use this command 
ssh -i <path of the SSH private file> ubuntu@<Ip_address>.
