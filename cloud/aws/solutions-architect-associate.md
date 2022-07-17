# Certification Overview

- AWS Overview > FAQ > Whitepaper 순으로 내용을 파악한 뒤 예제 문제들(dump)을 풀어보자.
- 65문제를 130분동안 시험을 보게 된다. 그러나 영어가 제 2 외국어인 사람이라면 ESL+30 으로 30분 추가 요청을 할 수 있다.
- 시험을 한국어로 신청해도 번역이 이상한 경우 영어 원문으로도 볼 수 있는 기능이 제공된다.
- 1000점 만점에 720 ~ 730점이 합격선이다. 애매한 문제를 만나면 표시(기능이 있다)해두고 나중에 다시 보면 된다.



# References

- [AWS SAA Challenge](https://pages.awscloud.com/GLOBAL_TRAINCERT_takethechallenge_resourcehub.html#GetStarted) 
- [AWS SAA Workshop](https://pages.awscloud.com/traincert_alwayslearning_virtualevents_kr.html?sc_channel=em&sc_campaign=APAC_TRAINCERT_WEBINAR_DG-Event_KR-builders-online_202107_APAC_FY21_Q2_tclever_FLC_EM&sc_medium=em_399146&sc_content=PA_t1_traincert&sc_geo=mult&sc_country=mult&sc_outcome=pa&trk=em_399146) 
- [AWS 공인 솔루션스 아키텍트 – 어소시에이트 자격증](https://aws.amazon.com/ko/certification/certified-solutions-architect-associate/) 
- [AWS Learning Paths](https://aws.amazon.com/ko/training/learning-paths/) 
- [AWS Builders](https://aws.amazon.com/ko/events/seminars/aws-builders/) 
- [AWS Open Guides](https://github.com/open-guides/og-aws) 



- [ ] Guides
    - [x] [Certification Guide](https://aws.amazon.com/ko/certification/certified-solutions-architect-associate/) 
    - [x] [Exam Guide](https://d1.awsstatic.com/ko_KR/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf) 
        - [x] [AWS SAA Exam Guide - ko.pdf](AWS SAA Exam Guide - ko.pdf) 
        - [x] [AWS SAA Exam Guide - en.pdf](AWS SAA Exam Guide - en.pdf) 
        - [ ] [AWS SAA Sample Questions - SAA-C02.pdf](AWS SAA Sample Questions - SAA-C02.pdf) 
        - [x] [AWS SAA Sample Questions - SAA-C01.pdf](AWS SAA Sample Questions - SAA-C01.pdf) 
        - [ ] [비공식 AWS 공인 솔루션스 아키텍트 - 어소시에이트 SAA-C02 (2020년 3월 출시) 수험 가이드](https://github.com/serithemage/AWSCertifiedSolutionsArchitectUnofficialStudyGuide) 
        - [x] [시험 시간 +30분 추가 요청](http://edu.supertrack.co.kr/community/news.php?ptype=view&idx=5177&page=1&code=news) 
- [ ] Courses
    - [x] [AWS Training](https://www.aws.training/Account/Transcript/Current) > [AWS Skill Builder](https://explore.skillbuilder.aws/) 
        - [x] 2021-08-13 [AWS Technical Essentials on AWS Training](https://www.aws.training/Details/eLearning?id=71079) > [AWS Technical Essentials on AWS Skill Builder](https://explore.skillbuilder.aws/learn/course/internal/view/elearning/1851/aws-technical-essentials) 
        - [x] The AWS Certification Quiz Show Series
        - [x] 2021-02-18 [Exam Readiness: AWS Certified Solutions Architect – Associate (Digital)](https://explore.skillbuilder.aws/learn/course/external/view/elearning/125/exam-readiness-aws-certified-solutions-architect-associate-digital) 
    - [ ] [Exam Readiness Webinar](https://aws.amazon.com/ko/training/events/?nc1=h_ls&get-certified-vilt-courses-cards.sort-by=item.additionalFields.startDateSort&get-certified-vilt-courses-cards.sort-order=asc&awsf.get-certified-vilt-courses-type=*all&awsf.get-certified-vilt-courses-series=series%23aws-certification-exam-readiness&awsf.get-certified-vilt-audience=*all&awsf.get-certified-vilt-locations=*all&awsf.get-certified-vilt-countries=*all&awsf.get-certified-vilt-languages=*all&awsf.get-certified-vilt-courses-level=*all&awsf.get-certified-vilt-courses-tech-category=*all&get-certified-vilt-courses-cards.q=solutions%2Barchitect%2B-%2Bassociate&get-certified-vilt-courses-cards.q_operator=AND&saa=sec&sec=prep) 
    - [ ] [AWS Power Hour: Architecting](https://pages.awscloud.com/traincert-twitch-power-hour-architecting.html?saa=sec&sec=prep) 
    - [x] [AWS Certified Solutions Architect - Associate 2020](https://www.youtube.com/watch?v=Ia-UEYYR44s) 
    - [ ] [AWS Certified Solutions Architect Associate 2021 Free Course](https://www.youtube.com/playlist?list=PLiH9_MU-6RjI9gdFqmvUfKRfw_zRxIb6o) 
    - [ ] [Builders](./builders) 
- [ ] Documents
    - [ ] [AWS FAQ](https://aws.amazon.com/ko/faqs/) 
        - [x] [EC2](https://aws.amazon.com/ko/ec2/faqs/?saa=sec&sec=prep) 
        - [x] [S3](https://aws.amazon.com/ko/s3/faqs/?saa=sec&sec=prep) 
        - [ ] [VPC](https://aws.amazon.com/ko/vpc/faqs/?saa=sec&sec=prep) 
        - [ ] [Route 53](https://aws.amazon.com/ko/route53/faqs/?saa=sec&sec=prep) 
        - [ ] [RDS](https://aws.amazon.com/ko/rds/faqs/?saa=sec&sec=prep) 
        - [ ] [SQS](https://aws.amazon.com/ko/sqs/faqs/?saa=sec&sec=prep) 
    - [x] [AWS Whitepapers](https://aws.amazon.com/ko/whitepapers/) 
    - [x] [AWS Well-Architected Framework](https://aws.amazon.com/ko/architecture/well-architected/?saa=sec&sec=prep) 
        - [x] [AWS Well-Architected Framework.pdf](AWS Well-Architected Framework.pdf) 



# Q&A

- What are some reasons to enable cross-region replication on an Amazon S3 bucket? (choose 2 answers)
    - You want a backup of your data in case of accidental deletion.
    - **You have a set of users or customers who can access the second bucket with lower latency.**
    - **For compliance reasons, you need to store data in a location at least 300 miles away from the first region.**
    - Your data needs at least five 9's of durability.



- Which Amazon EC2 feature ensures that your instances will not share a physical host with instances from any other AWS customer?
    - Amazon Virtual Private Cloud (VPC)
    - Cluster placement groups
    - **Dedicated Instances**
    - Reserved Instances 



- Your web application runs on multiple Amazon EC2 instances behind an Application Load Balancer. The load balancer is configured to perform health checks on the EC2 instances. If an instance fails to pass health checks, which statement will be true?
    - The instance is replaced automatically by the load balancer.
    - The instance is terminated automatically by the load balancer.
    - **The load balancer stops sending traffic to the instance that failed its health check.**
    - The instance is quarantined by the load balancer for root cause analysis. 



- Which of the following actions can be authorized by IAM? (Choose 2 answers)
    - Installing ASP.NET on a Windows Server
    - **Launching an Amazon Linux EC2 instance**
    - Querying an Oracle database
    - **Adding a message to an Amazon Simple Queue Service (SQS) queue** 



- What aspect of an Amazon VPC is stateful?
    - Network ACLs
    - **Security Groups**
    - VPC Peering
    - VPC Subnet 



- What are characteristics of the Amazon EC2 Auto Scaling service?
    - Sends traffic to healthy instances
    - Responds to changing conditions by Stopping and Starting Amazon EC2 instances
    - **Responds to changing conditions by Terminating and Launching Amazon EC2 instances**
    - **Enforces a minimum number of running Amazon EC2 instances** 



- When using Amazon RDS Multi-AZ, how can you offload read requests from the primary? (Choose 2 Answers)
    - Configure the application to connect to the secondary node for reads and the primary node for writes.
    - Amazon RDS automatically sends writes to the primary and sends reads to the secondary.
    - **Add a read replica DB instance, and configure the client's application logic to use a read-replica.**
    - **Use ElastiCache to cache frequently used data. Update the application logic to read/write from the cache.** 



- Your company has its primary production site in North America and its DR site in Asia. You need to configure DNS so that if your primary site becomes unavailable, you can fail DNS over to the secondary site. Which DNS routing policy would best achieve this?
    - Weighted routing
    - Geolocation routing
    - Simple routing
    - **Failover routing** 



- Your corporate data center was recently flooded, which caused significant outages.Your CIO mandated a move to the cloud but they are still concerned about catastrophic failures in a data center. What can you do to alleviate their concerns?
    - **Distribute the architecture across multiple Availability Zones.**
    - Use an Amazon VPC with subnets.
    - Launch the compute in a placement group.
    - Purchased Reserved Instances for the processing servers. 



- Which feature of AWS is designed to permit calls to the platform from an Amazon EC2 instance without needing access keys placed on the instance?
    - **IAM instance profiles**
    - IAM groups
    - IAM roles
    - Amazon EC2 key pairs



- Your company has 50,000 weather stations that send updates every 2 seconds. What service will enable you to ingest this stream of data and store It to Amazon S3 for future processing?
    - Amazon Simple Queue Service (Amazon SOS)
    - **Amazon Kinesis Data Firehose**
    - Amazon Elastic Compute Cloud (Amazon EC2)
    - Amazon Data Pipeline 



- You have an application that for legal reasons must be hosted in the United States when US citizens access it. The application must be hosted in the European Union when citizens of the EU access it. For all other citizens of the world, the application must be hosted in Sydney. Which routing policy should you choose in order to achieve this?
    - Latency-based routing
    - Data Governance routing
    - **Geolocation routing**
    - IP Lookup routing



- How can you grant a different AWS account permission to send messages to your Amazon SOS queue?
    - Have the other account's applications use your account's credentials to access the Amazon SOS queue.
    - Create an IAM User for the other account and add an IAM policy that grants access to the queue.
    - **Create an Amazon SOS policy that grants the other account access.**
    - Use Amazon VPC Peering between the two accounts. 



- You are building a photo management application that maintains metadata on millions of images in an Amazon DynamoDB table. When a photo is retrieved, you want to display the metadata next to the image. Which Amazon DynamoDB operation will you use to retrieve the metadata attributes from the table?
    - **Query operation**
    - Scan operation
    - Search operation
    - Find operation 



- An Amazon VPC security group can be associated with:
    - An Amazon EC2 instance
    - **An Elastic Network Interface**
    - An Amazon VPC subnet
    - An Elastic IP address 



- You have configured an AWS Lambda function to launch an Amazon EC2 Linux instance each Sunday night to run a 30-minute batch job. What would be the most reliable way to terminate the instance after it has completed the batch job?
    - **Have the instance run "sudo shutdown now -h".**
    - Create an AWS Batch job configured to run once per week to terminate the instance with a known Tag.
    - The instance sends a custom metric to Amazon Cloud Watch, which causes an Alarm to terminate the instance.
    - Program the Lambda function to wait 30 minutes, then have it terminate the instance.



- You have an Amazon EC2 instance that was launched from a Community AMI. You have received notice that this AMI is about to be deleted. How can you ensure that your instance will continue to run after the AMI is deleted?
    - Create a new AMI from the instance. Launch a new instance with it.
    - Create an Amazon EBS snapshot of your instance. Launch a new instance from the snapshot.
    - Use the Amazon Data Lifecycle Manager to replicate the AMI in your own AWS Account
    - **The AMI deletion will not affect your instance.**



- Your team is building an order processing system that will span multiple Availability Zones. During testing, the team wants to test how the application will react to a database failover. How can you enable this type of test?
    - **Reboot the primary instance from the Amazon RDS console.** 
    - Terminate the DB instance, then create a new one in a different AZ. Update the connection string to point to the new instance.
    - Create a Support Case asking for a failover of the instance, with at least 24 hours notice.
    - It is not possible to test a failover to a different AZ. 



- Which DNS records are commonly used to stop email spoofing and spam?
    - MX records
    - **SPF records**
    - A records
    - CNAMEs 



- Your company's IT management team is looking for an online tool to provide recommendations to save money, improve system availability and performance, and to help close security gaps. What can help the management team?
    - Amazon Cloud Watch
    - **AWS Trusted Advisor**
    - AWS Config
    - Configuration Recorder 



- Who can create objects in an Amazon S3 bucket with this bucket policy?

    ```json
    {
        "Version": "2012-10-17", 
        "Statement": [
            {
    			"Effect": "Deny", 
                "Principal": "*", 
                "Action": "s3:PutObject", 
                "Resource": "arn:aws:s3:::my-bucket/*"
            }
        ]
    }
    ```

    - Any IAM User
    - **Nobody**
    - Only IAM Users with "AdminAccess" permission
    - Only the IAM user who created the bucket



- Your account has a regional Reserved Instance for one Amazon Linux m5.2xlarge instance type. However, you have been running a m5.4xlarge instance instead. This was the only instance being used in the account. How will the usage of the m5.4xlarge be charged?
    - It will be charged at normal on-demand prices
    - It will be charged at double the normal on-demand price
    - **It will be charged at half the normal on-demand price**
    - It will use the instance reservation and will not be charged 



- You notice that all messages in an Amazon SQS queue are being redirected to the Dead Letter Queue even though they have been correctly processed by your application. What could be causing this?
    - The Retention Period of the queue is set too low
    - The 'maxReceiveCount' setting on the queue is too low
    - **The application is not deleting the message after processing it**
    - The Dead Letter Queue redrive policy has not been activated 



- When can an Amazon SNS message be deleted after being published to a topic?
    - Only if a subscriber(s) has/have not read the message yet
    - Only if the Amazon SNS recall message parameter has been set
    - Only if the subscribers are Amazon SQS queues
    - **Messages cannot be deleted** 



- A third-party security application is making API calls to your AWS account. However, it is failing to operate correctly and returns a generic "Access Denied" message. Where can you look to identify the cause of this error?
    - Amazon Cloud Watch Logs
    - IAM Policy Simulator
    - AWS API Gateway
    - **AWS CloudTrail**



- Your "www.example.com" domain is pointing to an Amazon EC2 instance. How could you keep this operating, but send requests for "www.example.com/news/" to an AWS Lambda function?
    - **Use an Application Load Balancer with a separate Target Group**
    - Add a "/news" CNAME record to Amazon Route 53 that resolves to AWS Lambda
    - Send all requests to an AWS Lambda function, which can route the request based on the URL
    - Use Amazon CloudFront with two configured Origins



- You are hosting a massively-multiplayer online game and want to allow customers to download client software whenever a new version is released. What would be the most cost-effective way to distribute this software?
    - **Use Amazon CloudFront Price Class 100**
    - Use Amazon S3 Auto-Compression option
    - Use Amazon S3 Reduced Redundancy transfer protocol
    - Use Amazon S3 Transfer Acceleration



- You have an Amazon VPC being used only for Dev/Test purposes. The VPC is connected to your data center via AWS Direct Connect. The VPC does not have an Internet Gateway attached. How can you provide applications running on Amazon EC2 instances with access to Amazon S3 without going via your data center?
    - Direct traffic via a secondary ENI in another subnet
    - Use a NAT Gateway to forward requests to Amazon S3
    - **Create a VPC Endpoint for Amazon S3**
    - It is not possible to access Amazon S3 without an Internet Gateway



- What is an efficient way to fan-out a single Amazon SNS message to multiple Amazon SOS queues?
    - Create one SQS queue that subscribes to multiple SNS topics.
    - **Create multiple SQS queues that subscribe to the SNS topic.**
    - Create an AWS Lambda function that subscribes to the SNS topic and sends copies to multiple SQS queues.
    - Use custom attributes on the SNS message to define which SQS queues should receive the message.



- How can you access files stored in Amazon S3 via NFS and SMB from both your data center and Amazon EC2 instances?
    - Install Amazon WorkDocs Sync Client on each machine
    - **Mount volumes from AWS Storage Gateway - File Gateway**
    - Use AWS DataSync in replication mode across Direct Connect
    - Use the native file share capabilities of each operating system 



- Your company provides media content via the Internet to customers through a paid subscription model. You leverage Amazon CloudFront to distribute content from an Amazon S3 bucket. What approach can you use to serve this private content securely to your paid subscribers?
    - **Provide signed CloudFront URLs to authenticated users.**
    - Add subscriber IP addresses to the CloudFront security group.
    - Use the geo restriction feature to restrict access to all of the paid subscription media at the country level.
    - Provide subscribers with an Origin Access Identity to grant them access to the CloudFront distribution.



- You have an application running on multiple Amazon EC2 instances in a single Availability Zone. The application calls a third-party API via the Internet. How can you provide the third-party with a single IP address to add to an access whitelist?
    - Assign an Elastic IP address to the instances
    - Assign a Public IP address to the instances
    - **Put the instances behind a NAT Gateway**
    - Put the instances behind a Network Load Balancer 



- Two teams in your company are using separate AWS accounts that were allocated using AWS Organizations. How can both teams launch Amazon EC2 instances in the same VPC?
    - Use VPC Peering to join two VPCs together.
    - **Use VPC Sharing to share a subnet between the accounts.**
    - Use a Transit Gateway to provide access to a shared VPC.
    - Use a Transit VPC to link two VPCs together. 



- Your company needs to store 200TB of product videos. The videos were created over the last several years, with the most recent being accessed the most often. The data must be accessed locally, but there is insufficient space in the data center to install sufficient local storage devices to store this data. What service will meet these requirements?
    - AWS Import/Export
    - Amazon EC2 instances with attached Amazon EBS volumes
    - AWS Snowball Edge
    - **AWS Storage Gateway - Cached volumes** 



- Which type of Elastic Load Balancer allows path-based routing for selecting the destination target group?
    - Classic Load Balancer
    - **Application Load Balancer**
    - Network Load Balancer
    - Target Load Balancer 



- What is the minimum number of Amazon EC2 instances required in an Auto Scaling group?
    - **Zero**
    - One per group
    - One per AZ
    - Two per VPC 



- Your web application needs 4 Amazon EC2 instances to support steady traffic nearly all of the time. On the last day of each month, the traffic triples. What is the most cost effective way to handle this traffic pattern?
    - Run 4 On-Demand Instances constantly, then add 8 more on On-Demand Instances on the last day of each month
    - **Run 4 Reserved Instances constantly, then add 8 On-Demand Instances on the last day of each month**
    - Run 4 On-Demand Instances constantly, then add 8 Reserved Instances on the last day of each month
    - Run 12 Reserved Instances all of the time



- How are you billed for an Elastic IP address?
    - Per second when associated with a running Amazon EC2 instance
    - **Per hour when not associated with an Amazon EC2 instance**
    - Per GiB for the amount of data that flows through them
    - Per network adapter based on the Amazon EC2 instance type



- Your company stores critical business documents in Amazon S3, and wants to minimize cost. Most documents are used actively for only about a month, then much less frequently. However, all data needs to be available immediately when requested. How can you meet these requirements?
    - Migrate to Amazon S3 Reduced Redundancy Storage (RRS) after 30 days
    - Migrate the data to Amazon Glacier after 3o days
    - **Migrate the data to Amazon S3 Standard - Infrequent Access after 30 days**
    - Turn on versioning, then migrate the older version to Amazon Glacier



- Which of the following are the minimum required elements to create an Auto Scaling launch configuration?
    - **Launch configuration name, Amazon Machine Image (AMI), instance type**
    - Launch configuration name, Amazon Machine Image (AMI), instance type, key pair
    - Launch configuration name, Amazon Machine Image (AMI), instance type, key pair, security group
    - Launch configuration name, Amazon Machine Image (AMI), instance type, key pair, security group, block device mapping 



- You are designing an e-commerce web application that will scale to potentially hundreds of thousands concurrent users. Which database technology is best suited to hold the session state for large numbers of concurrent users?
    - Relational database using Amazon RDS
    - **NoSQL database table using Amazon DynamoDB**
    - Data warehouse using Amazon Redshift
    - Amazon S3



- Which DNS record should you use to configure the transmission of email to your intended mail server?
    - SOA records
    - A records
    - **MX records**
    - CNAME records 



- You are working on a mobile gaming application and are building the leaderboard feature to track the top scores across millions of users. Which AWS services are best suited for this use case?
    - Amazon Redshift
    - Amazon ElastiCache using Memcached
    - **Amazon ElastiCache using Redis**
    - Amazon S3 



- When designing a loosely coupled system, which AWS services provide an intermediate durable storage layer between components? (Choose 2)
    - Amazon Cloud Front
    - **Amazon Kinesis**
    - Amazon Route 53
    - AWS CloudFormation
    - **Amazon Simple Queue Service (SQS)**



- Which type of DNS record should you use to resolve a domain name to another domain name?
    - An A record
    - **A CNAME record**
    - A D record
    - A PTR record



- Your application polls an Amazon SQS queue frequently and returns immediately, often with empty responses. What is one thing that can be done to reduce Amazon SQS costs?
    - Pricing on Amazon SQS does not include a cost for service requests; therefore, there is no concern.
    - Increase the timeout value for short polling to wait for messages longer before returning a response.
    - Change the message visibility value to a higher number.
    - **Use long polling by supplying a value for WaitTimeSeconds**



- Which AWS database service is best suited for traditional Online Transaction Processing (OLTP)?
    - Amazon Redshift
    - **Amazon Relational Database Service (RDS)**
    - Amazon ElastiCache
    - Amazon Neptune



- In the basic monitoring package for Amazon EC2, what Amazon CloudWatch metrics are available?
    - Web server visible metrics such as number of failed transaction requests
    - Operating system visible metrics such as memory utilization
    - Database visible metrics such as number of connections
    - **Hypervisor visible metrics such as CPU utilization**



- Which of the following is the Amazon side of an Amazon VPN connection?
    - An Elastic IP address (EIP)
    - A Customer Gateway (CGW)
    - An Internet Gateway (IGW)
    - **A Virtual Private Gateway (VPG)**



- How can you authenticate to a new Amazon Linux instance using SSH?
    - Decrypt the root password
    - Provide a username and password
    - **Use the private half of a key pair**
    - Use Multi-Factor Authentication (MFA)
    - Provide an Access Key and Secret Key



- What is needed to enable cross-region replication between two Amazon S3 buckets?
    - The buckets must be in the same AWS account
    - **Enable versioning on the buckets**
    - Enable static website hosting on the source bucket
    - The IAM User must have read access on the source bucket and write access on the destination bucket
    - Amazon S3 must be attached to an Internet Gateway



- Your company provides a mobile voting application for a popular TV show and 5-25 million viewers all vote in a 15-second timespan. What mechanism can you use to decouple the voting application from your back-end services that tally the votes?
    - Amazon ElastiCache
    - **Amazon Simple Queue Service (SQS)**
    - Amazon Redshift
    - Amazon Simple Notification Service (SNS)



- What type of AWS Elastic Beanstalk environment tier provisions resources to support a web application that handles background processing tasks?
    - Web server environment tier
    - **Worker environment tier**
    - Database environment tier
    - Batch environment tier

