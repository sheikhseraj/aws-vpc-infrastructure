# Infrastructure Validation

The infrastructure was tested after deployment rather than assuming that successful resource creation meant the architecture was working.

## Network Validation

- [x] VPC available
- [x] Public subnet configured
- [x] Private subnet configured
- [x] Internet Gateway attached
- [x] NAT Gateway available
- [x] Public route configured
- [x] Private NAT route configured

## Public EC2 Validation

- [x] EC2 instance running
- [x] Private IP assigned
- [x] Public IP assigned
- [x] SSH connectivity successful
- [x] Apache running
- [x] HTTP port 80 reachable from the internet

## Private EC2 Validation

- [x] EC2 instance running
- [x] Private IP assigned
- [x] No public IP assigned
- [x] Direct internet exposure avoided
- [x] SSH through public jump host successful

## Security Validation

- [x] Public SSH restricted to administrator IP
- [x] HTTP allowed to public web server
- [x] Private SSH allowed from Public EC2 Security Group
- [x] Private EC2 has no public IP
- [x] SSH private key remains on local machine

## NAT Gateway Validation

Outbound internet connectivity was tested from the private EC2 using:

`curl https://checkip.amazonaws.com`

Package repository connectivity was also tested using Amazon Linux package management.

## Final Result

Public workload:

`Internet → IGW → Public Route Table → Public Subnet → Public EC2`

Private outbound workload:

`Private EC2 → Private Route Table → NAT Gateway → IGW → Internet`

Administrative access:

`Local Machine → Public EC2 Jump Host → Private EC2`

All three paths were successfully validated.
