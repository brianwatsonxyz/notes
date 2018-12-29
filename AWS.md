# Amazon Web Services

## General

## IAM

Access to AWS resources are determined by the union of an entity's permissions.

### Policies

A policy document can be user-created, or provided by Amazon. They consist of an Effect (Allow/Deny), Action (service:APIs), and Resource (ARNs). Can be attached to Users, Roles, and Groups.

### Roles

Roles are associated with a number of policies. The policies can be from the set of user created and managed policies, or provided inline.

### Users

Allow access to both resources in AWS and access to the console. A user can have both a password and access key, with the access key providing the programmatic access.

### Groups

Groups are associated with a number of policies. They provide an easier way to manage access because they allow job-based management, and if a change is needed it can be made at the group level instead of each individual user.

- web identity federation - cognito, sign up/sign in, syncs data, allows guests, auth with provider to retrieve auth token that is exchanged for temp aws credentials
- user pools - jwt token from provider
- identity pool - uses token to gain temp access to resources

## S3

all buckets default to private. control access via bucket policy and acl. data encrypted in transit (tls/ssl), at rest (sse-s3, sse-kms - managed keys with audit/rotation trail, sse-c - customer provides key), by client (pre put). object-based storage (key/value). 0-5TB bytes. buckets, globally unique. https://s3-REGION.amazonaws.com/BUCKETNAME or for website https://BUCKETNAME.s3-website-REGION.amazonaws.com

- consistency - put (read after write) overwrite-put and delete (eventual)
- storage classes - standard (immediately available) 99.99 availability, 99.999999999 durability, stored across multiple devices, multiple facilities, designed to sustain loss of 2 facilities, s3-ia (infrequently accessed) lower fee, cost to retrieve, s3-rrs (reduced redudancy), aka one zone-ia, lower cost, lower resiliancy, glacier (archived, 3-5h wait time)
- versioning - pay for all versions stored. cannot be disabled. mfa delete.
- lifecycle - can be applied to current/previous versions. transition to ia storage after 30d (128k min), archive to glacier (30d after ia), permanently delete
- replication - cross region, versioning required on source bucket
- retention
- transfer acceleration - uses cloudfront edges to accelerate s3 uploads. upload to distinct (edge) url instead of directly to s3, edge then transfers to s3.
- components of an s3 object/bucket - key, value, version id, metadata, subresources (bucket policies, acls)
- can log all accesses
- objects can be encrypted at upload via x-amz-server-side-encryption parameter; either AES256 (ss3-s3 - s3 managed keys) or ams:kms (sse-kms - kms managed keys); when header includes parameter in PUT request, s3 encrypts the object at time of upload; can set bucket policy to deny any s3 PUT without the x-amz-server-side-encryption parameter in the header
- optimization - GET-heavy workloads use cloudfront to geo-cache objects at edge locations; mixed workloads should introduce randomness in the key names so that objects are stored in different partitions

### S3 FAQ

- supports single puts up to 5gb
- access controlled via iam, bucket policy, acl, query string (pre-auth, limited time)
- 11 9's durability = storing 10mil objects expect 1 loss every 10k years

## Storage Gateway

connects an on-premise software appliance to aws for seamless/secure integration between the customer and aws storage infrastructure. downloaded vm. four types:

- file gateway (nfs) - store flat files as objects in s3, access through nfs, ownership/permissions/timestamps stored as metadata, can be managed as s3 objects after transfer (versioning, security, replication)
- volume gateway (iscsi) - block-based storage, virtual hd, can be async backed up as ebs snapshots. incremental, snapshots only capture changed blocks. stored compressed.
- - stored volumes - all data stored locally, replicated to aws, 1g-16tb in size.
- - cached volumes - frequently accessed data stored locally, replicated to aws. 1g-32tb in size. data stored in s3.
- tape gateway (vtl) - backup/archive, create virtual tape backups and store in s3.

## Snowball

tb/pb/xb data storage boxes. can import to and export from s3. legacy from import/export (portable hd, ship-import).

## Cloudfront

cdn. edge locations - location where content will be cached, seperate from a region/az, not-read-only (can put an object). origin - source of all files the cdn will distribute (s3 bucket, ec2, elb, route53). distribution - name given to cdn which consists of a collection of edge locations, can be web or rtmp (streaming), can have multiple origins. a request first hits edge location, edge checks cache, if no match, fetches from origin, serves, caches up to ttl. requests automatically routed to nearest edge location. content can be static, streaming, interactive. optimized to work with other services. seamless with non-aws origins too. can geo-restrict access with whitelist or blacklist. can create an invalidation to remove a cached object. can force content be retrieved through cloudfront.

