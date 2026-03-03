# ☁️ AWS CLI

> Frequently used AWS CLI commands for managing cloud resources.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Installation & Configuration](#installation--configuration)
- [EC2](#ec2)
- [S3](#s3)
- [IAM](#iam)
- [VPC & Networking](#vpc--networking)
- [EKS](#eks)
- [ECR](#ecr)
- [RDS](#rds)
- [Lambda](#lambda)
- [CloudFormation](#cloudformation)
- [CloudWatch & Logs](#cloudwatch--logs)
- [Secrets Manager & SSM](#secrets-manager--ssm)

---

## Installation & Configuration

```bash
# Install AWS CLI v2 (Linux)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify
aws --version

# Configure credentials
aws configure
# AWS Access Key ID:     AKIA...
# AWS Secret Access Key: xxxxx
# Default region:        us-east-1
# Default output format: json

# Named profiles
aws configure --profile production
aws configure --profile staging

# Use a profile
aws s3 ls --profile production
export AWS_PROFILE=production

# View current config
aws configure list
aws sts get-caller-identity
```

---

## EC2

```bash
# List instances
aws ec2 describe-instances
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress,Tags[?Key==`Name`].Value|[0]]' --output table

# Start / stop / terminate instances
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0

# Launch an instance
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678 \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyServer}]'

# Key pairs
aws ec2 create-key-pair --key-name my-key --query 'KeyMaterial' --output text > my-key.pem
aws ec2 describe-key-pairs
aws ec2 delete-key-pair --key-name my-key

# Security groups
aws ec2 describe-security-groups
aws ec2 create-security-group --group-name my-sg --description "My SG" --vpc-id vpc-12345678
aws ec2 authorize-security-group-ingress --group-id sg-12345678 --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-12345678 --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 delete-security-group --group-id sg-12345678

# AMIs
aws ec2 describe-images --owners self
aws ec2 create-image --instance-id i-1234567890 --name "my-ami" --no-reboot
aws ec2 deregister-image --image-id ami-12345678

# Elastic IPs
aws ec2 allocate-address --domain vpc
aws ec2 associate-address --instance-id i-1234567890 --allocation-id eipalloc-12345678
aws ec2 release-address --allocation-id eipalloc-12345678
```

---

## S3

```bash
# List buckets
aws s3 ls
aws s3 ls s3://mybucket
aws s3 ls s3://mybucket/prefix/

# Create / delete bucket
aws s3 mb s3://mybucket --region us-east-1
aws s3 rb s3://mybucket --force     # force delete non-empty bucket

# Upload / download files
aws s3 cp file.txt s3://mybucket/
aws s3 cp s3://mybucket/file.txt ./
aws s3 cp s3://mybucket/file.txt s3://mybucket2/  # copy between buckets

# Upload directory
aws s3 cp ./dist s3://mybucket/ --recursive
aws s3 cp ./dist s3://mybucket/ --recursive --acl public-read

# Sync directory
aws s3 sync ./dist s3://mybucket/
aws s3 sync ./dist s3://mybucket/ --delete   # remove files not in source
aws s3 sync s3://mybucket/ ./backup/

# Move / remove
aws s3 mv file.txt s3://mybucket/
aws s3 rm s3://mybucket/file.txt
aws s3 rm s3://mybucket/ --recursive

# Presigned URL (temporary access)
aws s3 presign s3://mybucket/file.txt --expires-in 3600

# Bucket policy
aws s3api put-bucket-policy --bucket mybucket --policy file://policy.json
aws s3api get-bucket-policy --bucket mybucket
aws s3api delete-bucket-policy --bucket mybucket

# Enable versioning
aws s3api put-bucket-versioning --bucket mybucket \
  --versioning-configuration Status=Enabled

# Enable website hosting
aws s3 website s3://mybucket/ --index-document index.html --error-document error.html
```

---

## IAM

```bash
# Users
aws iam list-users
aws iam create-user --user-name myuser
aws iam delete-user --user-name myuser

# Access keys
aws iam create-access-key --user-name myuser
aws iam list-access-keys --user-name myuser
aws iam delete-access-key --user-name myuser --access-key-id AKIA...

# Groups
aws iam list-groups
aws iam create-group --group-name developers
aws iam add-user-to-group --user-name myuser --group-name developers
aws iam remove-user-from-group --user-name myuser --group-name developers

# Roles
aws iam list-roles
aws iam create-role --role-name myrole --assume-role-policy-document file://trust.json
aws iam attach-role-policy --role-name myrole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
aws iam list-attached-role-policies --role-name myrole
aws iam delete-role --role-name myrole

# Policies
aws iam list-policies --scope Local
aws iam create-policy --policy-name mypolicy --policy-document file://policy.json
aws iam get-policy --policy-arn arn:aws:iam::123456789012:policy/mypolicy
aws iam delete-policy --policy-arn arn:aws:iam::123456789012:policy/mypolicy

# Password policy
aws iam update-account-password-policy \
  --minimum-password-length 14 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters
```

---

## VPC & Networking

```bash
# List VPCs
aws ec2 describe-vpcs

# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create subnet
aws ec2 create-subnet --vpc-id vpc-12345678 --cidr-block 10.0.1.0/24 --availability-zone us-east-1a

# Internet Gateway
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-12345678 --vpc-id vpc-12345678

# Route tables
aws ec2 describe-route-tables
aws ec2 create-route-table --vpc-id vpc-12345678
aws ec2 create-route --route-table-id rtb-12345678 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-12345678

# NAT Gateway
aws ec2 create-nat-gateway --subnet-id subnet-12345678 --allocation-id eipalloc-12345678
```

---

## EKS

```bash
# List clusters
aws eks list-clusters

# Create cluster
eksctl create cluster --name mycluster --region us-east-1 --nodegroup-name workers \
  --node-type t3.medium --nodes 3 --nodes-min 1 --nodes-max 5

# Update kubeconfig
aws eks update-kubeconfig --name mycluster --region us-east-1

# Describe cluster
aws eks describe-cluster --name mycluster

# List nodegroups
aws eks list-nodegroups --cluster-name mycluster

# Scale nodegroup
aws eks update-nodegroup-config --cluster-name mycluster \
  --nodegroup-name workers --scaling-config minSize=1,maxSize=5,desiredSize=3

# Delete cluster
eksctl delete cluster --name mycluster
```

---

## ECR

```bash
# Create repository
aws ecr create-repository --repository-name myapp --region us-east-1

# List repositories
aws ecr describe-repositories

# Get login token (Docker)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com

# Tag and push image
docker tag myapp:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# List images in repository
aws ecr list-images --repository-name myapp

# Delete images
aws ecr batch-delete-image --repository-name myapp --image-ids imageTag=latest

# Delete repository
aws ecr delete-repository --repository-name myapp --force
```

---

## RDS

```bash
# List DB instances
aws rds describe-db-instances

# Create DB instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --master-username admin \
  --master-user-password password123 \
  --allocated-storage 20

# Create snapshot
aws rds create-db-snapshot --db-instance-identifier mydb --db-snapshot-identifier mydb-snap-$(date +%Y%m%d)

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier mydb-restored \
  --db-snapshot-identifier mydb-snap-20240101

# Start / stop instance
aws rds start-db-instance --db-instance-identifier mydb
aws rds stop-db-instance --db-instance-identifier mydb

# Delete instance
aws rds delete-db-instance --db-instance-identifier mydb --skip-final-snapshot
```

---

## Lambda

```bash
# List functions
aws lambda list-functions

# Create function
aws lambda create-function \
  --function-name myfunction \
  --runtime python3.11 \
  --role arn:aws:iam::123456789012:role/lambda-role \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

# Update function code
aws lambda update-function-code \
  --function-name myfunction \
  --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
  --function-name myfunction \
  --payload '{"key":"value"}' \
  response.json

# Get logs
aws lambda invoke --function-name myfunction --log-type Tail response.json \
  --query 'LogResult' --output text | base64 -d

# Delete function
aws lambda delete-function --function-name myfunction
```

---

## CloudFormation

```bash
# List stacks
aws cloudformation list-stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

# Create stack
aws cloudformation create-stack \
  --stack-name mystack \
  --template-body file://template.yaml \
  --parameters ParameterKey=Env,ParameterValue=production \
  --capabilities CAPABILITY_IAM

# Update stack
aws cloudformation update-stack \
  --stack-name mystack \
  --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name mystack

# Describe stack
aws cloudformation describe-stacks --stack-name mystack

# Stack events
aws cloudformation describe-stack-events --stack-name mystack

# Validate template
aws cloudformation validate-template --template-body file://template.yaml
```

---

## CloudWatch & Logs

```bash
# List log groups
aws logs describe-log-groups

# List log streams
aws logs describe-log-streams --log-group-name /aws/lambda/myfunction

# Get log events
aws logs get-log-events \
  --log-group-name /aws/lambda/myfunction \
  --log-stream-name "2024/01/01/[$LATEST]abc123"

# Tail logs (filter)
aws logs filter-log-events \
  --log-group-name /aws/lambda/myfunction \
  --filter-pattern "ERROR"

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name HighCPU \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0
```

---

## Secrets Manager & SSM

```bash
# Secrets Manager
aws secretsmanager list-secrets
aws secretsmanager create-secret --name myapp/db-password --secret-string "s3cr3t"
aws secretsmanager get-secret-value --secret-id myapp/db-password
aws secretsmanager update-secret --secret-id myapp/db-password --secret-string "newpassword"
aws secretsmanager delete-secret --secret-id myapp/db-password --force-delete-without-recovery

# SSM Parameter Store
aws ssm put-parameter --name "/myapp/db-host" --value "db.example.com" --type String
aws ssm put-parameter --name "/myapp/db-password" --value "s3cr3t" --type SecureString
aws ssm get-parameter --name "/myapp/db-password" --with-decryption
aws ssm get-parameters-by-path --path "/myapp" --with-decryption
aws ssm delete-parameter --name "/myapp/db-host"
```

---

[← Back to Home](../README.md)
