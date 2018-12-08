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


### EBS
Can be snapshoted (incrementally). A snapshot can be used to create a new EBS or an AMI

Initialization occurs the first time a storage bloc on the volume is read and performance can be impacted by up to 50%. This can be avoid by manually reading all the blocks

2 types :
- SSD
   - General purpose : Performance based on 3 IOPS/GB. Can burst up to 3000 IOPS (credit based)
   - Provisioned IOPS : Up to 32000 IOPS per volume with a maximum of 80000 IOPS per instance
- Hard Disk Drive
   - Throughput Optimized : 500MB/s
   - Cold HDD : Lower cost but 250MB/s

### Instance storage
Hypervisor local storage. Only exist the duration and can't be snapshoted. In case of instance reboot, it's still maintained

SSD : Only for c3, f1, g2, i3, m3, r3, x1
HDD : Only for h1, d2

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

# VPC
## Design
A VPC is a private network. Each VPC must have an internet gateway attached to it.

A VPC :
- Is attached to only one region
- Spans multiple availability zones in the region
- Contains a DNS server, but you can run your own by changing the DHCP option in the VPC configuration


Private Network features:
- Private and public subnets
- Scalable architecture
- Ability to extend on premise network through VPN
- Ability to configure routes between subnets
- Ability to configure an internet gateway for resources inside the VPC
- Layered security
   - Instance level security : Security Group
   - Subnet level network ACL : Firewall Rules

## Internet Gateway
It's a service that connect our VPC to the Internet
Provides NAT translation

Must be attached to a VPC to provide internet access to instance

## Route tables
Think to add a default route to permit internet access

## VPC Security
### NACLs : Network Access Control List
Operate at subnet level and allow all traffic by default. Attached to a subnet

Stateless, so return traffic need an outbound rule.

Generally not use for general use case. SG is prefered

### Security Group
Security Group are stateful. There is no deny rule to Security Group because there is an implicit deny at the end. Only Allow traffic has to be defined.





# Services
## Elastic Load Balancing (ELB)
An ELB can load balance traffic to instances located across multiple Availibility Zone

ELB can be paired with Auto-scaling
ELB has its own DNS Record set.

An ELB can be public facing (internet) or internal. SSL certification can be applied directly to en ELB

### Classic ELB
Simple load balancing to EC2 instances. No specific rules for routing traffic, ...
Support : TCP, SSL, HTTP and HTTPS

### Application ELB
It's a layer 7 load balancer. It supports target routing based on specific rules (Host, Path, Conten based, ...)
It also supports Access Logs, Sticky session and AWS WAF

Like classic ELB, it's based on instances

### Network LB
Layer 4 load balancer. Designed for extreme performance because it's not based on instances (Can scale very quickly).
Works only inside the same AZ. IP adresses as targets ans no SSL offloading.


## NAT Gateway
Provides internet access to private IP only instances

Must be created in the public subnet and be part of the private subnet route table

## VPC Endpoints
Most of AWS services uses public IP adresses to be joined.

To avoid NAT ou public IP using, you can create an endpoint inside your VPC. It has a link to your private network and a link inside a private AWS network from where it can join AWS services.

2 types :
- Gateway endpoints : S3 and DynamoDB
- Interface endpoints : CloudWatch logs, CodeBuild, KMS, Kinesis, Service Catalog, ...

A IAM Policy can be applied to a VPC endpoint

## Auto-scaling
Based on Cloudwatch metrics

Components :
- Launch configuration : Use EC2 template (AMI, instance type, user-data, storage, security group, ...)
- Auto-scaling group : Min/Max instances, VPC and AZ, Scaling policy, SNS notifications, ...
- Cloudwatch alamrs : Alarms are triggered when metrics exceed thresholds

## Stateless architecture
- Store state information off-instance
   - NoSQL Database (DynamoDB, Redis, ...)
   - Shared Filesystem

## Route 53
DNS as a service