## EC2

- pricing models
- - on demand - fixed rate by the second; low cost up-front, no commitment; apps with short-term, spikey usage, or cannot be interrupted; test beds
- - reserved - discount, 1y, 3y terms; apps with predictable usage or require reserved capacity; make upfront payments (standard RI 75% off, convertable RI 54% off, but can change/upgrade RI, scheduled RI launch within time window, useful for weekly sales, etc), can transfer RI between AZs
- - spot - set bid price, useful for flexible start/end times; apps/functions only feasible at low cost, batch, genomics; if you terminate, you pay for the hour, if aws terminates, you get that hour free
- - dedicated hosts - physical ec2 hosts, utilize existing software licenses; conform to regulations; can be purchased on-demand and as RI

- instance types
- - D2 - dense storage, fileservers/data warehousing
- - R4 - memory optimized, memory intensive apps/db's
- - M4 - general purpose, app servers
- - C4 - compute optimized, app servers
- - G2 - graphics intensive, video encoding/3d
- - I2 - iops/high speed storage, nosql db's/data warehousing
- - F1 - fpga, hardware acceleration
- - T2 - low cost, general purpose, web/app servers
- - P2 - graphics/general purpose, ml/crypto
- - X1 - memory optimized, db's/spark

- http://169.254.169.254/latest/meta-data/...

## EBS

cannot mount 1 ebs volume to multiple instances. can modify type/size for all but magnetic(standard). ebs/ec2 instance must be in same az. can create copy ec2 instance by snapshotting ebs volume (or directly from instance image), create new image in different az and attach to new ec2 instance in that az. snapshots exist on s3. snapshots are point-in-time copies of volumes. snapshots are incremental. can create ami from volumes and snapshots. snapshots of encrypted volumes are encrypted automatically, volumes restored from encrypted snapshots are encrypted automatically. can share unencrypted snapshots.

- - general purpose ssd (gp2) - 3 iops per gb up to 10k iops, burst up to 3k iops
- - provisioned ssd (io1) - designed for high i/o, rdms/nosql, use if > 10k iops needed, can provision up to 20k iops per volumn
- - throughput optimized hdd (st1) - big data, logging, cannot be boot volume
- - cold hdd (sc1) - low cost, infrequently accessed, cannot be boot volume
- - magnetic hdd (standard) - lowest \$/gb, infrequently accessed and lowest cost, can be boot volume

ami's are either ebs backed or use instance store. ebs is root device for instance via ebs snapshot, instance store volume created from s3 template. instance store = ephemeral storage. instance store volumes cannot be stopped, can be restarted, data saved if rebotted, ebs backed instances can be stopped without loosing data. both volume types will be deleted on termination, unless ebs and marked to keep.

to encrypt a root volume, use os-level tools (bitlocker), or snapshot, copy snapshot with encryption, then make an ami and launch new instance. attached volumes can be encrypted via console, cli, or api

## Security Groups

a virtual firewall. 1 instance can have multiple sg's.

- all inbound traffic blocked by default
- all outbound traffic is allowed
- changes to sg immediate
- any number of ec2 instances per sg
- ec2 instances can have multiple sg's
- sg's are stateful, inbound traffic allowed back out
- cannot block specific ip's
- only specify allow rules

## Elastic Load Balancers

- application load balancers - layer 7, http/https, terminates tls, operates at the request level
- network load balancer - layer 4, best for tcp traffic, extreme perf; static ip; operates at connection level
- classic load balancer - layer 4 ob tcp, legacy, load balance http/https using layer 7 features (x-forwarded)
- errors respond 504 error, gateway timed out, application issues (web, db server)
- instances reported as InService, OutofService
- use dns name

## Route 53

- ipv4 (32 bit), ipv6 (128 bit)
- ns record - nameserver; tld servers use to direct traffic for dns resolutions
- a record - address; resolves domain name to ip
- cname record - canonical name; resolves one domain name to another
- alias record - alias; resolves domain name to target resource
- elb's do not have ipv4 addresses, just dns names
- routing policies (simple, weighted, latency, failover, geo)
- given a choice, choose alias record as there is no charge, cname resolutions infer costs
- routing policies
- - simple - default, common with single resource; in route 53, A record as alias to target resource
- - weighted - splits traffic based on assigned weights, common in a/b testing
- - latency - routes on lowest latency; select from set of regions
- - failover - routes to secondary if primary's health fails, active/passive setup, failover to DR
- - geolocation - routes based on location (by country, state, etc); can set a default

## RDS

