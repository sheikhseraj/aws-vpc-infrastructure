# Troubleshooting

This document records real issues encountered while building and validating the AWS VPC infrastructure project.

## 1. VPC Creation Permission Denied

### Problem

Creating a VPC with AWS CLI returned an `UnauthorizedOperation` error.

### Cause

The AWS training account did not allow the current IAM user to perform `ec2:CreateVpc`.

### Resolution

Instead of claiming that the VPC was created manually, the existing VPC available in the training environment was used for the project.

### Lesson Learned

Authentication and authorization are different.

The AWS CLI can be correctly authenticated while the IAM user still does not have permission to perform a specific action.

---

## 2. Duplicate Security Group Rule

### Problem

While configuring inbound rules, AWS returned:

`InvalidPermission.Duplicate`

### Cause

The same security group rule had already been created earlier.

### Resolution

The existing security group rules were inspected instead of creating the same rule again.

### Lesson Learned

Before adding a new rule, verify the current security group configuration to avoid duplicate entries.

---

## 3. SSH Private Key Invalid Format

### Problem

SSH to the public EC2 instance failed with:

`Load key ".\\cloud-city-key.pem": invalid format`

and:

`Permission denied (publickey...)`

### Cause

The local `.pem` file was empty and therefore did not contain a valid EC2 private key.

### Resolution

A new EC2 key pair was created and the private key was saved correctly.

The new key file was verified before launching a replacement EC2 instance.

### Validation

The new key file:

- contained a valid RSA private key header
- had a non-zero file size
- successfully authenticated to the EC2 instance

### Lesson Learned

Always verify a downloaded or generated private key before depending on it for EC2 access.

Never commit `.pem` private keys to GitHub.

---

## 4. SSH Through the Public EC2 Jump Host

### Problem

Connecting to the private EC2 using the basic `ssh -J` command failed because the SSH key was not being applied correctly to the jump-host connection.

### Resolution

An explicit SSH `ProxyCommand` was used so the private key was applied to both the public and private EC2 connections.

### Result

The private EC2 was successfully accessed through the public EC2 without copying the private key onto the public server.

### Lesson Learned

A jump host can provide administrative access to private resources without exposing those resources directly to the internet.

---

## 5. Private EC2 Internet Connectivity

### Validation

The private EC2 had:

- a private IP address
- no public IP address
- outbound internet connectivity through the NAT Gateway

Connectivity was validated using:

`curl https://checkip.amazonaws.com`

and Amazon Linux repository access.

### Result

The private EC2 successfully reached the internet while remaining inaccessible directly from the public internet.

### Lesson Learned

A NAT Gateway allows private resources to initiate outbound internet connections without assigning them public IP addresses.