- Domain registration
- DNS service
- Health Checking (Send request over the internet to verify that it's reachable, available and functionnal)

Can be used :
- External DNS
- Internal DNS
- Latency, GEO, basic and failover routing policies allow for region-to-region fault tolerant architecture design
- Can easily configure for failover to S3 (if website is down to use a static version of the website from S3)

Route 53 Hosted Zone : Store DNS records

Record sets:
- Can be : A, AAAA, CNAME, MX
- Alias record sets : Alias to an AWS specific ressource
- Routing policy :
   - Simple : Route all traffic to one endpoint
   - Weighted : Route traffic to multiple endpoints
   - Latency : Route traffic to an endpoint based on the users latency
   - Failover : Route traffic to a secondary endpoint if the primary is unavailable
   - Geolocation ; Route traffic to an endpoint based on the geographical location of the user
- Evaluate health check : Can monitor the health of the application and trigger an action

## CloudFront
Content Delivery CDN.

It contains few services :
- AWS Shield : Anti-DDoS
- AWS WAF : Web Application Firewall
- Lambda@Edge
- S3 Transfert  
- API Gateway

## RDS : Relational Database Service
Fully managed relational Database service. You can't connect to the underlying system.

- Ability to provision/resize hardware on demand for scaling
   - On the fly for Storage
   - Need a reboot for CPU and RAM
- Multi AZ deployments possible for backup and high availability
- OLTP database
- Utilize read replica for MySQL/PosgreSQL/Aurora
    - Supported in cross region
    - Read replica can be promoted primary in case of disaster

Supports :
- MySQL / MariaDB
- PostgreSQL
- Oracle
- MS SQL Server
- Aurora

Benefits :
- Automatic minor updates
- Automatic backups (point in time snapshot)
    - 35 retentions for automatic snapshots
    - Manual snapshots can be retained as long as needed
- PaaS solution
- Multi AZ

### Stand-by instance : RDS Multi AZ failover
RDS instance built in another AZ with synchronous replication.

AWS automatically switch the DNS record to this RDS instance in case of : Service outage, primary DB instance failure, instance server type changed, manual failover, updates

RDS backup are taken against this stand by instance to reduce I/O freezes

Database instance must be launched into a subnet group

### Aurora
AWS own Database fully compatible with MySQL or PostgreSQL (5 times more performant than MySQL and 3 times than PostgreSQL)

Lower price than commercial databases

Features :
- Continous backup to S3
- Up to 15 replicas across 3 AZ
- Replication lag of single digit milliseconds
- Backtrack in seconds -> Database rollback
- Multi master option

### Aurora Serverless

- Autoscaling
- No instance management
- Scales to zero
- Pay as you go (ACUs, Storage, I/O)


## DynamoDB
NoSQL managed database (Document type NoSQL DB)

Built home grown by AWS but similar to MongoDB

Schemaless and use a key-value store

Fully distributed and scales automatically
Built as a fault tolerant and highly available service
    - Full synchronization of the data across all AZ within a region


## Neptune
Fully managed NoSQL graph Databases but not serverless.

Up to 64 To and 15 read replicas


## ElastiCache
Fully managed in-memory data store

Application use API to set and get data

Engines availables in ElastiCache :
- Memcached
    - Simple model
    - Easy to scale
    - Multithread
- Redis
    - Complex data types
    - Multi AZ failover
    - Data persistence
    - Snapshots
    - Encryption

## Redshift
It's an OLAP datawarehouse service fully managed and scalable

It's a fork of postgresSQL. Massive parallel processing (A leader node and compute nodes)

It's serverless and pay as you go


## S3
Need internet access or NAT or VPC endpoint

### Essentials
- A bucket resids only inside one region
- No charge for transfert inside a same region
- Object versioning
- Lifecycle policies
- Read consistency : Object can be immediately available after been put in a bucket
- Eventual consistency for PUTS overwritting an existing object or DELETE

### Components

- Bucket
    - Need a unique name across ALL of AWS
    - Need a DNS compliant name and no underscores
    - Only 100 buckets par account at a time (Soft limit)
- Object
    - Object are spread across the S3 cluster regardings the prefix name -> Do not use the same prefix for all your files to increase performance
    - An object can be encrypted (By aws keys or by a provided key)
    - Organized into sub-name spaces called folder
- Folders
    - Support concept of folders
    - In fact this is a grouping objects

### Main features

- Versioning
- Versioning permit crossed region replication
- Storage classes
    - Standard :
        - Availability of 99.99%
        - Most expensive storage class
        - Default storage option
    - S3-IA (Infrequent Access) :
        - Availability of 99.9%
        - Less expensive than standard class
        - 30 days minimum charge
    - S3 One Zone-IA
        - Availability of 99.5%
        - Less expensive than S3-IA class
        - 30 days minimum charge
    - Glacier
        - Long term storage
        - May take several hours for object to be retrieved
        - 90 days minimum charge
        - Metadata of object that is moved to glacier keep stay in S3
- Lifecycle policies
    - A set of rules that automate the migration of an object storage class to a different class
    - Disabled by default
    - Separate policies for current version and previous versions
- S3 event notifications
    - Can be sent to SNS, Lambda or SQS Queue
    - On object creation (PUT, POST, COPY) or object remove
- S3 permissions
    - 3 levels
        - IAM policy -> Apply to a group or role
        - ACL policy -> Apply to users in other aws account or to the public
        - Bucket policy -> Apply to a particular bucket
- Static web hosting
    - Amazon route 53 maps human readable domain names to static hosting bucket
    - Ideal for failover solution
    - Ideal to store and deliver static part like js, css or image from a web site

### Glacier

It's an archival storage type.
Should NOT be used as a backup solution

Archive are stored in a vault

Vault lock feature -> When locked, data could not be changed for a period of time defined when locking is triggered

Data retrieval (pricing varies)
- Expedited : 1-5 minutes
- Standard : 3-5 hours
- Bulk : 5-12 hours


### Transferring or backuping data to S3

- PUT file
- Multipart upload
    - Upload a single file as sets of parts
    - Allows for uploading parts of a file concurrently
    - Allows for stopping/resuming file uploads
    - Required for objects larger than 5GB
- S3 Transfert acceleration
    - Upload content through CloudFront Edge Location
    - Change endpoint (mybucket.s3.amazonaws.com to mybucket.s3-accelerate.amazonaws.com)

- Snowball
    - Snowball : Transfert Appliance. 80TB per appliance
    - Snowball Edge : Idem to Snowball but with onboarding compute capabilities (S3 API Interface and Lambda functions)
    - Snowmobile : 100PB per snowmobile (A truck)

- Storage Gateway
    - Connect Datacenter to S3
    - VMWare and Hyper-V
    - Volume Gateway :
        - EBS snapshots for disaster recovery
        - Gateway-cached Volumes : Create volumes and mount as iSCSI to on-prem. The gateway stores data into S3
        - Gateway-Stored Volumes : Store all data locally in storage volume. Periodically snapshots as incremental backup to S3
    - File gateway : Local NFS. Objects are stored an retrievable in S3
    - Tape gateway : Emulate iSCSI based virtual tape library for Veeam, Veritas, Arcserve, Dell, ...


## Hybrid environnment
### VPN

It's a IPSec VPN using 2 parallel routes for redundancy.
Only one VPG (Virtual Private Gateway) per VPC

Use static or dynamic routing (BGP)

### AWS Direct Connect

Direct connection to AWS through AWS Direct Connect locations. Provide 1 Gbps or 10 Gbps access.
Each AWS Direct Connect location is associated with a particular region.

### VPC Peering

It's the best way to permit communication between different VPC. Works inside the same region or with different regions.


## SNS (Simple Notification Service)

Coordinates and manage the sending and delivery of messages.

Components :

- Topic (With message <256KB)
- Subscriber : Can be HTTP, HTTPs, Email, SQS, Application, Mobile App notifications, Lambda, SMS
- Publisher : Application, S3 events or Cloudwatch Alarm
    - ! PUSH Only to subscriber

For sending email with attachement and size >256KB, use SES (Send Email Service)

One to many.

## SQS (Simple Queue Service)

Goals : Decoupled architecture -> Asynchronous communication
- Loosely coupled system : Multiple components that can process without being connected (Use message queue)
- Thightly coupled system : Multiple components are dependent upon each other. If one fails, all components fail

Poll only from workers

Functionalities:
- 2 types of polling
    - Short polling : Response immediately if there is a message or not. -> Increase API request, then cost !!!!! TO CHECK
    - Long polling : Allow SQS service to wait until a message is available. (From 1 to 20 seconds) !!!!! TO CHECK
- Visibility timeout : How long the message is visible in the queue
- Auto scaling can be applied based on the queue size
- Larger than 256KB messages must use S3
- 2 differents types of queues :
    - Standard : Guarantees delivery of each message at least once, but not the order
    - FIFO queue : Order and exactly once processing. But limited to 300 transactions per second (3000 if limit increase requested)

One to one.


## Amazon MQ

It's an Apache MQ managed service.
Usefull for compatibility with legacy

Based on EC2 (Single instance or Highly Available) -> Not scalable

Work as SNS and SQS (one to one or one to many)


## SWF (Simple Work Flow)

Fully managed workflow orchestration.
SWF has consistent execution and guarantees the order in which tasks are executed

A workflow execution can last up to 1 year

Components :
- Starter : Start the workflow
- Workflow : Sequence of step
- Activites : A single step
- Task : What interact with the worker
    - Activity task : Tell the worker to perform an action
    - Decision task : Tell the worker to decide
- Worker : Responsible for receiving a task and taking action on it


## API Gateway

Fully managed service.

Features:
- RESTful API
- Deploy to stage (Multiple environnments)
- Rollback to previous API
- Custom domain name support
- Create and manage API key for access AND meter usage through cloudwatch logs
- Set rules based of the number of requests per second (HTTP 429 response above)
- Security using Signature v4 to sign and authorize API calls
    - Temporary credentials through STS service

Benefits:
- Ability to cache API response
- DDoS protection through CloudFront
- SDK generation for Android, Javascript and IOS
- Support swagger (Popular framework for API)
- Request/response data transformation (Json to XML, ...)



## Lambda

Serverless computing plateform service.

- Amount of resources (CPU, Memory) available for a function can be configured
- From millisecond to 5 minutes running function
- Integrated with many other services
- Can be plugged to a VPC (to reach instances inside the VPC)
- Supports
    - Node.JS
    - Java
    - C#
    - Python
    - Go

Generally used in response to an events (triggered by the event)
    - Change in S3 bucket
    - Call to API Gateway
    - Updates to an DynamoDB table
    - Custom events generated by your applications
    - Kinesis Streams




.
