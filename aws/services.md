## IAM (Identity Access Management)

- **Roles**: Can be used to assign permissions against users, apps or services to interact with other AWS resources
- **IAM Policy**: Permission definitions attached to users, groups or roles
  - **AWS Managed Policies**: IAM policies defined by AWS
  - **Customer Managed Policies**: Policies created inside an AWS account that can be applied to services within that account
  - **Inline Policies**: A policy embedded within a single user, role or group; strictly attached to the user/role/group
- **Web Identity Federation**: Auth with external providers - receive authentication code that can be traded with AWS creds

## Cognito

- Provides temp creds through web federation and mapped to an IAM role for access to AWS resources
- Data synced across user devices
- **User Pool**: Directories to manage sign-up/sign-in
- **Identity Pool**: Takes a user's web JWT and provides temp AWS creds
- Uses Push Synchronisation with SNS to update user data across all devices

## Security Token Service (STS)

- `assume-role-with-web-identity-api`: Return temp creds for authenticated users
  - Usually for web apps; Cognito recommended for mobile apps

## EC2 (Elastic Compute Cloud)

- PAYG VM stored on AWS
- **Payment Options**:
  - **On demand**: Pay by the hour - for flexible, unpredicatable workloads
  - **Reserved**: Reserved capacity for 1 or 3 years with discount on hourly charge - for steady and predicatble workloads
  - **Spot**: Use unused capacity - good for short term, high load work - cheap (with changing prices based on demand) but unstable 
  - **Dedicated hosts**: Physical dedicated server - good for compliance or licensing which requires dedicated server (i.e. no multi-tennancy)
- Pay up front or commit for 1/3 years for discounts
- **Encryption**:
  -  HTTPS connections on EC2 instances to ensure end-to-end data encryption
  - To run a decrypt operation on files encrypted with KMS, the EC2 instance must have an instance role for the decrypt operation

## Elastic Container Service (ECS)

- Manages running docker containers on a group of EC2 instances

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
    - Supports routing of access logs into an S3 bucket
  - **Network Load Balancer**: TCP - High performance
    - High performance for low latencies, but expensive
  - **Classic Load Balancer**: HTTP/HTTPS and TCP - Legacy
    - Supports some HTTP headers (e.g. `X-Forwarded-For`)
- **`X-Forwarded-For` Header**: Includes IPv4 address of destination user
- **Gateway Timeout Error (504)**: 
  - May be caused by ELB unable to connect to service/app or crash in apps

## Route 53

- DNS service used to map domain names to EC2 instances, load balancers, or S3 buckets
- Can buy/register new domain names
- **Hosted zones**: Container of DNS records for a domain
- **Alias**: Used to route traffic at the top of a DNS namespace to an AWS resource
- **A Record**: Route traffic to a resource with an IPv4 address

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

## Secrets Manager

- Used to securely store, retrieve, and automatically rotate **database credentials**

## S3 (Simple Storage Service)

- S3 is an object-based key-value store, with the object filename as the **key**
  - Objects also have a **version ID** and **metadata**
  - Objects can be up to 5TB
  - Largest object upload size via a single PUT request is 5GB
  - Recommended to use **multi-part** uploads for files >100MB
