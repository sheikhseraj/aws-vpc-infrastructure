# AWS CLI Commands

This document contains the AWS CLI commands used while building and validating the VPC infrastructure project.

> The commands have been sanitized for documentation. Resource IDs, IP addresses and environment-specific values should be replaced with values from your own AWS environment.

## 1. Verify AWS Authentication

```powershell
aws sts get-caller-identity

````
Check the configured AWS region:

```powershell
aws configure get region
```

The project was implemented in:

`us-east-1`

---

## 2. Inspect the VPC

The VPC was already available in the AWS training environment.

```powershell
aws ec2 describe-vpcs --query 'Vpcs[*].[VpcId,CidrBlock,State,IsDefault]' --output table
```

Project VPC CIDR:

`10.0.0.0/16`

---

## 3. Create Public Subnet

```powershell
aws ec2 create-subnet --vpc-id <VPC_ID> --cidr-block 10.0.1.0/24 --availability-zone us-east-1a --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=cloud-city-public-subnet}]'
```

Enable automatic public IPv4 assignment:

```powershell
aws ec2 modify-subnet-attribute --subnet-id <PUBLIC_SUBNET_ID> --map-public-ip-on-launch
```

---

## 4. Create Private Subnet

```powershell
aws ec2 create-subnet --vpc-id <VPC_ID> --cidr-block 10.0.2.0/24 --availability-zone us-east-1b --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=cloud-city-private-subnet}]'
```

---

## 5. Create Internet Gateway

```powershell
aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=cloud-city-igw}]'
```

Attach it to the VPC:

```powershell
aws ec2 attach-internet-gateway --internet-gateway-id <IGW_ID> --vpc-id <VPC_ID>
```

---

## 6. Create Public Route Table

```powershell
aws ec2 create-route-table --vpc-id <VPC_ID> --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=cloud-city-public-rt}]'
```

Create the internet route:

```powershell
aws ec2 create-route --route-table-id <PUBLIC_ROUTE_TABLE_ID> --destination-cidr-block 0.0.0.0/0 --gateway-id <IGW_ID>
```

Associate the public subnet:

```powershell
aws ec2 associate-route-table --route-table-id <PUBLIC_ROUTE_TABLE_ID> --subnet-id <PUBLIC_SUBNET_ID>
```

---

## 7. Allocate Elastic IP

```powershell
aws ec2 allocate-address --domain vpc
```

---

## 8. Create NAT Gateway

The NAT Gateway must be deployed in the public subnet.

```powershell
aws ec2 create-nat-gateway --subnet-id <PUBLIC_SUBNET_ID> --allocation-id <EIP_ALLOCATION_ID> --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=cloud-city-nat-gateway}]'
```

Check its status:

```powershell
aws ec2 describe-nat-gateways --nat-gateway-ids <NAT_GATEWAY_ID> --query 'NatGateways[0].State' --output text
```

Expected:

`available`

---

## 9. Create Private Route Table

```powershell
aws ec2 create-route-table --vpc-id <VPC_ID> --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=cloud-city-private-rt}]'
```

Create the NAT route:

```powershell
aws ec2 create-route --route-table-id <PRIVATE_ROUTE_TABLE_ID> --destination-cidr-block 0.0.0.0/0 --nat-gateway-id <NAT_GATEWAY_ID>
```

Associate the private subnet:

```powershell
aws ec2 associate-route-table --route-table-id <PRIVATE_ROUTE_TABLE_ID> --subnet-id <PRIVATE_SUBNET_ID>
```

---

## 10. Create Public Security Group

```powershell
aws ec2 create-security-group --group-name cloud-city-public-sg --description "Public EC2 security group" --vpc-id <VPC_ID>
```

Allow HTTP:

```powershell
aws ec2 authorize-security-group-ingress --group-id <PUBLIC_SG_ID> --protocol tcp --port 80 --cidr 0.0.0.0/0
```

Allow SSH only from the administrator's current public IP:

```powershell
aws ec2 authorize-security-group-ingress --group-id <PUBLIC_SG_ID> --protocol tcp --port 22 --cidr <YOUR_PUBLIC_IP>/32
```

---

## 11. Create Private Security Group

```powershell
aws ec2 create-security-group --group-name cloud-city-private-sg --description "Private EC2 security group" --vpc-id <VPC_ID>
```

Allow SSH from resources using the public EC2 Security Group:

```powershell
aws ec2 authorize-security-group-ingress --group-id <PRIVATE_SG_ID> --protocol tcp --port 22 --source-group <PUBLIC_SG_ID>
```

---

## 12. Create EC2 Key Pair

```powershell
aws ec2 create-key-pair --key-name cloud-city-key-v2 --query 'KeyMaterial' --output text | Set-Content -Path .\cloud-city-key-v2.pem -Encoding ascii
```

Verify that the private key file is not empty:

```powershell
(Get-Item .\cloud-city-key-v2.pem).Length
```

> Never commit `.pem` private keys to GitHub.

---

## 13. Launch Public EC2

```powershell
aws ec2 run-instances --image-id <AMI_ID> --instance-type t2.micro --key-name cloud-city-key-v2 --security-group-ids <PUBLIC_SG_ID> --subnet-id <PUBLIC_SUBNET_ID> --associate-public-ip-address --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=cloud-city-public-ec2}]'
```

Verify the instance:

```powershell
aws ec2 describe-instances --instance-ids <PUBLIC_INSTANCE_ID> --query 'Reservations[0].Instances[0].[InstanceId,State.Name,PrivateIpAddress,PublicIpAddress]' --output table
```

---

## 14. Connect to Public EC2

```powershell
ssh -i .\cloud-city-key-v2.pem ec2-user@<PUBLIC_EC2_IP>
```

---

## 15. Install Apache

Run on the public EC2:

```bash
sudo dnf install -y httpd
```

```bash
sudo systemctl enable --now httpd
```

Check the service:

```bash
sudo systemctl status httpd
```

Test from the local machine:

```powershell
curl http://<PUBLIC_EC2_IP>
```

---

## 16. Launch Private EC2

Notice that `--associate-public-ip-address` is intentionally not used.

```powershell
aws ec2 run-instances --image-id <AMI_ID> --instance-type t2.micro --key-name cloud-city-key-v2 --security-group-ids <PRIVATE_SG_ID> --subnet-id <PRIVATE_SUBNET_ID> --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=cloud-city-private-ec2}]'
```

Verify that there is no public IP:

```powershell
aws ec2 describe-instances --instance-ids <PRIVATE_INSTANCE_ID> --query 'Reservations[0].Instances[0].[InstanceId,State.Name,PrivateIpAddress,PublicIpAddress,SubnetId]' --output table
```

Expected:

`PublicIpAddress = None`

---

## 17. Connect to Private EC2 Through Jump Host

The private key remains on the local machine.

```powershell
ssh -i .\cloud-city-key-v2.pem -o "ProxyCommand=ssh -i .\cloud-city-key-v2.pem -W %h:%p ec2-user@<PUBLIC_EC2_IP>" ec2-user@<PRIVATE_EC2_IP>
```

Connection path:

`Local Machine → Public EC2 → Private EC2`

---

## 18. Validate NAT Gateway

Run from the private EC2:

```bash
curl https://checkip.amazonaws.com
```

The returned address should be the public address used for outbound NAT traffic, not the private EC2 address.

Test package repository connectivity:

```bash
sudo dnf check-update
```

---

## 19. Inspect Route Tables

```powershell
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=<VPC_ID>" --output table
```

---

## 20. Inspect Security Groups

```powershell
aws ec2 describe-security-groups --group-ids <PUBLIC_SG_ID> <PRIVATE_SG_ID> --output table
```

---

## Security Notes

````
Note: I intentionally used:

```text
<VPC_ID>
<PUBLIC_SUBNET_ID>
<PRIVATE_SUBNET_ID>
<NAT_GATEWAY_ID>
<PUBLIC_SG_ID>
<PRIVATE_SG_ID>
<PUBLIC_EC2_IP>
````