- two types of backups: automated backups allow recovery of db at any point during retention period (1-35d), consist of daily snapshot plus transaction logs throughout the day, can recover to a second, free backup storage up to size of db; snapshots are done manually, stored even after rds instance terminated (unlike automated backups)
- restoring either backup type will be a new rds instance and endpoint
- encryption at rest is suppported; once encrypted all backups, read replicas, snapshots are also encrypted
- multi-AZ for DR, automatic fail-over
- read replicas for performance, each replica has own endpoint, up to 5 replicas per db, replicas can be multi-AZ, must have automatic backups on, can be promoted to their own db (breaks replication), replicas can be in another region

## DynamoDB

- set read and write throughput
- read - all reads rounded up to increments of 4kb; eventually consistent reads (default) consist of 2 reads/sec, strongly consistent reads consist of 1 read/sec
- write - all writes are 1kb; all writes 1 write/sec
- read throughput = (size of read rounded to nearest 4kb chunk / 4kb) \* no of items and divide by 2 if eventually consistent, round final value up to next whole number
- if provisioned throughput exceeded status 400 ProvisionedThroughputExceededException is returned
- local secondary index - only created when creating table (cannot add/remove/modify it later); has same partition key as original table, but different sort key; different view of data organized by alternative sort key; queries on this sort key much faster using the index than the main table
- global secondary index - can create during or after table is created; different partition key and sort key; completely different view of data; speeds up queries relating to the alternative partition and sort keys
- query
- - finds items based on primary key and distinct value to search for
- - use optional sort key to filter results
- - can use a ProjectionExpression to only return specific attributes
- - results are sorted by sort key (numerically, ascii char codes)
- - can reverse order by setting ScanIndexForward to false (only applies to queries)
- - by default queries are eventually consistennt (must explicitly set to strongly consistent)
- scan
- - examines every item in the table
- - can use a ProjectionExpression to only return specific attributes
- query vs scan
- - queries more efficient than a scan
- - scan dumps entire table, then filters out unmatched data
- - scan takes longer
- - scan can use up provisioned iops in a single operation
- improving performance
- - reduce the impact of a query or scan by setting smaller page size
- - prefer larger number of smaller operations to avoid throttling
- - avoid scans (use Query, Get, BatchGetItem apis)
- - by default, scan processes data sequentially in 1mb chunks, one partition at a time; configure parallel scans by logically dividing table/index into segments and scanning in parallel; avoid parallel scans if table already under heavy load
- dynamodb accelerator (dax)
- - fully managed, clustered, in-memory cache for dynamodb
- - 10x read performance improvment
- - millions of requests per second
- - ideal for read-heavy, bursty workloads
- - write-through caching service
- - point dynamodb api calls at dax cluster, returns cache-hits, otherwise does eventual consistent GetItem read against dynamodb, caches result
- - only supports eventually consistent reads
- - not as suitable for apps that are write-heavy, do few reads, do not require microsecond response times

## Kinesis

- streams - 24h - 1week retention, producers put to shards, consumers process. shards handle 5 transactions per second for reads, up to maximum rate of 2mb/second; up to 1k transactions per second for writes, up to maximum total data write rate of 1mb/second - per shard. total data capacity is sum of all shards.
- firehose - more automated, producers put to firehose, data pushed to S3, can be processed by lambda.
- analytics - run sql queries on data in streams or firehose, store output in S3, Redshift, Elasticsearch.

## SQS

service that provides message queues, pull based, 256k max size, guaranteed to be processed at least once. long-polling does not return unless a message arrives (or the long poll times out). messages can be stored in queue from 1m - 14d, 4d is default. standard - no guaranteed order, messages can be delivered more than once. fifo - guaranteed order. max long-poll time is 20s

vto - visability time out is amount of time a message is invisible after it is picked up by a reader. the reader has vto-time to process and delete the message, else it will become visible to (and processed by) other readers again (delivered twice+). vto maximum is 12h. ChangeMessageVisibility extends processing time. default vto = 30s.

## SWF

retention period up to 1y. task-oriented api. ensures task is assigned only once and never duplicated. keeps track of all tasks and events. actors - starters (can initiate a workflow), deciders (control the flow of activity tasks, handles completion, failures), workers (carry out the activity tasks). domain - collection of related workflows.

## SNS

subscribers (http/s, email, sqs, application, lambda). pub-sub. push based. [LIMITS, STATS] can group recipients by topic. one topic can deliver to multiple endpoints.

- topic names are limited to 256 characters, alphanumeric characters plus hyphens (-) and underscores (\_) are allowed
- by default, SNS offers 10 million subscriptions per topic, and 100,000 topics per account

