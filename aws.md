# Well architected Framework

5 pillars :
- Operational excellence
   - Operations as code
   - Annotate documentation
   - Make frequent small and reversible change
   - Refine operations procedures frequently
   - Learn from all operation failure
- Reliability
   - Test recovery procedure
   - Automaticly recover from failure
   - Scale horizontaly
   - Stop guessing capacity
   - Automate change
- Security
   - Strong identity fondation
   - Tracability
   - Apply security at every level
   - Automate security
   - Protect data in transit and at rest
   - Prepare for security events
- Performance efficiency
   - Democratize advanced technologie
   - Go global in minute
   - Use serverless architecture
   - Experiment more often
- Cost optimization
   - Adopt consumption model
   - Mesure overall efficiency
   - Stop spending money on datacenter
   - Analyze and attribute expenditures
   - Use managed services






# AWS architecture
## Regions
Each region is independent. The price of services may change regarding the region you use

## Availibility zones
Grouped in a region (18 active regions now) and separated from few miles

## Edge locations
Are not affiliated to a region. They are AWS Datacenter PoP that operates only 2 services:
- Route 53
- CloudFront

## Regional Edge Cache
A frequently accessed data is cached in an Edge location, then when this data is supposed to be removed from the Edge cache location (because lessly used), it is cached in the Regional Edge Location that is present in a region


# Security
## Authentication
Can use a MFA (Multi Factor Authentification) -> Physical or virtual

## IAM
### Principles
It's a global scope service (one service for all regions)

IAM Manage:
- Users
- Groups
- Roles (for servers that wanna use AWS services)

IAM maintain:
- Access Policy
- API Keys (Up to 2 per user)
- Password policy and MFA requirements

For all users except root, permissions must be given

### Best practices
- Delete root access keys
- Activate MFA on root account
- Create and use an IAM user with Admin privileges instead of the root account
- Create individual IAM users
- Use groups to assign permissions
- Principle of Least Privilege
- Apply an IAM password Policy

### Policy
A policy is a json file that formally states one or more permissions.
By default, all permissions are denied
All explicit deny, always overrides an explicit allow
More than one policy can be attached to a user or a group at the same time

IAM provides pre-built policy template

### Role
Role doesn't need pass or store permanent credential and can be associated to a user, a group, a service, Federated users (Active Directory, LDAP, Web Entity)

User must have the PassRole permission.

An EC2 instance can only have ONE role attached at a time

A role has a duration from 15 min to 12 or 36 hours, but EC2 metadata service will keep rotate the role to allow the instance to keep working long time if needed (Using STS)

Can be also used for cross-account : For instance to create a read only access to external auditor.
- Create a role with readonly permission
- Attach it to an external AWS userid
-> The auditor can use my account to readonly without my password

Use access keys, but theses are provided by STS service

### API key
API access key are permanent and need to be removed or rotate manually.

### STS : Security Token Service
Used to provide key automatically.
Best for :
- Identity Federation : Support SAML (used by Active Directory)
- Role for cross-account access
- Role for EC2 or other AWS services

Used when you want to receive temporary credential from IAM

### Identity Federation
Managed by STS service. Authenticate users using :
- Custom Identity Provider
- LDAP / Active Directory - With SAML
- Web Identity - With OpenID

### Organization
AWS organization is composed of a root, then multiple OU (Organisational Unit).
Specific IAM policy can be set to an OU.


# EC2
## EC2 essentials
There is an import service to import VMWare virtual machine to AWS.

AMI = Amazon Machine Image (template)

AWS propose 2 types of virtualizations (Choose the good AMI for the virtualization you want)
- HVM (HardWare Virtual Machine) : Hardware Virtual Machine. Emulate the Bare Metal. No Os modification needed
   -> Recommanded solution
- PV (ParaVirtual) : No need for OS modification, but cannot use enhanced hardware like GPU or specific network tuning

Instance type
- In forme of LetterNumber.Size (T2.medium or G3.large - G for GPU)
- T2 instance type : Don't give a dedicated CPU. It's a shared CPU.
   - Can active T2 unlimited:
      - When you don't use the CPU, you gain credit
       - When you over use the CPU, you can burst it and you loose credit (and be charged if no credit left)

## EC2 storage

Storage for EC2:
- EBS (Elastic Block Store) : Network persistent storage
- Instance store : Instance ephemeral storage (local to hypervisor)
- Elastic File Storage : NFS

EBS : Available on all instance type
SSD : Only for c3, f1, g2, i3, m3, r3, x1
HDD : Only for h1, d2

### EBS
Can be snapshoted (incrementally). A snapshot can be used to create a new EBS or an AMI

Initialization occurs the first time a storage bloc on the volume is read and performance can be impacted by up to 50%. This can be avoid by manually reading all the blocks

2 types :
- SSD
   - General purpose : Performance based on 3 IOPS/GB. Can burst up to 3000 IOPS (credit based)
   - Provisioned IOPS : Up to 32000 IOPS per volume with a maximum of 80000 IOPS per instance
- Hard Disk Drive
   - Throughput Optimized : 500MB/s
   - Clod HDD : Lower cost but 250MB/s

### Instance storage
Hypervisor local storage. Only exist the duration and can't be snapshoted. In case of instance reboot, it's still maintained

### EFS
NFS as a service
Only support NFSv4 (4.0 or 4.1)
Burst network performance up to 100MB/s

Encrypted data at rest using AWS KMS (Key Management Service)


## EC2 Network and security
At least 1 security Group is required to build an instance

Network bandwith and performance depends of the instance type (like CPU number or Memory)

EC2 IP adresses:
- Private
- Public
- EIP : Elastic Public IP Adress -> A fixed public IP adress that can be attached to an Instance

Tenancy:
- Shared
- Dedicated : Use a non shared hypervisor
- Dedicated Host : Use a dedicated host -> Generally used for dedicated software that need specific hardware

## Bootstrapping and user-data/meta-data
### Bootstrapping
Specific command without external input during the creation process

### User-data
Basch script for your own command (install httpd for instance)

### User-Data/Meta-data
Can be view through REST API:
- curl http://169.254.169.254/latest/user-data/
- curl http://169.254.169.254/latest/meta-data/

## Buying option
- On demand : Per second pricing with a minimum au 60 seconds (Amazon Linux, Ubuntu, ...) or per hour pricing (Windows, RHEL, ...)
- Reserved : For 1 to 3 years
   - Standard, convertible (can change/upgrade), scheduled (1y term)
- Spot instances :
   - Spot price fluctuate according to the demand
   - Instances launch when spot price is less than you maximum price
   - Automatically terminate or hibernate when spot price exceeds your maximum (2 minutes warning)
   - You can use spot blocks with a lower discount but specify the duration (Up to 6 hours)

## Placement group
- Cluster placement group on the same host or close host
- Spread placement group : On distinct host hardware

Troubleshooting :
If an instance in a placement group is stopped, it continues to be a member of the placement once it started again. It can have a placement problem "insufficient capacity". That's why AWS suggest launching all the required instances within the the placement group in a single request


# Services
## Route 53
DNS

## CloudFront
Content Delivery CDN.

It contains few services :
- AWS Shield : Anti-DDoS
- AWS WAF : Web Application Firewall
- Lambda@Edge
- S3 Transfert  
- API Gateway
