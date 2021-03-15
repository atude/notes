## IAM (Identity Access Management)

- **Roles**: Can be used to assign permissions against users, apps or services to interact with other AWS resources
- **IAM Policy**: Permission definitions attached to users, groups or roles

## EC2 (Elastic Compute Cloud)

- PAYG VM stored on AWS
- **Payment Options**:
  - **On demand**: Pay by the hour - for flexible, unpredicatable workloads
  - **Reserved**: Reserved capacity for 1 or 3 years with discount on hourly charge - for steady and predicatble workloads
  - **Spot**: Use unused capacity - good for short term, high load work - cheap (with changing prices based on demand) but unstable 
  - **Dedicated hosts**: Physical dedicated server - good for compliance or licensing which requires dedicated server (i.e. no multi-tennancy)
- Pay up front or commit for 1/3 years for discounts

## EBS (Elastic Block Store)

- Storage volumes attached to EC2
- **AMI**: Amazon Machine Image - The OS to use on the EC2 instance
- **EBS Volume Types**
  - **gp2** - General purpose SSD	
    - 3 IOPS/GiB
    - Generally used for boot disks
  - **io1/io2** - Provisioned IOPS SSD
    - 50 IOPS/GiB
    - More expensive; good for I/O intensive applications or latency sensitive work
    - io2 is the latest version with more durability, more IOPS and same price
  - **st1** - Throughput Optimised HDD 
    - Low-cost 
    - Good for frequently accessed large data workloads (data warehouses, big data)
  - **sc1** - Cold HDD
    - Cheapest option - poor performance, good for rarely accessed data

- Volumes can be created from previously created volumes (**snapshots**) 
  - Volumes created from encrypted snapshots will be encrypted and vice versa

## ELB (Elastic Load Balancer)

- **Load Balancer**: Distributes network traffic across servers
  - Can detect failing servers and prevent routing traffic to it
  - Can increase capacity to direct traffic to more servers when required
- **Types of load balancers**:
  - **Application Load Balancer**: HTTP/HTTPS
    - Routes requests based on HTTP headers and type of request
  - **Network Load Balancer**: TCP - High performance
    - High performance for low latencies, but expensive
  - **Classic Load Balancer**: HTTP/HTTPS and TCP - Legacy
    - Supports some HTTP headers (e.g. `X-Forwarded-For`)
- **`X-Forwarded-For` Header**: Includes source IPv4 address used to verify request origination
- **Gateway Timeout Error (504)**: 
  - May be caused by ELB unable to connect to service/app or crash in apps.

## Route 53

- DNS service used to map domain names to EC2 instances, load balancers, or S3 buckets
- Can buy/register new domain names
- **Hosted zones**: List of registered domain names

## RDS (Relational Database Service)

- **Online Transaction Processing** (**OLTP**): Processes data from transactions in real time
  - Used for processing large number of small transactions in real time
  - Used with RDS
- **Online Analytics Processing** (**OLAP**): Processes queries to analyse data
  - Used for data analysis of large amounts of data that takes a long time to process
  - Not used by RDS (used by services such as Redshift)
- **Multi-AZ**: An exact copy of the database in another availability zone (**AZ**)
  - A **standby instance** in an inaccesible, invisible copy of the database that acts as a backup
  - Writes to a prod DB automatically syncs across standby DBs
  - Used for disaster recovery through automatic failover; not a means of scaling
- **Read Replica**: A read-only copy of a primary DB used to reduce read workload on primary DB
  - Hosted on different DNS endpoint
  - Can be promoted to an independent DB (by losing read replica functionality)
  - Requires automatic backup 
  - Can have up to 5 replicas for each DB instance
- **Database snapshot**: A manual backup initiated by user stored in a S3 bucket
  - No retention period - not automatically deleted
- **Automated backup**: Create daily backups and DB transaction logs during a specified time window stored in a S3 bucket
  - Can recover a DB at any point in time within retention period specified by user (1 - 35 days)
  - Recovery process restores snapshot from S3 bucket and replays transaction logs to restore DB to that state
  - Free storage space for backups equal to size of the RDS instance
  - May temporarily hinder latency during backup process
- Encryption can only be enabled when the DB is first created
  - To encrypt an existing DB, take a snapshot and encrypt it and restore that snapshot
- Restored versions of RDS database will be a new instance with a new DNS endpoint
- When connecting to an EC2 instance, **inbound rules** for the RDS security group must be specified to allow the EC2 instance to access RDS

## Elasticache

- A key-value in-memory cache used to retrieve data quickly by caching DB queries and session data
- ElastiCache is useful for low frequency changing data with heavy read loads using OLTP.
- **Types of caches**:
  - **Memcache**: Basic object caching without persistence, Multi-AZ or failover
    - Scales horizontally
  - **Redis**: Supports persistence, replication, Multi-AZ and failover
    - Used for data sorting and ranking (e.g. leaderboards)
    - Supports complex data type caching

## Systems Manager Parameter Store

- Used to store secrets and config data with optional encryption
- Can be integrated with AWS services or referenced by paramater name in bootstrap scripts
- Free option has 10k parameters with 4KB parameters and no parameter policies
- Paid advanced option available for storing 10k+ parameters with 8KB paramaters and parameter policies
- 