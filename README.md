# MATLAB Parallel Server - Private Networking Configuration

## Overview

This guide outlines the networking prerequisites for deploying the MATLAB Parallel Server cluster in an AWS environment without public IP addresses. This is a common security requirement that ensures all cluster traffic remains within your private network.

## Private Networking Deployment

You can deploy the cluster without public IPv4 addresses by setting the `EnablePublicIPAddress` parameter to `No` during stack creation. In this configuration, all communication occurs via private DNS names or private IP addresses.

## Prerequisites

### Network Configuration

- **VPC Requirement**: The head node and worker nodes must be deployed in the same VPC (automatically configured by the CloudFormation template)
- **DNS Resolution**: All cluster instances must be able to resolve each other's private DNS names
- **S3 Access (Required)**: The cluster and head node require access to Amazon S3 for reading/writing secrets, cluster profiles, and configuration files. This must be configured before deployment.
- **Internet Connectivity**: Required for auto-scaling, auto-termination, and shared storage features via NAT Gateway

### DNS Resolution Requirements

#### 1. MATLAB Client Resolution

The MATLAB client (whether on-premises or in AWS) must be able to resolve:
- The head node's private DNS name
- Private DNS names of all worker nodes in the Auto Scaling Group

#### 2. Cluster Internal Resolution

- Head node must be able to resolve private DNS names of all worker nodes
- Worker nodes must be able to resolve other worker nodes' private DNS names

> **Note**: If using the default Amazon-provided DNS (AmazonProvidedDNS), internal DNS resolution is automatically configured. Custom DNS servers require additional configuration (see below).

#### 3. Communication Mode (R2025a and Later)

For MATLAB R2025a and newer releases, you can configure the cluster to use the `CommunicationMode` parameter to communicate via private IPv4 addresses instead of DNS names. This simplifies deployment by eliminating the need to register cluster nodes with custom DNS servers.

### Internet Connectivity

If using any of the following features, ensure cluster subnets have internet access via NAT Gateway:
- Auto-scaling
- Auto-termination
- Shared storage

## S3 Access Configuration (Required)

### Overview

The MATLAB Parallel Server cluster requires access to Amazon S3 to:
- Store and retrieve cluster configuration files
- Manage secrets and credentials
- Store cluster profiles
- Handle job data and intermediate results

**S3 access must be configured before deploying the CloudFormation stack. One way to provide private subnets access to the Amazon S3 service is via [S3 Gateway Endpoint](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html).**

## Custom DNS Server Requirements

### DNS Auto-Registration

When using a custom DNS server in the cluster VPC, you must implement automatic registration and deregistration of A records with the DNS server as instances launch and terminate.

**Example DNS Mapping**:
- DNS Name: `ip-10-0-10-1.mycompany.internal`
- Private IP: `10.0.10.1`

### Why This Matters

Auto-registration of records is critical for:
- **Worker-to-Worker Communication**: Jobs requiring direct worker communication
- **Client-to-Worker Communication**: MATLAB client direct access to specific workers

If the DNS server cannot resolve the private DNS names of the headnode or workers, jobs may fail or hang indefinitely.