message contents:

MessageId: A Universally Unique Identifier, unique for each notification published.
Timestamp: The time (in GMT) at which the notification was published.
TopicArn: The topic to which this message was published
Type: The type of the delivery message, set to “Notification” for notification deliveries.
UnsubscribeURL: A link to unsubscribe the end-point from this topic, and prevent receiving any further notifications.
Message: The payload (body) of the message, as received from the publisher.
Subject: The Subject field – if one was included as an optional parameter to the publish API call along with the message.
Signature: Base64-encoded “SHA1withRSA” signature of the Message, MessageId, Subject (if present), Type, Timestamp, and Topic values.
SignatureVersion: Version of the Amazon SNS signature used.

## SES

- email service, pay as you go
- can receive emails to s3 bucket (can trigger lambda)
- good for automated emails, confirmations, marketing

## Elastic Transcoder

converts media files between different formats. output format device presets. billed based on minutes and resolution transcoded.

## Elastc Beanstalk

- app is collection of components (environments, versions, configurations)
- high level configuration
- entire app as one eb app or multiple
- can have multiple environments (prod, dev, front-end, etc)
- either single instance or scalable
- either web server or worker
- each app can have many versions, deployed to multiple environments
- platforms: node, php, python, ruby, tomcat, .net, java, go, packer, docker
- if eb configures db, eb will remove on termination, otherwise create db first and connect to it
- updates can be 1 instance, a % of instances, or an immutable update
- deployments:
- - all at once - all instances simultaneously updated, all instances out of service during update, on failure must re-deploy original version
- - rolling - deployed in batches, batch of instances taken out of service while updating, on failure must perform additional rolling update to roll back changes
- - rolling with additional batch - launches additional batch of instances, deploys in batches, maintains full capacity, if update fails must perform additional rolling update to roll back changes
- - immutable - deploys new version to group of instances in new autoscaling group, when healthy, are moved to existing autoscaling group and old instances are terminated, maintains full capacity, on failure just terminate new autoscaling group
- eb is free, pay for resources you use
- apache, nginx, tomcat, passenger, iis
- no persistent storage on instances, unless configured outside eb
- can configure eb by providing .config files in .ebextensions folder in root of app pkg

## Cloud Formation

- service is free, pay for resources used
- auto-rollback on error on by default, charged for errors
- stacks can wait on WaitCondition
- route 53 supported
- various functions: GetAtt, Sub
- provision across regions
- by default, only launch 200 stacks per region
- up to 60 parameters and 60 outputs in a template

## Elasticache

- improves latency
- memcached - pure object cache; no persistence; managed as pool of nodes that can grow/shrink; use when 1) object caching is main goal 2) simple caching model 3) large cache nodes, multi-core 4) scale horizontally
- redis - key-value store (data structures, sets, lists); supports master/slave replication, multi-AZ; use when 1) caching advanced data types (lists, hashes, sets) 2) sorting/ranking datasets needed 3) persistence important 4) need multi-AZ with failover 5) pub-sub capabilities needed
- two caching strategies
- - lazy-loading - app checks cache, on hit returns data, else app fetches data, writes to cache before returning; only requested data is cached, but cache miss requires penalty of cache miss + write to cache; node failures not fatal (just need to cache from empty); data can become stale; can set ttl
- - write-through - app writes all updates to data store and cache; data in cache never stale, but penalty for cache-write each time data is updated; wasted resources if cache-writes never read

## VPC

- allowed 5 vpcs per region (default)
- by default, subnets within vpc can communicate with each other
- vpc endpoints - can add routes to aws services that operate over private ip

## Lambda

- node, java, c#, python, go
- first 1M requests are free
- scales out automatically
- functions can trigger functions (fan-out)
- serverless services - lambda, s3, dynamodb, apigw
- triggers - apigw, aws iot, alexa skills kit, alexa smart home, cloudfront, cloudwatch events, cloudwatch logs, codecommit, cognito sync trigger, dynamodb, kinesis, s3, sns, sqs
- can have multiple versions
- latest version = \$LATEST
- qualified version includes ":version"
- versions are immutable
- can split traffic between aliases to different versions
- cannot split traffic with \$LATEST (instead create alias and use it)

## API Gateway

- responses can be cached
- low cost, scales automatically
- can throttle to prevent attacks; defaults limited to 10k requests per second, 5k concurrent requests across all apis within an aws account; exceeding limits results in 429 Too Many Request error
- log to cloudwatch
- cors configurable
- can import apis (supports swagger 2.0 definition files); can create new api via post, update existing api using a put, merge api
- can configure apigw as soap web service passthrough