- S3 bucket names are globally unique
- S3 URLs are structured as `https://<bucket-name>.s3.<region>.amazonaws.com/<filename/key>`
- **Lifecycle management**: Can automatically transition objects to a cheaper S3 tier and delete objects when unused
- **Storage Tiers**:
  - **Standard**:
    - 99.9% availability, 99.99... % durability (11 9's)
    - Available in at least 3 AZs
    - Designed for frequent access; suitable for websites, games, content distribution, etc
    - Most expensive option but no retrieval fee
  - **Standard-IA (Infrequent Access)**:
    - Same availability/durability as S3 standard
    - Used for less frequent used data but needs quick access
    - Pay to access data per GB
    - Designed for long term storage and backups
  - **One Zone - IA**
    - Similar to S3 IA but in one AZ only and with 99.5% availability
    - 20% cheaper than S3 IA
    - Designed for infrequently accessed, non-critical data
  - **Glacier**:
    - Same availability/durability as standard
    - Cheap, has low fee to access data
    - Designed for infrequently accessed data or archives
    - Retrieval can take 1m to 12hrs
  - **Glacier Deep Archive**:
    -  Same characteristics as Glacier but taskes 12hrs to retrieve files
  - **Intelligent Tiering**:
    - Same availability/durability as standard
    - Automatically moves data to most cost effective tier based on how frequently objects are accessed
    - Also has small monthly fee as well as access fee
- All buckets created private by default
- **Bucket policies**: Access controls over a bucket 
- **Access Control Lists (ACLs)**: Which AWS accounts or groups have access and what type of access over an S3 object
- **Access logs**: Logs all requests made to a buckets and can store these logs to another bucket
  - Disabled by default
- Files can be encrypted via:
  - **In-transit** through SSL/TLS or HTTPS
  - **Server Side Encryption** through S3 or KMS keys
    - `x-amz-server-side-encryption` header can be used for SSE
      - Header will trigger S3 to encrypt object at upload time
      - Set to either `AES-256` (S3 managed keys) or `aws:kms` (KMS managed keys)
  - **Client side encryption** as encrypted by the user before upload
- Encryption can be enforced by:
  - **AWS console** during bucket creation
  - Enabling default encryption on the bucket for server-side encryption
  - A **bucket policy** by denying all PUT requests without an SSE encryption header
    - Can be done in **policy generator** by adding a condition 
- S3 can be used to host static websites
  - Assets can be referenced across buckets if **CORS** is configured by adding the endpoint in the `AllowedOrigin` setting
- Use pre-signed URLs to grant temporary access to an object within a bucket, even if it is private
- 

## CloudFront

- A CDN that distributes content from the **origin** to dispersed **edge locations** that can cache the content
  - Origin can be an S3 bucket, EC2 instance, load balancer, etc
  - Edge locations are **writeable** - utilised by S3 Transfer Acceleration to reduce upload latency
- Default cache TTL is. 1 day
  - Can be manually cleared but will be charged a fee
- Use signed URLs or signed cookies to restrict viewer access (e.g. for paid content)
- Change the **Viewer Protocol Policy** to enable HTTPS requests

## Lambda

- An **alias** is a pointer to point to a specific version of a lambda function 
  - Aliases can be referenced via the ARN (`arn:aws:...:function:<lambda-name>:<version-name>`) 
  - *$LATEST* is the initial and latest version of the function
  - Lamba aliases will not automatically use new code when you upload new code
- By default is limited to **1000** concurrent executions per region
  - Will return 429 status code with *TooManyRequestsException* error
  - Can request an increase by contacting AWS support centre
  - Use **reserved concurrency** to guarantee a set amount of executions
- Lambda can connect to a VPC if required
  - It requires the **private subnet ID** and **security group ID with access** to setup an **Elastic Network Interface** (ENI) to interface with the private subnet
  - You can do this via CLI with the `--vpc-config` flag

## Step Functions

- Visual interface to build serverless applications as a series of steps
- Includes sequencing, error handling, retry logic and branching workflows
- Logs the state of each step to make debugging easy

## X-Ray

- Helps developers analyse and debug distributed apps
- Automatically captures metadata of API calls made to AWS services in apps that use AWS SDK
- **Service Map**: Visual representation of an app from X-Ray
- Requires X-Ray **SDK** and X-Ray **Daemon** to be configured correctly
  - On **EC2**, the daemon should be installed on the instance
  - On **ECS**, the daemon should be installed in a separate Docker container alongside the app container
- **Annotations**: Key-value pairs that can be used to search for, group and filter traces

## API Gateway

- Can import an existing API using a definition file in OpenAPI (i.e. Swagger) format
- API Gateway can be used for legacy services such as SOAP through a web service passthrough or by converting the XML responses to JSON
- **API Gateway Caching** can be enabled to cache endpoints with a default TTL of 300s
- **API Gateway Throttling**:
  - Limits steady-state request rate to 10,000 requests per second (per region)
  - Limits concurrent requests to 5000 requests across all APIs (per region)
  - If it receives 10,000 requests but no more immediately, it will process 5000 requests first concurrently, then the other 5000 within a one second period without error
  - Otherwise, it will return 429 *TooManyRequests*
  - Limits can be increased through support
- Use **Stage Variables** (key-value pairs) based on the API deployment stage to interact with different endpoints

## DynamoDB

- Supports JSON, HTML and XML

- Data is spread around 3 data centres

- By default uses **eventually consistent reads** (takes 1s for all writes to be reflected among all locations)

  - Can choose **strongly consistent reads** for guaranteed consistency

- **DynamoDB** supports **ACID** transactions

- **Keys**:

  - **Primary Key**: A unique key used as an index to query on
  - **Secondary Key**: 
    - Enables faster queries compared to using a standalone primary key and enable viewing data in a different perspective
    - **Local Secondary Index**: A key that uses the same partition key but a different sort key
      - Can only be created when the table is created; cannot be modified or deleted
    - **Global Secondary Index**: A key that uses a different partition and sort key
      - Can be created at any time

  - **Partition Key**: An attribute based key that can be used as a primary key if unique
  - **Composite Key**: A partition key combined with a sort key that can be a primary key if the partition key is not unique

- IAM policy **conditions** can be used to restrict specific items in a table based on the key (`dynamodb:LeadingKeys`)

- Query results are always sorted in ascending order by the sort key (if set)

  - Use the `ScanIndexForward` to reverse the order of query results

- Use the `ProjectionExpression` paramater to define which attributes should be returned for a query/scan

- **Parallel scanning** can be used to improve scan performance by segmenting tables

- **Write capacity unit**: 1 x 1KB write per second

- **Read capacity unit**: 1 x 4KB strongly consistent reads per second or 2 x 4KB eventually consistent reads per second

- **DynamoDB Accelerator (DAX)**: Fully managed in memory cache for DynamoDB reads - up to 10x read performance

  - Only works for eventually consistent read applications
  - Not beneficial for write intensive tables or low-frequency read tables
  - Only required if extremely low-latency reads are needed
  - API calls should be pointed at the DAX cluster instead of DynamoDB

- Old or redundant data can be automatically deleted by using a TTL assigned to a timestamp attribute to save costs

- **DynamoDB Streams**: A time ordered log of item modifications

  - Encrypted and stored for 24 hours
  - Can be accessed with a separate endpoint
  - Used for archiving transactions, triggering events based on transactions or replicating data

- **Exceeding Provisioned Throughput**

  - If there are too many r/w requests, you may receive a `ProvisionedThroughputExceededException`
  - When using AWS SDK it will auto retry on fai with **exponential backoff**, otherwise a client-side **exponential backoff** flow is required

## KMS (Key Management Service)

- **Envelope encryption**: Using a master key to generate keys that encrypt other data
  - Can help reduce sending chunks of data to KMS by sending the data key over the network instead
- **CMK (Customer Master Key)**: Used to generate, encrypt and decrypt data keys 
- Can enable automatic key rotation which will rotate keys every 365 days

## SQS (Simple Queue Service)

- Pull based message queue
- Messages can be kept in queue from 1 min to 14 days - 4 days by default
- Allows decoupling of application components by using SQS as a message queue inbetween them 
  - Can improve performance if one components is more performant than another
- **Queue Types**:
  - **Standard**: Unlimited transactions, guarantees messages delivered once, potential duplicates and tries to maintain message ordering
  - **FIFO**: Guaranteed ordering, guarantees messages delivered once, no duplication, limited to 300 transactions per second
    - Used for critical order queueing
- **Visibility Timeout**: Amount of time a message becomes invinsible after a consumer picks up a message
  - Default timeout of 30s; max option of 12hrs
  - After timeout, message becomes visible again for consumers to process
- **Polling types:**
  - **Short polling**: Returns an immediate response even if the queue is empty
    - Will waste resources/money if the queue is left empty; not preferrable
  - **Long polling**: Periodically polls the queue and only responds when a message arrives or poll timeout is reached
    - Max poll timeout at 20s
- **Delay queues**: Postpone delivery of new messages (messages remain invisible)
  - Default 0s; max 900s
  - Useful for large distributed apps where we need to wait for other services
- For large SQS messages (>256KB and <2GB) use **S3** to store the messages and **SQS Extended Client Library for Java** + **AWS Java SDK** to manage them 
  - Cannot use AWS CLI, management console, SQS API etc. to manage these

## SNS (Simple Notification Service)

- A pub-sub service used for push notifications, SMS, email, SQS, HTTP endpoints or to trigger Lambdas
- Topics are groups in SNS that subscribers will receive messages from
- Messages in SNS are stored across multiple AZs

## SES (Simple Email Service)

- Service used to send or receive emails (into an S3 bucket)
- Can be used to trigger a Lambda or SNS notification
- Not subscription based; only requires an email address

## Kinesis

- Used to collect, process and analyse streaming data in real-time
- **Services**:
  - **Kinesis Data Streams**: Stream data or video to build applications that process that data in real-time
    - **Shard**: A sequence of data records where kinesis data streams data is stored
      - 5 reads per second (2MB max per second)
      - 1000 writes per second (1MB max per second)
      - Data output to consumers is load balanced based on number of consumers
      - Ideally the number if consumer instances should not be more than the number of shards
      - One consumer can process multiple shards, but one shard only needs one consumer
    - 1 day retention by default (up to 7 days)
    - Requires data producer and consumer apps to process data 
  - **Kinesis Data Firehose**: Capture, transform and load data streams into AWS stores for near-real-time analytics
    - Uses no shards/no data retention
    - Does not require consumer
  - **Kinesis Data Analytics**: Analyse, query and transform streamed data in real-time using SQL
    - Can run SQL queries from data streams/firehose output 
    - Output can be stored in data stores

## Elastic Beanstalk

- Allows deployment and scaling of web apps
- Automatically configures load balancers, EC2 instances with HTTP servers and auto-scaling, data stores, databases and monitoring
- Automatically manages infrastructure, infrastructure provisioning and updates
- Elastic Beanstalk itself is free (not including resources used)
- Supports containers using Docker or with custom AMI's using Packer
- **Rollout methods**:
  - **All at once**: Deploys to all instances simultaneously
    - Will experience outage for all instances while redeploying
  - **Rolling**: Deploys in batches
    - Environment capacity will be reduced while certain batches are redeployed
  - **Rolling with additional batch**: Launches an additional batch of instancs, then do a rolling deploy
    - Can help maintain environment capacity
  - **Immutable**: Deploys to a new group of instances and then deletes old instances once complete
  - **Traffic splitting**: Deploys to a new group of instances but only directs a percentage of traffic to the new instances
- Configuration files can be used for defining packages to install, creating linux users, running commands, configuring load balancer and services, etc. in JSON or YAML
  - Must have `.config` extension and be placed inside a `.ebextensions` folder in the root directory
- RDS instances can be configured inside or outside the EB environment
  - Terminating an environment within the EB environment will delete the DB too (good for test purposes)
  - Configuring outside of an EB environment requires configuration of a security group and DB connection strings

## CloudFormation

- Infrastructure as code service - create templates to manage, configure and provision AWS infrastructure
- Free to use - only charged for resources used
- **Parameters**: Input values that are used in the configuration
- **Conditions**: Do actions based on parameters
- **Mappings**: Map diferent key-values (e.g. AMI can be mapped to different AMI IDs depending on region)
- **Transform**: Import and use external snippets to add to the template
- **Resources**: The AWS resources and their configurations that will be provisioned and deployed
  - This is the only mandatory section of the template
- **Outputs**: References of resources from output of stack - can be used as inputs to another CloudFormation stack
- Can have nested CloudFormation stacks by specifying a resource with type `AWS::CloudFormation::Stack` and by specifying a template URL which is the stack template
- **Change Sets** can be used to update existing stacks

## Serverless Application Model (SAM)

- Used to configure, package and deploy serverless applications with Lambda, DynamoDB and S3 in a CloudFormation stack

## CloudWatch

- Monitoring service to monitor health and performance of AWS services
- Metrics stored indefinitely by default
- **CloudWatch Agent**: Define your own metrics
  - Can also be used to collect OS level metrics
  - Can be used to monitor app or system log files in near real-time
- **CloudWatch Alarms**: Trigger alarms based on utilisation, latency, or charges on AWS bills

## CloudTrail

- Records user activity in an AWS account (essentially an audit trail of an AWS account)
- Can view the last 90 days of activity (by default)

## CodeDeploy

- Supports EC2, on-premise servers, ECS, Lambda and Fargate