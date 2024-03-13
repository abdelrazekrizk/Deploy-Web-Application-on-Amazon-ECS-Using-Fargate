# Developing and Deploying a Basic Web Application on Amazon ECS Using Fargate


**Architecture Diagram:**

<img src="./Architecture/Web-Application-Autoscaling.png" width="1000"/>

This is a guide on how to Developing and Deploying a Basic Web Application on **Amazon ECS** <br>
## Learning Objectives:

* Gain hands-on experience with containerization using **Docker**.<br>
* Understand how to use **Amazon ECR** to store and manage **container images**.<br>
* Learn the basics of creating and managing an **Amazon ECS cluster**.<br>
* Learn about **ECS** and its role in orchestrating containerized applications, including **task definitions**, **clusters**, and **services**.

Whether you are new to **Amazon ECS** or looking to level up your skills, this repository has you covered.<br>

`Note`: that Tutorial using `AIM with Administrative User`.<br>
Assuming you have an `AWS account`, Docker Engine AWS CLI follow these steps:

`Note`: that Tutorial using **EC2** with `ubuntu-jammy-22.04-amd64-server` Operation System<br>
        Docker version `24.0.5`, build `24.0.5-0`ubuntu1`~22.04.1` <br>
For other Operation System Follow the Official Docker Documunts [here](https://docs.docker.com/desktop/install/linux-install/).

Below are the steps to follow:

## Table of Contents
- [Step 1: Create a NodeJS Server Web Application](#step-1-create-a-nodejs-server-web-application)
- [Step 2: Create a Dockerfile](#step-2-create-a-dockerfile)
- [Step 3: Build, Run Docker Image and Test Locally.](#step-3-build-run-docker-image-and-test-locally)
- [Step 4: Build and Push Docker Image to Amazon ECR Repository](#step-4-build-and-push-docker-image-to-amazon-ecr-repository)
- [Step 5: Create an Amazon VPC](#step-5-create-an-amazon-vpc)
- [Step 6: Set Up Amazon ECS](#step-6-set-up-amazon-ecs)
- [Step 7: Create a Task Definition](#step-7-create-a-task-definition)
- [Step 8: Deploy the Application](#step-8-deploy-the-application)
- [Step 9: Test The Application](#step-9-test-the-application)
- [Step 10: Cleanup](#step-10-cleanup)

## Step 1: Create a NodeJS Server Web Application.
Create a basic `Node.js Server Web Application`. <br> [server.js](./app/server.js):

## Step 2: Create a Dockerfile
Create a `Dockerfile` to containerize the Flask application.<br> [Dockerfile](./app/Dockerfile):

## Step 3: Build, Run Docker Image and Test Locally.
- Open a terminal or command prompt and navigate to the directory containing the **Dockerfile** and run the following command:
```
docker build -t <image-name> .
```
- Run the newly built image.
```
docker run -p <container-Port>:<host-Port> <image-name>
```
- **Replace:**<br>
`<image-name>` with your **image-name** `ex: hello-world`.<br>
`<container-Port>` with your **container-Port** define within Dockerfile`ex: 3000`.<br>
`<host-Port>` with the application port to listen on **host-Port** `ex: 3000`.<br>

- Access `http://localhost:3000` or `http://127.0.0.1:3000/`in your web browser to see the `"Hello, this is a basic web application running on Amazon ECS!"` message


## Step 4: Build and Push Docker Image to Amazon ECR Repository.

* Create an **ECR repository** to store your Docker images.
* Make note of the repository URI.

### 4.1. Ensure ECR Permissions.

- Verify that your IAM user or role has the necessary permissions to access the ECR repository.<br>
 The required permissions include: <br>

`ecr:GetAuthorizationToken `<br>
`ecr:BatchCheckLayerAvailability` <br>

You can attach the `AmazonEC2ContainerRegistryPowerUser` policy to your **IAM user** or **role** to grant these permissions.

### 4.2. Log in to AWS ECR.

 ### *private Registry.*

- Authenticate a private registry.
```
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<your-region>.amazonaws.com
```
- **Replace:**<br>
`<your-region>` with your desired **region**.<br>
`<aws_account_id>` with your **aws_account_id** <br>

### 4.3. Create a Private Amazon ECR Repository.
```
aws ecr create-repository \
        --repository-name <repository-name> \
        --region <your-region>
```
- **Replace:**<br>
`<repository-name>` with your **repository-name**.<br>
`<your-region>` with your desired **region**.<br>

### 4.4. Tag and Push Docker Image.

- Tag the image to push to your private repository.<br>
- List the images you have stored locally to identify the image to tag and push.
```
docker images
```
- Tag the image to push to `Amazon ECR private` repository.
```
docker tag <image-name:tag> <aws_account_id>.dkr.ecr.<your-region>.amazonaws.com/<repository-name:tag>
```
- Push the image.
```
docker push <aws_account_id>.dkr.ecr.<your-region>.amazonaws.com/<repository-name:tag>
```
- **Replace:**<br>
`<image-name:tag>` with your **image-name:tag** `ex: hello-world:latest`<br>
`<aws_account_id>` with your **aws_account_id**.<br>
`<your-region>` with your desired **region**.<br>
`<repository-name>` with your **repository-name**.<br>
`Note`: ECR repository URI= `<aws_account_id>.dkr.ecr.<your-region>.amazonaws.com/<repository-name:tag>`

Follow the Official Documunts Amazon Quick start: Publishing to `Amazon ECR Pprivate repository` using the AWS CLI [here](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html).

  ### *Public Registry.*

- Authenticate a public registry.
```
aws ecr-public get-login-password --region <your-region> | docker login --username AWS --password-stdin public.ecr.aws
```
### 4.5. Create a Public Amazon ECR Repository.
- Open a terminal or command prompt and navigate to the ECR directory and run the following command:.
```
aws ecr-public create-repository \
        --repository-name <repository-name> \
        --catalog-data file://repositorycatalogdata.json \
        --region <your-region>
```
- **Replace:**<br>
`<repository-name>` with your **repository-name**.<br>
`<your-region>` with your desired **region**.<br>

### 4.6. Tag and Push Docker Image.
- Tag and Push an image to `Amazon ECR Public`repository. <br>
List the images you have stored locally to identify the image to tag and push.
```
docker images
```
- Tag the image to push to your repository with your `public repository URI` <br>
which was in the response to the `create-repository` call you made in the previous step.
```
docker tag <image-name:tag> public.ecr.aws/<registry_alias>/<repository-name>
```
- Push the image to the Amazon ECR.
```
docker push public.ecr.aws/<registry_alias>/<repository-name>
```
- Pull the image from the Amazon ECR.
```
docker pull public.ecr.aws/g7p1j8g3/hello-flask:v.1
```
- **Replace:**<br>
`<registry_alias>` with your **registry_alias**.<br>
`<repository-name>` with your **repository-name**.<br>

Follow the Official Amazon Quick start: Publishing to `Amazon ECR Public` using the AWS CLI [here](https://docs.aws.amazon.com/AmazonECR/latest/public/getting-started-cli.html).

## Step 5: Create an Amazon VPC.
- you can create VPC using `AWS CLI`or `cloudformation`
- Create an Amazon VPC in `two availability zone` with `two Public` and `two Private subnets` and configure `security group` rules to allow traffic on **Port 3000**using `AWS CLI`.

### 5.1. Create VPC:
- Open a terminal or command prompt and run the following command:.
```
aws ec2 create-vpc \
    --cidr-block <CIDR block> \
    --region <your-region> \
    --tag-specification ResourceType=vpc,Tags='<Tags>'
```
- **Replace:**<br>
`<CIDR block>` with your desired **CIDR block.**`ex: 10.0.0.0/16`.<br>
`<your-region>` with your desired **region**.<br>
`<Tags>`  with your desired **Tags** `ex: Tags='[{Key=Name,Value="ECS-VPC"},{Key=Owner,Value="Network Team"}]'`.<br>

* Retrieve the VPC ID:

```
aws ec2 describe-vpcs \
        --query 'Vpcs[0].VpcId'
```
- outputs

```
"vpc-0fc3b7820d89cea5a"
```
- Make note and write down `VpcId`:<br>

### 5.2. Create Internet Gateway:
- allows communication between your VPC and the internet.
```
aws ec2 create-internet-gateway \
        --region <your-region> \
        --tag-specification ResourceType=internet-gateway,Tags='<Tags>'
```
### 5.3. Attach Internet Gateway to VPC:
* Retrieve the Internet Gateway ID:
```
aws ec2 describe-internet-gateways \
        --query 'InternetGateways[0].InternetGatewayId'
```
- outputs

```
"igw-024ee93e80e7ca1ec"
```
- Make note and write down `InternetGatewayId`:<br>

```
aws ec2 attach-internet-gateway \
        --internet-gateway-id <Internet-Gateway-ID> \
        --vpc-id <VPC_ID>
```
- **Replace:**<br>
`<VPC_ID>` with your Retrieved **VPC ID** .<br>
`<Internet-Gateway-ID>` with your Retrieved **INTERNET GATEWAY ID** .<br>
`<your-region>` with your desired **region**.<br>
`<Tags>` with your desired **Tags** `ex: Tags='[{Key=Name,Value="ECS Internet-Gateway"},{Key=Owner,Value="Network Team"}]'`.<br>

### 5.4. Create Subnets:
- Create two Subnets on one availability zone .
```
aws ec2 create-subnet \
        --vpc-id <VPC_ID> \
        --cidr-block <CIDR block> \
        --region <your-region> \
        --availability-zone <your-az-1> \
        --tag-specification ResourceType=subnet,Tags='<Tags>'
```
- Create two Subnets on second availability zone .
```
aws ec2 create-subnet \
        --vpc-id <VPC_ID> \
        --cidr-block <CIDR block> \
        --region <your-region> \
        --availability-zone <your-az-2> \
        --tag-specification ResourceType=subnet,Tags='<Tags>'
```
- **Replace:**<br>
`<VPC_ID>` with your Retrieved **VPC ID** .<br>
`<CIDR block>` with your desired **CIDR block.** `ex: 10.0.1.0/24`.<br>
`<your-region>` with your desired **region**.<br>
`<your-az-1>,<your-az-2> with your desired **availability zones** `ex: us-east-1c`.<br>
`<Tags>` with your desired **Tags** `ex: Tags='[{Key=Name,Value="ECS private subnets_1"},{Key=Owner,Value="Network Team"}]'`.<br>

Make note and write down:<br>
            - Subnets IDs in the **VPC** `<PrivateSubnet01>, <PrivateSubnet02>, <PublicSubnet01>, <PublicSubnet02>`.<br>

### 5.5. Create an Elastic IP (EIP):
- Allocates two Elastic IP address
```
aws ec2 allocate-address \
        --domain vpc <VPC_ID> \
        --region <your-region> \
        --tag-specification ResourceType=elastic-ip,Tags='<Tags>'
```
- **Replace:**<br>
`<VPC_ID>` with your Retrieved **VPC ID** .<br>
`<your-region>` with your desired **region**.<br>
`<Tags>` with your desired **Tags** `ex: Tags='[{Key=Name,Value="ECS-elastic-ip"},{Key=Owner,Value="Network Team"}]'`.<br>

### 5.6. Create a NAT Gateway:
- Creates `two` **NAT gateway** for each **Public Subnet**
```
aws ec2 create-nat-gateway \
        --subnet-id <subnet-id> \
        --allocation-id <Elastic IP-id> \
        --region <your-region> \
        --tag-specification ResourceType=natgateway,Tags='<Tags>'
```
- **Replace:**<br>
`<subnet-id>` with your Retrieved public **subnet-id** .<br>
`<Elastic IP-id>` with your Retrieved **region**.<br>
`<your-region>` with your desired **region**.<br>
`<Tags>` with your desired **Tags** `ex: Tags='[{Key=Name,Value="ECS-NAT-Gateway"},{Key=Owner,Value="Network Team"}]'`.<br>

### 5.7. Create Route Table:
- Creates `four` Route Table for your VPC
```
aws ec2 create-route-table \
        --vpc-id <VPC-ID> \
        --region <your-region> \
        --tag-specification ResourceType=route-table,Tags='<Tags>'
```
- **Replace:**<br>
`<VPC-ID>` with your Retrieved **<VPC-ID>** .<br>
`<your-region>` with your desired **region**.<br>
`<Tags>` with your desired **Tags** `ex: Tags='[{Key=Name,Value="ECS-pub-sub-1-route-table"},{Key=Owner,Value="Network Team"}]'`.<br>

### 5.8. Associates a Route Table with Subnets:
- Associates each public subnet in your VPC to your public route table
- Associates each private subnet in your VPC to your each private route table
```
aws ec2 associate-route-table \
        --route-table-id <route-table-id> \
        --subnet-id <subnet-id>
```
- **Replace:**<br>
`<subnet-id>` with your Retrieved **subnet-id** .<br>
`<route-table-id>` with your Retrieved **Route Table ID**.<br>

### 5.9. Add Routes to the Route Table :
- Creates a route betwen `private route` table and `NAT gateway `
```
aws ec2 create-route \
        --route-table-id <route-table-id> \
        --destination-cidr-block 0.0.0.0/0 \
        --gateway-id <Internet-Gateway-ID>
```
- **Replace:**<br>
`<route-table-id>` with your Retrieved **Route Table ID**.<br>
`<gateway-id>` with your Retrieved **NAT Gateway ID**.<br>

### 5.10. Create Security Group:
```
aws ec2 create-security-group \
        --group-name <security-group-name> \
        --description <Description> \
        --vpc-id <VPC_ID> \
        --region <your-region> \
        --tag-specification ResourceType=security-group,Tags='<Tags>'
```
- **Replace:**<br>
`<security-group-name>` with your desired **security-group-name** .<br>
`<Description>` with your **security group description** `ex: "Security group for port 3000" `.<br>
`<VPC_ID>` with your Retrieved **VPC ID** .<br>
`<your-region>` with your desired **region**.<br>
`<Tags>` with your desired **Tags** `ex: Tags='[{Key=Name,Value="ECS-security-group"},{Key=Owner,Value="security Team"}]'`.<br>

### 5.11. Configure Security Group Ingress Rules:
```
aws ec2 authorize-security-group-ingress \
        --group-id <security-group-id> \
        --protocol <protocol> \
        --port <port> \
        --cidr <cidr> \
        --region <your-region> \
        --tag-specification ResourceType=security-group-rule,Tags='<Tags>'
```
- **Replace:**<br>
`<security-group-id>` with your Retrieved **security group ID** .<br>
`<protocol>` with your desired **protocol** `ex: protocol name (tcp , udp , icmp , icmpv6 ) or number` [Protocol_Numbers](http://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml).<br>
`<port>` with **port** for the application to listen on. `ex: 3000 `.<br>
`<cidr>` with your desired **IPv4 CIDR range** `ex: 0.0.0.0/0` .<br>
`<your-region>` with your desired **region**.<br>
`<Tags>` with your desired **Tags** `ex: Tags='[{Key=Name,Value="ECS-security-group"},{Key=Owner,Value="security Team"}]'`.<br>

### 5.12. Create an Amazon VPC using cloud formation.
- Create an Amazon VPC with public and private subnets that meets Amazon ECS requirements.
- Open a terminal or command prompt and navigate to the **cloudformation directory** and run the following command:.

        aws cloudformation create-stack \
            --region <your-region> \
            --stack-name <stack-name> \
            --template-body file://amazon-ECS-vpc-private-subnets.yaml

- **Replace:**<br>
`<your-region>` with your **region**.<br>
`<stack-name>` with your **stack-name**.<br>

- Navigate to the `cloud-formation outputs` Make note and write down:<br>
            - **Subnets IDs** in the **VPC** `<PrivateSubnet01>, <PrivateSubnet02>, <PublicSubnet01>, <PublicSubnet02>`<br>
            - **SecurityGroups** `<Security group>`
            -
## Step 6: Set Up Amazon ECS.

 ### 6.1.Create an ECS Cluster:
```
aws ecs create-cluster \
        --cluster-name <cluster-name> \
        --region <your-region> \
        --tags <tag>
```
- **Replace:**<br>
`<cluster-name>` with your **cluster name**. <br>
`<your-region>` with your desired **region**.<br>
`<tag>`  your **tag** : Add tags for better organization `ex: Replace <tag> with Key=name,Value=ECS-project key=dep,value=dev.`<br>
Follow the Official Creating a cluster with a Fargate Linux task using the AWS CLI [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_AWSCLI_Fargate.html)

## Step 7: Create a Task Definition:
```
aws ecs register-task-definition \
        --family <task-name> \
        --execution-role-arn "arn:aws:iam::<aws_account_id>:role/ecsTaskExecutionRole" \
        --network-mode <network-mode> \
        --region <your-region> \
        --cpu "<cpu>" \
        --memory "<memory>" \
        --container-definitions '[{"name":"<container-name>","image":"<aws_account_id>.dkr.ecr.<your-region>.amazonaws.com/<image-name>:<tag>","essential":true,"portMappings":[{"containerPort":<container-Port>,"protocol":"tcp"}]}]' \
        --tags <tag>
```
- **Replace:**<br>
`<task-name>` with your **task-name** .<br>
`<aws_account_id>` with your **aws_account_id**.<br>
`<your-region>` with your desired **region**.<br>
`<network-mode>` with your desired **network-mode**The valid values are `none` , `bridge` , `awsvpc` , and `host` . If no network mode is specified, the default is `bridge` .<br>
`<cpu>` with your desired **cpu** `ex: "256"`.<br>
`<memory>` with your desired **memory** `ex: "512"`.<br>
`<container-Port>` with your the application **port** to listen on `ex: 3000`.<br>
`<image-name>:<tag>` with your **image-name:tag** `ex: hello-world:latest`.<br>
`<container-name>` with the **container-name** .<br>

`Note`: ECR repository URI= `<aws_account_id>.dkr.ecr.<your-region>.amazonaws.com/<image-name:tag>`.<br>

Follow the Official Amazon ECR image and task definition [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/example_task_definitions.html#example_task_definition-iam)
and [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_AWSCLI_EC2.html#AWSCLI_EC2_register_task_definition)

## Step 8: Deploy the Application

### 8.1.Create an ECS Service launch Type FARGATE:
```
aws ecs create-service \
        --cluster <cluster-name> \
        --service-name <service-name> \
        --task-definition <task-name> \
        --desired-count 1 \
        --launch-type <launch-type> \
        --region <your-region> \
        --network-configuration "awsvpcConfiguration={subnets=[<PublicSubnet01>,<PublicSubnet02>,<PrivateSubnet01>,<PrivateSubnet02>],securityGroups=[<Security-GroupIds>],assignPublicIp=ENABLED}" \
        --tags <tag>
```
- **Replace:**<br>
`<cluster-name>` with your **cluster name**. <br>
`<service-name>` with your **service name**. <br>
`<task-name>` with your **task-name** .<br>
`<desired-count>` with your desired **instantiations-number** of the specified task definition to place and keep running in your service.<br>
`<your-region>` with your desired **region**.<br>
`<launch-type>` with your desired **launch-type** that match **network-mode** The Possible values are :`EC2`, `FARGATE` and `EXTERNAL`.<br>
`<subnets-id>` with your **subnets**
`<security-Group>` with your **security Groups**
`<tag>`  your **tag** : Add tags for better organization `ex: Replace <tag> with Key=name,Value=ECS-project key=dep,value=dev.`<br>

## Step 9: Test the Application.

1. Open the AWS console [here](https://console.aws.amazon.com/ecs/v2.)
2. In the navigation pane, choose Clusters [Amazon Elastic Container Service](./screenshot/Amazon%20Elastic%20Container%20Service.png).
3. Choose the cluster where you ran the service.[Cluster overview](./screenshot/Cluster%20overview.png)
4. In the Cluster overview Choose Services tab, under Service name, choose the service you created in Step 7 [Services](./screenshot/service.png).
5. Choose the Tasks tab, and then choose the task in your service.[Tasks](./screenshot/Tasks.png).
6. On the task page, in the Configuration section, under Public IP, choose Open address [Public IP](./screenshot/Public%20IP.png).
7. On your browser edit IP and add `:3000`
8. Now you can see `"Hello, this is a basic web application running on Amazon ECS!"`[Deployment](./Screenshot/deployment.png)


## Step 10: Cleanup.
- Cleaning up your resources is an important part of managing your cloud environment. <br>
- By following these steps, you can ensure that your resources are always **clean** and **tidy** and it will also help you to **avoid unnecessary costs**.

### 10.1. Deregister Task Definitions:
```
aws ecs deregister-task-definition \
        --task-definition <task-name>
```
- **Replace:**<br>
`<task-name>` with your **task-name** .<br>

### 10.2. Delete Task Definitions:
```
aws ecs delete-task-definitions \
        --task-definition <task-name>
```
- **Replace:**<br>
`<task-name>` with your **task-name** .<br>

### 10.3. Delete Service:
```
 aws ecs delete-service \
        --cluster <cluster-name> \
        --service <service-name> \
        --force
```
- **Replace:**<br>
`<cluster-name>` with your **cluster name**. <br>
`<service-name>` with your **service name**. <br>
`<task-name>` with your **task-name** .<br>

### 10.4. Delete Cluster:
```
aws ecs delete-cluster \
         --cluster <cluster-name>
```
- **Replace:**<br>
`<cluster-name>` with your **cluster name**. <br>

### 10.5. CloudFormation Stack:
```
aws cloudformation delete-stack \
  --stack-name <stack-name>
```
- **Replace:**<br>
`<stack-name>` with your **stack-name**. <br>

### 10.6. EC2 (Stop or Terminate):

* To stop an EC2 instance:

```
aws ec2 stop-instances \
--instance-ids <instance-id>
```

* To terminate an EC2 instance:

```
aws ec2 terminate-instances \
--instance-ids <instance-id>
```
- **Replace:**<br>
`<instance-id>` with your **instance-ids**. <br>

- Remember to always follow the best practices and guidelines provided by your cloud provider to ensure a smooth and efficient cleanup process.
- Regularly reviewing and optimizing your resource utilization can not only save costs but also improve the overall performance and efficiency of your cloud infrastructure.<br>

  Keep in mind that this example is minimal and focuses on the basic steps. <br>
  In a production scenario, you would likely include more `features`, `security measures`, and `configurations`.<br>
