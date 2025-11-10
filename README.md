# Private Networking Requirements for MATLAB Parallel Server Cluster

## Overview

You can deploy the MATLAB Parallel Server cluster and head-node without Public IPv4 addresses by setting the `EnablePublicIPAddress` parameter to `No` during stack creation. In this setting, the head-node and the worker nodes are configured to communicate with each other and the MATLAB client via their private DNS names.

## Prerequisites

Before deploying the template, ensure that the following requirements are met:

- The head-node and worker nodes (ASG cluster) must be deployed in the same VPC. This is done by default by the CloudFormation template.
- All instances in the cluster , including the head-node, must be able to resolve each other's private DNS names
- For features like auto-scaling, auto-termination, or shared storage, cluster subnets must have internet connectivity via a NAT Gateway to reach AWS service endpoints

### DNS Resolution Requirements

#### 1. MATLAB Client DNS Resolution
The MATLAB Client (whether on-premises or in AWS) must be able to resolve the private DNS names of:
- The head-node instance
- All worker node instances in the Auto Scaling Group

#### 2. Cluster Internal DNS Resolution
- The head-node must be able to resolve the private DNS names of all worker nodes
- Worker nodes must be able to resolve the private DNS names of other worker nodes in the cluster

**Note:** If your VPC uses the default Amazon-provided DNS server (AmazonProvidedDNS), internal cluster DNS resolution is automatically satisfied. However, if you are using a custom DNS server with your VPC, you must configure it to resolve these private DNS names.

For MATLAB R2025a and newer releases, the cluster can be configured to communicate via Private IPv4 addresses via the `CommunicationMode` parameter, instead of DNS names. This relaxes the requirement of registring the cluster nodes with custom DNS servers since the MATLAB Client can now connect to the head-node and worker nodes via their Private IP addresses.

### 3. Internet Connectivity
If you are using auto-scaling, auto-termination, or shared storage features, ensure that:
- Cluster subnets are connected to the internet via a NAT Gateway
- Security groups allow outbound traffic to AWS service endpoints

## Requirements for VPCs with Custom DNS Servers

### DNS Auto-Registration Setup

When using a custom DNS server, you must implement a mechanism to automatically register and deregister A records (that maps the private DNS names to corresponding private IPs, e.g., ip-10-0-10-1.mycompany.internal maps to 10.0.10.1) of the instances with your DNS server as they launch and terminate.

This is required for jobs where the workers must communicate with each other and the MATLAB Client must directly communicate with one or more workers.

If the DNS server cannot resolve the private DNS names of the workers, the job might fail or get stuck indefinitely.
