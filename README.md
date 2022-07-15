To run the code you must change to your profile in the provider.tf folder
install Jenkins after the instance have been provisioned by doing steps mentioned in "JENKINS INSTALLATION ON DOCKER CONTAINERS "
Test autoscaling as mentioned in "AUTOSCALING CHECKING"
The main focus was to complete the task to acceptance criteria mentioned by asignee like JIRA TICKET
More az/subnets and instances can be modified by adding details.
Additional services can be added to enhance user expereince by clients.
I added in autoscaling by imagining a burstableload due to any event at a specified time with "start_time = "2022-08-09T18:00:00Z" 
I commented NACLs but security is enhanced through WAF, AWS Shield , AWS Guard and AWS Inspect. SGs have access with ssh to instances.
AWS Promethus Monitoring service is added as amp.tf folder using default sns topic as receiver. You may explore the rules and alert as asked in requirements.
I thought to bake customised AMI with Ansible Playbook but then decided on installing Jenkins as mentioned above on docker container.

#######################################################
JENKINS INSTALLATION ON DOCKER CONTAINERS 
#######################################################

#1
yum install docker* -y

#2
systemctl start docker
#3
systemctl enable docker

#4
Open Jenkins website and look for Downloads. Click then Docker and it would take you to Dockerhub website of Jenkins image

#5 To ensure no container is running prior to running RUN command
docker container ls

#6 8081(dockerhost port) and 8080(Jenkins port)
docker container run -itd-p 8081:8080 jenkins

#7 To check docker container running
docker container ls
#8 Copy the public ip of docker host and paste in the browser like this 3.215.77.219 is sample ip
3.215.77.219:8081

#9 Step 8 if successful should open Jenkins main page and it would ask for Administrator password

#10 To get inside the container type in "docker container ls" without "" to get the container id
docker container exec -it eacaa6a0a580 /bin/bash

#11
cd /var/jenkins_home/

#12 To get into secrets
cd secrets/

#13
ls -l

#14 cat initialAdminPassword

#15 It should display the password and it would look like mix of 31 alphabets and numbers.Copy paste this password into Jenkins requirement of password and SAVE

#16 It would install plugins and after installation is finished it would open up Getting Started page after you click CONTINUE.

#17 Create First Admin User. Here type in Username : admin , Password , Full Name

#18 Start Using Jenkins

##########################################################################
AUTOSCALING CHECKING
##########################################################################

# CentOS/REHL 7/Amazon Linux
sudo yum install stress-ng -y

# CentOS/REHL 7/Amazon Linux
sudo yum install stress -y

#Now run this command to see all available options:
stress-ng --help

Stress using CPU-bound task:
stress-ng --cpu 4 -t 30s



################################################################################################################
PACKER INSTALLATION TO BAKE AMIs AND DEPLOY ANSIBLE PLAYBOOKS BY MAKING AS ec2 SERVER as PACKER/TERRAFORM SERVER
################################################################################################################

sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install packer




######################################################################
PACKER TEMPLATE I WROTE - ADDITINAL STUFF - NOT REQUIRED BY ASSIGNMENT
######################################################################

"variables": {
    "aws_access_key" : "",
    "aws_secret_key" : ""

},


"builders": [
    {
    "type" : "amazon-ebs",
    "region" : "eu-west-1",
    "aws_access" : "{{user `aws_access_key`}}",
    "secret_key" : "{{user `aws_secret_key`}}",
    "source_ami" :  "ami-0bba0a4cb75835f71",
    "instance_type" :  "t2.micro",
    "ssh_username": "ec2-user"  ,
     "ami_name" :  "sky-ami-{{timestamp}}"
     } 	
	
	
],

"provisioner": [
     {
      
     "type" : "file",
     "source" : " ./cloudknowledgeuk/",
      "destination" : "/tmp",
     },


     {

     "type" : "shell",
     "pause_before" : "30s", 
     "max_retries" : "5",  
     "inline" : [
          "sleep 30",
          "sudo yum update -y",
          "sudo yum install httpd -y",
          "sudo cp -rvf /tmp/* /var/www/html",
          "sudo service httpd start",
          "sudo chkconfig httpd on",
          "sudo touch /tmp/abc{1..4}"

           


     },
     {
     },

     ]


###########################################################
GRAFANA PROMETHUS INSTALLATION WITH DOCKER COMPOSE FILE
###########################################################

version: '2'
services:
  prometheus:
    image: prom/prometheus
    ports:
      - '9090:9090'
    container_name: prometheus
    restart: always
    network_mode: host
    volumes:
      - '$HOME/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml'
  grafana:
    image: grafana/grafana
    ports:
      - '3000:3000'
    container_name: grafana
    restart: always
    network_mode: host
    depends_on:
      - prometheus
    volumes:
      - '$HOME/grafana/config/grafana.ini:/etc/grafana/grafana.ini'


