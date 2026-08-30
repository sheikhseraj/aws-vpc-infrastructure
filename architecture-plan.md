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

Architecture planned. Infrastructure implementation not started yet.
