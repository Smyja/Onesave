# smyja.github.io

font-family: knowledge-medium,helvetica,arial,sans-serif;
font-weight: 500;
font-size: 1.8em;
line-height: 21px;

# udacity-high-availability-app
Udacity Cloud DevOps ND / High Availability App / Cloudformation <br>
This repository was created as submission to Udacities 'Deploy a high-availability web app using CloudFormation' Project (Cloud DevOps Nanodegree).

## Introduction
The goal of this project is to deploy an application in an automated fashion using AWS cloudformation. <br>
The resulting infrastructure can be discarded and rebuild at any time. <br>

In this scenario an app called Udagram is supposed to be deployed in a high-available and automated fashion. <br>
The application data is located in a S3 Bucket. <br>

To-Do's:
* create an Infrastructure Diagram
* create the S3 Bucket and place the application code in it (since there is no bucket yet)
* create the neccessary Cloudformation Scripts

## Infrastructure Diagram
![alt text][architecture]

[architecture]: infrastructure-diagram.png "Architecture Diagram"

## S3 Bucket
The S3 Bucket is needed to hold the application data (udacity.zip). On start each EC2-Instance will download and unpack this data.
The S3 Bucket can simply be created via the management console. Afterwards the data can be uploaded to the bucket.

In order to make its data accessible to other ressources, a bucket policy must be created.
[The following rule grants EC2 instances access to all actions to a specific bucket and to all the objects stored in it.](https://aws.amazon.com/de/premiumsupport/knowledge-center/s3-instance-access-bucket/) 
(This policy is probably to open and could be narrowed down)
```json
{
    "Version": "2012-10-17",
    "Id": "Policy1564815775429",
    "Statement": [
        {
            "Sid": "Stmt1564815768845",
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::<YOUR-BUCKET-NAME>",
                "arn:aws:s3:::<YOUR-BUCKET-NAME>/*"
            ]
        }
    ]
}
```
## CloudFormation
All required scripts are available in this repository. <br> 
network.yml sets up the required network infrastructure (VPC, Subnets, Gateways, Routing, etc.). <br> 
servers.yml will set up all components which are required to deploy the app (Load Balancer, Auto Scaling Group, Security Groups, Policies (for accessing the S3 Bucket), etc.). <br> 
Furthermore, additional bastion hosts are deployed to grant SSH access to the EC2 instances in the private subnet. <br>
In order to run these scripts AWS CLI is needed.

### Bastion Hosts
![alt text][ssh-agent-forwarding]

[ssh-agent-forwarding]: ssh-agent-forwarding.png "ssh-agent-forwarding"

[Bastion Hosts](https://docs.aws.amazon.com/quickstart/latest/linux-bastion/architecture.html) are used to connect to internal infrastructure without exposing it to the Internet. <br>
To connect from the local system to the internal EC2 instances, ssh-agent-forwarding is used. For this, 2 Key-Pairs are needed. The private keys are stored on the local system while the public keys are distributed over the bastion hosts and the EC2 instances (or rather the autoscaling group), since it is not recommended to store private keys on shared servers.

Key-Pairs can be created via the AWS management console. The private Key can be downloaded and stored afterwards. <br>
`1.` Add the key to the ssh-agent (Make sure that your ssh-agent is running)<br>
```ssh-add -k <PEM-File>```<br>
`2.` Connect<br>
```local:~$ ssh -A ubuntu@<IP-Addr1>```<br>
```ubuntu@<IP-Addr1>:~$ ssh ubuntu@<IP-Addr2>```<br>

Side Note: <br>
It is recommended to assign elastic IPs to each Bastion Host. These scripts use randomly assigned public IP adresses by AWS. 
The EC2 Security Group is configured in a way that it will always only allow the current private IP adress of the bastion hosts to access Port 22. 

### Create Stacks
The infrastructure is devided in 2 Stacks. The **network** Stack sets up the required network-infrastructure. The **servers** Stack sets up the required server to deploy the application. It requires the network-Stack to be already running.

`1.` Create the network stack <br>
```create_stack.sh network network.yml network-parameters.json```<br>
`2.` Create the servers stack<br>
```create_stack.sh servers servers.yml servers-parameters.json```<br>
or (in order to create IAM profiles)<br>
```aws cloudformation update-stack --stack-name servers --template-body file://servers.yml --parameters file://servers-parameters.json --capabilities CAPABILITY_IAM```

