# AWS VPC Infrastructure — Architecture Plan

## Project Goal

Build a secure AWS network architecture with public and private subnets and deploy EC2 instances to understand routing, internet connectivity, security controls and troubleshooting.

## Planned Architecture

- VPC: `10.0.0.0/16`
- Public Subnet: `10.0.1.0/24`
- Private Subnet: `10.0.2.0/24`
- Internet Gateway
- NAT Gateway
- Public Route Table
- Private Route Table
- Public EC2 Instance
- Private EC2 Instance
- Security Groups

## Traffic Flow

Public EC2:

Internet → Internet Gateway → Public Route Table → Public Subnet → EC2

Private EC2 outbound access:

Private EC2 → Private Route Table → NAT Gateway → Internet Gateway → Internet

## Project Validation

The infrastructure will be validated using:

- AWS CLI
- SSH
- curl
- ping where applicable
- route table inspection
- security group inspection
- connectivity troubleshooting

## Project Status

Infrastructure implementation completed and validated.

### Implemented

- VPC available in the AWS training environment
- Public subnet: `10.0.1.0/24`
- Private subnet: `10.0.2.0/24`
- Internet Gateway attached to the VPC
- NAT Gateway deployed in the public subnet
- Public route table configured with internet access through the Internet Gateway
- Private route table configured with outbound internet access through the NAT Gateway
- Public EC2 instance deployed in the public subnet
- Private EC2 instance deployed without a public IP
- Security Groups configured for controlled SSH and HTTP access
- Apache web server installed and running on the public EC2

### Validated

- SSH access from the local machine to the public EC2
- HTTP access to the Apache web server from the internet
- Private EC2 has no public IP address
- SSH access to the private EC2 through the public EC2 as a jump host
- Outbound internet connectivity from the private EC2 through the NAT Gateway
- Amazon Linux repository connectivity from the private EC2
