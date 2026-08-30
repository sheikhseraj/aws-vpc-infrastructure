# Infrastructure Cleanup

AWS infrastructure should be removed when it is no longer required to avoid unnecessary charges.

Resources such as NAT Gateways, EC2 instances and public IPv4 addresses may generate costs while provisioned.

## Recommended Cleanup Order

### 1. Terminate EC2 Instances

Terminate:

- Public EC2
- Private EC2

Wait until the instances reach the terminated state.

### 2. Delete NAT Gateway

Delete the NAT Gateway and wait until deletion completes.

### 3. Release Elastic IP

After the NAT Gateway has been deleted, release its Elastic IP address.

### 4. Remove Route Table Associations

Remove custom subnet associations where required.

### 5. Delete Custom Route Tables

Delete:

- Public route table
- Private route table

### 6. Detach Internet Gateway

Detach the Internet Gateway from the VPC.

### 7. Delete Internet Gateway

Delete the detached Internet Gateway.

### 8. Delete Subnets

Delete:

- Public subnet
- Private subnet

### 9. Delete Security Groups

Delete project-specific security groups after all dependent resources have been removed.

## Training Environment

The VPC used by this project was available in the AWS training environment.

It should not be deleted unless the environment permits it and the VPC is not required by other lab resources.

## Important

Always inspect dependencies before deleting AWS resources.

Infrastructure cleanup is part of responsible cloud engineering and cost management.
