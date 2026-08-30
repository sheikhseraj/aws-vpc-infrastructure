# Implementation Guide

This document describes the implementation sequence used to build the AWS VPC infrastructure project.

## 1. Network Environment

The project uses a VPC with the following CIDR range:

`10.0.0.0/16`

The VPC was available in the AWS training environment.

## 2. Create the Subnets

Two subnets were configured:

### Public Subnet

`10.0.1.0/24`

Used for:

- Public EC2
- NAT Gateway

Public IPv4 assignment was enabled for resources requiring direct internet connectivity.

### Private Subnet

`10.0.2.0/24`

Used for:

- Private EC2

The private EC2 was deployed without a public IP address.

## 3. Configure Internet Connectivity

An Internet Gateway was attached to the VPC.

The public route table contains:

`0.0.0.0/0 → Internet Gateway`

This route enables internet connectivity for resources in the public subnet.

## 4. Configure Private Outbound Connectivity

An Elastic IP was allocated for the NAT Gateway.

The NAT Gateway was deployed inside the public subnet.

The private route table contains:

`0.0.0.0/0 → NAT Gateway`

This enables resources in the private subnet to initiate outbound internet connections without receiving public IP addresses.

## 5. Configure Security Groups

### Public EC2 Security Group

Inbound:

- HTTP TCP 80 from `0.0.0.0/0`
- SSH TCP 22 restricted to the administrator's public IP

### Private EC2 Security Group

Inbound:

- SSH TCP 22 from the Public EC2 Security Group

The private EC2 does not accept SSH connections directly from the internet.

## 6. Deploy Public EC2

An Amazon Linux 2023 EC2 instance was deployed in the public subnet.

Apache HTTP Server was installed and started.

The web server was validated from an external client.

## 7. Deploy Private EC2

A second Amazon Linux 2023 EC2 instance was deployed in the private subnet.

The instance:

- has a private IP address
- has no public IP address
- uses the private security group

## 8. Configure Administrative Access

The public EC2 was used as an SSH jump host to access the private EC2.

The SSH private key remained on the local machine and was not copied to the public EC2 instance.

Connection pattern:

`Local Machine → Public EC2 → Private EC2`

## 9. Validate NAT Connectivity

Outbound internet connectivity from the private EC2 was tested using:

`curl https://checkip.amazonaws.com`

Amazon Linux repository connectivity was also tested.

## 10. Result

The architecture successfully demonstrated:

- public internet connectivity
- isolated private compute
- controlled administrative access
- NAT-based outbound connectivity
- Security Group-based access control
- AWS routing
- infrastructure troubleshooting
