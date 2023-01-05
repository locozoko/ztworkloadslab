# Zero Trust for Workloads (ZPA) Hands-On Lab

The Zscaler Hands-On Lab uses these AWS CloudFormation Templates to build out the required lab infrastructure. This includes VPCs, subnets, NAT and 
Internet Gateways, Route Tables, App and Workload EC2 instances, and Zscaler components. 

## Table of Contents

- [Prerequisites](#prerequisites)
- [Lab Instructions](#lab-instructions)
  - [Create a ZPA App Connnector Provisioning Key](#create-a-zpa-app-connector-provisioning-key)
  - [Deploy the Application VPC Resources](#deploy-the-application-vpc-resources)
  - [Deploy the Workload VPC Resources](#deploy-the-workload-vpc-resources)
  - [Deploy the Cloud Connector Resources](#deploy-the-cloud-connector-resources)
  - [Verify No Connectivity Between VPCs](#verify-no-connectivity-between-vpcs)
  - [Configure ZPA Application Segments](#configure-zpa-application-segments)
  - [Configure ZPA Access Policy](#configure-zpa-access-policy)
  - [Configure ZPA Client Forwarding Policy](#configure-zpa-client-forwarding-policy)
  - [Configure Cloud Connector Traffic Forwarding Policy](#configure-cloud-connector-traffic-forwarding-policy)
  - [Configure Workloads to Route to Cloud Connector](#configure-workloads-to-route-to-cloud-connector)
  - [Deploy AWS DNS Resolvers for Lab Domain](#deploy-aws-dns-resolvers-for-lab-domain)
  - [Test Workload to Private App Access](#test-workload-to-private-app-access)
  - [Tear Down Load](#tear-down-lab)

# Prerequisites
<sup>[(Back to top)](#table-of-contents)</sup>

## Git

- Clone this repo to obtain all the required CloudFormation Templates. 
- You can also manually download this repo by clicking the green "Repo" button > Download ZIP.

## AWS Account

- An AWS Account is required to deploy this lab. 
- The templates use the smallest instances sizes possible for minimal costs.

## AWS EC2 SSH Key

- All the templates that deploy EC2 instances require an existing AWS Key Pair to be selected. 
- Please create a Key Pair prior to starting this lab or use an exising Key Pair.
- This can be done in AWS > EC2 > Key Pairs > Create key pair
- [For help, see this AWS KB Article on Creating Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)

## Cloud Connector

- All Cloud Connector prerequisites should be completed as normal. This is out of scope for the lab.
- The Cloud Connector tenant must have the ZPA-SERVER-PRE SKU enabled
- [For help, see this Cloud Connector AWS Prerequistes Guide](https://docs.google.com/document/d/1Fvo151a_bcj-p8xX7y3FOeh2uKhXxf5SQJ9c7mzWNv8/edit)
- [For more help, see the Cloud Connector Help Portal on this topic](https://help.zscaler.com/cloud-connector/deploying-cloud-connector-amazon-web-services)

## Zscaler Private Access

- A Zscaler Private Access (ZPA) tenant is required with administrative privileges

# Lab Instructions
<sup>[(Back to top)](#table-of-contents)</sup>

## Create a ZPA App Connnector Provisioning Key
<sup>[(Back to top)](#table-of-contents)</sup>

1. Log into the ZPA Admin Console [https://admin.private.zscaler.com](https://admin.private.zscaler.com)
1. Navigate to Administration > App Connectors
1. Click Add App Connector
1. Select Create New Provisioning Key and click Next
1. Select the Connector certificate and click Next
1. Select Add App Connector Group and fill out the following:
    * Name: ZT for Workloads AWS Lab
    * Scroll down to the Map, search for "Northern Virginia, VA, USA" for the Location
1. Click Next
1. Provide the following App Connector information:
    * Name: ZT for Workloads AWS Lab Connectors
    * Maximum Reuse of Provisioning Key: 1
1. Click Next
1. Click Save
1. Copy the App Connector Provisioning Key
    > **_NOTE:_** You will need this in the next section
    > **_NOTE:_** If you forget to copy the key, you can navigate to App Connector Provisioning Keys to copy it again later
1. Click Done

## Deploy the Application VPC Resources
<sup>[(Back to top)](#table-of-contents)</sup>

1. Log into your AWS Account
1. Change to the following region: *US East (N. Virginia) us-east-1*. 
    > **_NOTE:_** You can use other regions but this lab guide and CloudFormationtemplates were built in the us-east-1 region so I know it works.
1. Navigate to the CloudFormation service
1. Click Create Stack > With new resources (Standard)
1. Select the Upload a template file option
1. Click Choose file
1. Navigate to the directory where you cloned/downloaded the CFTs in this repo
1. Select the file: 1-Create_Application_VPC.yaml
1. Click Next
    * Provide a stack name such as: ZPACCLAB-APPVPC
    * Fill out the My IP Address field with your Public IP Address (this can be found at ip.zscaler.com, ipchicken.com, etc). 
        > **_NOTE:_** This is used to lock down SSH Access to the Public Bastion Host.
    * Select your Key Pair in the EC2 Key Pair field
    * Leave the default environment name
    * Paste your ZPA App Connector Provisioning Key into the ProvisioningKey field
    * Type a domain name to use for the lab in the Domain field. This will create a private DNS Zone only so you can use any domain you want
    * Select the region you are deploy the resources into. It must be the same region you are currently using within the AWS admin console
1. Click Next
1. Click Add new tag
    * Key: Owner
    * Value: your_first_name
1. Click Next
1. Check the box for I acknowledge that AWS CloudFormation might create IAM resources
1. Click Submit
1. Wait until the CloudFormation Stack Status changes from CREATE_IN_PROGRESS (blue) to CREATE_COMPLETE (green) before continuing
    > **_NOTE:_** This can take serveral minutes to complete
1. TEST 123



## Deploy the Workload VPC Resources
<sup>[(Back to top)](#table-of-contents)</sup>

1. Log into your AWS Account
1. Change to the following region: *US East (N. Virginia) us-east-1*. 
    > **_NOTE:_** You can use other regions but this lab guide and CloudFormationtemplates were built in the us-east-1 region so I know it works.
1. Navigate to the CloudFormation service
1. Click Create Stack > With new resources (Standard)
1. Select the Upload a template file option
1. Click Choose file
1. Navigate to the directory where you cloned/downloaded the CFTs in this repo
1. Select the file: 2-Create_Workload_VPC.yaml
1. Click Next
    * Provide a stack name such as: ZPACCLAB-WKLDVPC
    * Fill out the My IP Address field with your Public IP Address (this can be found at ip.zscaler.com, ipchicken.com, etc). 
        > **_NOTE:_** This is used to lock down SSH Access to the Public Bastion Host.
    * Select your Key Pair in the EC2 Key Pair field
    * Leave the default environment name
    * Paste your ZPA App Connector Provisioning Key into the ProvisioningKey field
    * Type a domain name to use for the lab in the Domain field. This will create a private DNS Zone only so you can use any domain you want
    * Select the region you are deploy the resources into. It must be the same region you are currently using within the AWS admin console
1. Click Next
1. Click Add new tag
    * Key: Owner
    * Value: your_first_name
1. Click Next
1. Check the box for I acknowledge that AWS CloudFormation might create IAM resources
1. Click Submit
1. Wait until the CloudFormation Stack Status changes from CREATE_IN_PROGRESS (blue) to CREATE_COMPLETE (green) before continuing
    > **_NOTE:_** This can take serveral minutes to complete
1. TEST 123

## Deploy the Cloud Connector Resources
<sup>[(Back to top)](#table-of-contents)</sup>

## Verify No Connectivity between VPCs
<sup>[(Back to top)](#table-of-contents)</sup>

## Configure ZPA Application Segments
<sup>[(Back to top)](#table-of-contents)</sup>

## Configure ZPA Access Policy
<sup>[(Back to top)](#table-of-contents)</sup>

## Configure ZPA Client Forwarding Policy
<sup>[(Back to top)](#table-of-contents)</sup>

## Configure Cloud Connector Traffic Forwarding Policy
<sup>[(Back to top)](#table-of-contents)</sup>

## Configure Workloads to Route to Cloud Connector
<sup>[(Back to top)](#table-of-contents)</sup>

## Deploy AWS DNS Resolvers for Lab Domain
<sup>[(Back to top)](#table-of-contents)</sup>

## Test Workload to Private App Access
<sup>[(Back to top)](#table-of-contents)</sup>

## Tear Down Lab
<sup>[(Back to top)](#table-of-contents)</sup>