## Step Functions

- great way to visualize serverless application
- automatically triggers each step
- logs the state of each step

## X-Ray

- sdk includes intercaptors (add to code to trace incoming http requests), client handlers (instrument calls from app code to other aws services), http client (instrument calls from app code to other web services)
- integrates with elb, lambda, apigw, ec2, beanstalk
- languages - java, go, node, python, ruby, .net
- provides visualization of requests, tracks errors, provides logs

## KMS

- keys are regional
- customer master key (CMK) - alias, creation date, description, key state, key material (customer provided or AWS provided)
- can never be exported
- setup by entering alias, material option, define key admin and key usage permissions
- aws kms encrypt
- aws kms decrypt
- aws kms re-encrypt
- aws kms enable-key-rotation
- envelope encryption - encrypts the key that encrypts data
- CMK - decrypts the data key (envelope key), envelope key used to crypt data

## CodeCommit

- git

## CodeBuild

## CodeDeploy

- deployment options:
- - in-place - app is stopped, latested version installed, instance out of service during, if behind lb, can configure lb to stop sending it requests; also known as a rolling update; only for ec2 and on-premise; roll backs require re-deploy of prevvious app
- blue/green - new instances provisioned and app installed; blue are active, green are new; green instances are registered with elb, traffic rerouted, blue instances are terminated; advantages are new instances created ahead of time, traffic then switched; reverting to previous deployment simply requires re-routing traffic back (if they're not yet terminated)
- appspec defines all deployment parameters; for ec2/on-premise, must be in root and be .yml; lambda supports json or yaml

- terminology
- - deployment group - set of ec2 instances or lambda funcs to which a new revision is to be deployed
- - deployment - proccess and components to apply a new revision
- - deployment configuration - set of deployment rules and success/failure conditions
- - appspec file - defines the deploy actions you want codedeploy to perform
- - revision - everything needed to deploy new revision (appspec, files, assets, configs)
- - application - unique identifier for the application to deploy

- in-place deployment hooks:
- - BeforeBlockTraffic
- - BlockTraffic
- - AfterBlockTraffic
- - ApplicationStop
- - DownloadBundle
- - BeforeInstall
- - Install
- - AfterInstall
- - ApplicationStart
- - ValidateService
- - BeforeAllowTraffic
- - AllowTraffic
- - AfterAllowTraffic

## CodePipeline

- comprised of different tasks
- can configure for every push to s3, codecommit triggers pipeline

## ECS

## AWS Shared Responsibility Model

## AWS Well-Architected Framework

- 5 pillars

- - operational excellence - run and monitor systems to create business value; continually improve supporting processes and procedures
- - security - protect information, systems, and assets while delivering business value through risk assessments and mitigation strategies
- - reliability - recover from infrastructure or service disruptions; dynamically acquire resources to meet demand; mitigate disruptions due to configuration or network issues
- - performance efficiency - use computing resources efficiently to meet requirements; maintain efficiency as demands change and technology evolves
- - cost optimization - avoid or eliminate uneeded cost or suboptimal resource usage

- design principles

- - stop guessing capacity; scale up and down as needed
- - test systems at production scale; spin up and tear down full-scale test environments
- - automate architecture; allows low-cost creation w/no manual effort; audit changes; easy to revert
- - allow for evolutionary architectures to experiment and reduce the risk from design changes
- - drive architectures using data; collect and analyze architectural data to inform choices
- - use game days to simulate events in production
    pg5

## AWS Security Best Practices

- AWS provides secure infrastructure and services, while you, the customer, are responsible for secure operating systems, platforms, and data.
- pg10

## Practice

- with lambda, feed a kinesis stream twitter data, utilize kinesis data firehose to store the data in s3; process the data in s3 with lambda and store in dynamodb; use aws cli to query the data
- with lambda, feed an sqs queue twitter data, utilize elastic beanstalk to poll the queue and store the data in rds; create a lambda function that returns the last 10 data items; use aws cli to invoke lambda
- create a codebuild project with a buildspec that uses all phases and features
- create an ec2 deployment using codedeploy; use all available hooks
- create a lambda blue/green deployment using codedeploy; use all available hooks
- create a codepipeline project that includes using both an action and lambda in the pipeline
- create a dockerized app and deploy it with elastic beanstalk; load balance the app and deploy using each deployment policy including blue/green
- use elasticache to do both write-through and lazy-load in front of rds
- create an ecs cluster of tasks behind a load balancer using port-based and path-based balancing
