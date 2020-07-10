# Notes for AWS Certified Solutions Architect Associate


## Contents

- [FAQs](#faqs)
- [Well-Architected Framework](#well-architected-framework)
- [VPC](#vpc-virual-private-cloud)
- [Route 53](#route53)
- [ELB and Autoscaling](#elb-and-autoscaling)
- [VPC VPN & Direct Connect](#vpc-vpn-direct-connect)
- [S3](#s3)
- [CloudFront](#cloudfront)
- [RDS, ElastiCache](#rds-elasticache)
- [Aurora](#aurora)
- [DynamoDB](#dynamodb)
- [In-Memory Caching](#in-memory-caching)
- [EC2](#ec2)
    - [EBS](#ebs)
    - [EBS Encryption](#ebs-encryption)
    - [Enhanced Networking](#enhanced-networking)
    - [Placement Groups](#placement-groups)
    - [Security Group](#security-groups)
    - [EC2 Monitoring](#ec2-monitoring)
    - [Instance Metadata](#instance-metadata)
    - [AMI](#ami-amazon-machine-images)
    - [Bootstrapping](#bootstrapping)
    - [EC2 Networking](#ec2-networking-eni-ip-dns)
    - [Instance Roles](#instance-roles)
    - [EC2 Billing](#ec2-billing)
    - [Dedicate hosts](#dedicated-hosts)
- [EFS](#efs)
- [Lambda](#lambda)
- [API Gateway](#api-gateway)
- [ECS](#ecs)
- [Identity Federation](#identity-federation)
- [STS](#sts-security-token-service)
- [SNS](#sns-simple-notification-service)
- [SQS](#sqs-simple-queue-service)
- [Elastic Transcoder](#elastic-transcoder)
- [Elastic Beanstalk](#elastic-beanstalk)
- [OpsWorks](#opsworks)
- [Storage Gateway](#storage-gateway)
- [IAM, Cognito and Directory Services](#iam-cognito-and-directory-services)
- [CloudWatch](#cloudwatch)
- [CloudWatch Events](#cloudWatch-events)
- [KMS and CloudHSM](#kms-and-cloudhsm)
- [Kinesis](#kinesis)
- [Athena](#Athena)
- [EMR](#emr)
- [Misc](#misc)



# FAQs

Main services to cover for the exam are:

Below services FAQs are very long but they will help alot in the exam. It's just a revision notes apart from above notes.

- [EC2](https://aws.amazon.com/ec2/faqs/)
- [S3](https://aws.amazon.com/s3/faqs/)
- [VPC](https://aws.amazon.com/vpc/faqs/)
- [Route 53](https://aws.amazon.com/route53/faqs/)
- [RDS](https://aws.amazon.com/rds/faqs/)
- [SQS](https://aws.amazon.com/sqs/faqs/)



# Well-Architected Framework

The five pillars are —

1. Operational Excellence 
2. Security
3. Reliability
4. Performance Efficiency
5. Cost Optimization


## Operational Excellence

### Design Principles

- Perform operations as code
- Annotate documents
- Make frequent, small, reversible changes
- Refine operations procedures frequently
- Anticipate failure
- Learn from all operational failures

### Best Practices

- Prepare
- Operate
- Evolve

__Key AWS Service__ — AWS CloudFormation.


## Security

### Design Principles

- Implement a strong identity foundations
- Enable traceability
- Apply security at all layers
- Automate security best practices
- Protect data in transit and at rest
- Keep people away from data
- Prepare for security events

### Best Practices

- Identity and Access Management
- Detective Controls
- Infrastructure Protection
- Data Protection
- Incident Response

__Key AWS Service__ — AWS Identity and Access Management (IAM).


## Reliability

### Design Principles

- Test recovery procedures
- Automatically recover from failure
- Scale horizontally to increase aggregate system availability 
- Stop guessing capacity
- Manage change in automation

### Best Practices

- Foundations
- Change Management
- Failure Management

__Key AWS Service__ — Amazon CloudWatch.


## Performance Efficiency

### Design Principles

- Democratize advanced technologies
- Go global in minutes
- Use serverless architecture
- Experiment more often
- Mechanical sympathy

### Best Practices

- Selection
    - Compute
    - Storage
    - Database
    - Network
- Review
- Monitoring
- Tradeoffs

__Key AWS Service__ — Amazon CloudWatch.


## Cost Optimization

### Design Principles

- Adopt a consumption model
- Measure overall efficiency
- Stop spending money on data center operations
- Analyze and attribute expenditure
- Use managed and application level services to reduce cost of ownership

### Best Practices

- Expenditure Awareness
- Cost-Effective Resources
- Matching Supply and Demand
- Optimizing Over Time

__Key AWS Service__ — Cost Explorer.



# VPC (Virual Private Cloud)

It's private data-center inside the AWS platform. Can be a mix of Public/Private.
    - Regional, Highly-available, can connect to on-premise networks
    - Isolated from other VPCs by default

## VPC and Subnet

Each Subnet can only be associated with single AZ.

Number of Subnets
- Max: __/16__ CIDR block __(65,536 IPs)__
- Min: __/24__ CIDR block __(16 IPs)__

__Default VPC__ --
- /16 CIDR block __(172.31.0.0/16) is default CIDR__
- /20 subnets in each AZ
- Pre-configured with all required networking/security
- Attached internet gateway with "main" route table sending all IPv4 traffic to the internet gateway using a 0.0.0.0/0 route
- A default DHCP option set attached
- SG: Default -- all from itself, all outbound
- NACL: Default -- allow all inbound and outbound

__Tenancy__ - Default is shared hardware, Dedicated is dedicated hardware.

Each Subnet will have __5 reserved IPs__:
- .0 network
- .1 Router
- .2 DNS
- .3 Future
- .X Broadcast (Based on the CIDR range last one is always Broadcast)


__DHCP Options set__ can be created, but can not be modified. Can be attached to one VPC only (1:1). If we want to change DHCP Options set, then need to create new one and associate by removing old one.

__DHCP Options set__ helps instances inside subnets getting private & public IPs allocated.


Subent can be shared with Other AWS accounts. Other accounts can provision resources in these Subnets.


## Routing and Internet Gateway

__VPC Routing__ (Route Table)
- Every VPC has a virtual routing device called the __VPC router__ -- __collection of routes__
- It has __an interface in any VPC subnet__ known as the __subnet+1__ address -- for 10.0.1.0/24, this would be 10.0.1.1/32
- The __router is highly available, scalable, and controls data entering and leaving the VPC__ and its subnet
- Each VPC has a __"main" route table__, which is allocated to all subnets in the VPC by default. __A subnet must have one route table.__
- "custom" route tables can be created and associated with subnets -- but only __one route table(RT) per subnet__.
- A route table __controls what the VPC router does with traffic__ leaving a subnet
- An internet gateway is created and attached to a VPC(1:1). It can route traffic for public IPs to and from the internet.

__Routes__
- A __RT is a collection of routes__ that are used when traffic from a subnet arrives at the VPC router.
- Every route table has a __local route__, which matches the CIDR of the VPC and __lets traffic be routed between subnets__. (Local route is Default route)
- A route contains a destination and a target. Traffic is forwarded to the target if its detination matches the route destination
- If __multiple routes apply for same destination__, he most specific is chosen. A /32 is chosen before /24, before a /16 (Bigger prefix takes prriority)
- Default routes(0.0.0.0/0 v4 and ::/0 v6) can be added that match any traffic not already matched
- targets can be IPS or AWS networking gateways/objects
- A __subnet is a public subnet__ if it is
    - If configured to allocate public IPs(__Auto-assign public IP__)
    - If the __VPC has an associated internet gateway__
    - If that __subnet has a default route to that internet gateway__

__Internet Gateway__ (IGW)
    - Attaches with single VPC(1:1) with 0.0.0.0/0 IP/Port
    - IGW connects with further public subnets
    - Responsible for Public internet connection for Subnets
    - Highly available by design
    - Responsible for packets re-routing to/from internet and public subnets
    - Performs __Static Network Address Translation (sNAT)__ (Instances doen't store public IP, done by IGW and process is call Static NAT)

__No AWS products has Public IPs stored. This is done by Internet gateway sitting in each VPC these resources are.__

__Best practices with IGW and Route Tables__:
- In route tables, always keep "Main" as it is.
- Create custom route table
- Add route from 0.0.0.0/0 to target IGW
- Them, associate public subnets to this route table

## Bastion Hosts // JumpBox

- A host that sits at the perimeter of a VPC
- It functions as an entry point to the VPC for trusted admins
- Allows for updates or configuration tweaks remotely while allowing the VPC to stay private and protected
- Generally connected to via SSH(Linux) or RDP(Windows)
- Bastion hosts must be kept updated, and security hardened and audited regularly
- Security: Multifactor authentication, ID federation, IP blocks
- Secure public access to private resources
- __Custom VPC doesn't enable public IPv4 DNS hostnames__, Need to enable DNS hostnames

__SSH Agent forwarding__ is used with Bastion Host to connect private instances with the same key, bu only from Bastion Host.

## NAT, NAT instance, NAT Gateway

Network address Translations is a __process where the source or desination attributes of an IP packet are changed__

Statis NAT is the process of 1:1 translation where an internet gateway converts a private address to a public IP address

__Dynamic NAT__ is a variation that __allows many private IP addresses to get outgoing internet access using a smaller number of public IPs__(generally one)

Dynamic NAT is provided within AWS using a NAT gateway that allows private subnets in an AWS VPC to access the internet

__NAT Best pracices__ For high availability Have each AZ one dedicated NAT gateway assigned. 2 AZ - 2 NAT gateway

Supports 5 GiB of bandwidth by default and support upto 45 GiB bandwidth.

__DNAT gateways__ provide two functions __for IPv4__ resources:
    - Sharing a __single public IP__ address for __private resources__
    - __Outgoing-only access__


## Network ACLs

- Controls the traffic between subnets.
- NACLs are collections of rules that can __explicitly allow or deny traffic__ based on its __protocol, port range, and source/destination__
- Rules are processed in __number orders, lowest first__. When a match is found, that action is taken and processing stops
- The __'*' rule is prcessed last__ and is an __implicit deny__
- NACLs have two sets of rules: __inbound__ and __outbound__
- __NACLs__ operates at __Layer 4(Transport)__ and __Security groups__ operate at __Layer 5(Session)__


## VPC Peering

- Allows __direct connections__ between __two isolated VPCs__ at __Layer 3__
- VPC Peering uses a peering connection, which is a __gateway object linking two VPCs__ __Private IP to Private IP__
- __Span AWS accounts and Regions__
- Data is encrypted and transits via the AWS global backbone.
- E.G. Company mergers, shared services, company and vendors, auditing

__VPC Peering Limitations__
- __VPC CIDR__ blocks __cannot overlap__
- __Only__ connect __two VPCs__ -- routing is not trasitive
- Rotes are required at both sides (Remove CIDR -> Peer Connection>)
- __NACLs and SGs__ can be used to __control access__
- __SGs__ can be __referenced but not cross-regions__
- IPv6 support is not available cross-region
- __DNS resolution__ to private IPs can be enabled, but it's a setting needed __at both sides__

__VPC peering__ connection route contains Target as __`pcx-xxxxxx`__.

__VPN connection__ // __Direct Connect__ connection route contains Target as __`vgw-xxxxxx`__.


## VPC Endpoints

- __Used to connect to AWS public sevices__ such as S3 (__without needeing for the VPC to have an attached internet gateway__)
- Are also __gateway object__ created within a VPC

VPC Endpoint __Types__
- __Gateway__ endpoints -- __DynamoDB, S3__
- __Interface__ endpoints -- Everything else (__SNS, SQS, KMS, logs, monitoring, Kinesis, Sagemaker, Glue__)

__When to use VPC EPs__
- If entire VPC is provate with no IGW
- If a specific instance has no public IP/NATGW and needs to access public services
- To access resources restricted to specific VPCs or endpoints (private S3 buckets)

VPC EP __considerations__
- Gateway endpoints
    - used via __route table entries__ -- they are __gateway devices__.
    - Prefix lists for a service are used in the destination field with the gateway as the target
    - __restricted via policies__
    - are HA acress AZs in a region
- Interface endpoints
    - Creates __Elastic Network Interfaces (ENI)__
    - IEs are Interfaces in a specific subnets
    - For HA, you need to add multiple interfaces -- one per AZ
    - __controlled via SGs__ on that interface
    - NACLs also impact traffic
    - __Add or replace the DNS for the service__ -- __no route table updates are required__
    - Code changes to use the endpoint DNS are required, or __enable private DNS to override the default service DNS__

## IPv6 in VPC

- Not available in every AWS products
- Currently opt-in -- disabled by default
- To use it -- request an IPv6 allocation. Each VPC is allocated a /56 CIDR from the AWS pool -- this cannot be adjusted
- With the VPC IPv6 range allocated, subnets can be allocated a /64 CIDR from the within the /56 range
- Resources launched into a subnet with an IPv6 range can be allocated a IPv6 address via DHCP6
- __IPv6 CIDR block range__ can be __attached to Operating System__

__Limitations and considerations__
- DNS names are not allocated to IPv6 addresses
- IPv6 addresses are all publicly routable -- there is no concept of private vs public with IPv6
- With IPv6, the OS is configured with this public address via DHCP6
- Elastic IPs aren't relevent with IPv6
- Not currently supported for VPNs, customer gateways, and VPC endpoints

## Egress-Only Gateway

- Works with IPv6
- Need to create a gateway object and add to VPC route table
- Once added to VPC route table, all associated subnets can now only access IPv6 internet traffic outside to VPC
- Egress only means, only outbound connections
- No IPv6 device can access/initiate-connection to these subnets from outside internet.


We cannot route traffic to a __NAT gateway__ or __VPC gateway__ endpoints through a __VPC peering__ connection, a __VPN connection__, or __AWS Direct Connect__. A NAT gateway or VPC gateway endpoints cannot be used by resources on the other side of these connections. Conversely, a NAT gateway // VPC gateway endpoints cannot send traffic over VPC endpoints, AWS VPN connections, Direct Connect or VPC Peering connections either.

Every route table contains a __local route__ for communication within the VPC over IPv4. We __cannot modify or delete__ these routes.

__VPC Endpoints always take precedence__ over NAT Gateways or Internet Gateways. 

Network ACL __rules are evaluated in order__, starting with the lowest numbered rule. As soon as a rule matches, it is applied regardless of any higher numbered rule that may contradict it.

SSH connections are between port 22 of the host and __an ephemeral port of the client__. In fact, this is true for any TCP service.

Security groups are __stateful__, this means any connection initiated successfully will be completed.

__NACLs__ are __stateless__. Yes! Network access control lists can allow/deny inbound/outbound traffic. That is just great!

We can create __S3 proxy server__ for enabling use cases where S3 has to be accessed privately through VPN connection, AWS Direct Connect or VPC peering.

AWS __reserves 5 IPs for every subnet__, not for every VPC.

Instances in __custom VPCs don't get public DNS hosts by default__, we have to set the `enableDnsHostnames` attribute to true. The `enableDnsSupport` is to be set to true too, but that is done by default.

We can set a __custom route table as the main route table__.

We can add __secondary CIDR ranges__ to an existing VPC. When a secondary CIDR block is added to a VPC, a route for that block with target as "local" is automatically added to the route table.

__VPN__ is established over a __Virtual Private Gateway__.

AWS __VPC Endpoints support S3 and DynamoDB__. For __Amazon ECR__, we have to use __AWS PrivateLink__.

__Difference between DirectConnect and VPN__ — DirectConnect does not involve the Internet, while VPN does.

__AWS Direct Connect__ doesn't __encrypt in transit data__, while __VPN__ does.

To establish a __VPN connection__, we need —
- A public IP address on the customer gateway for the on-premise network.
- A virtual private gateway attached to the VPC.

To setup __AWS VPN CloudHub__ —

- Each regional site should have non overlapping IP prefixes.
- BGP ASN should be unique at each site.
- If BGP ASN are not unique, additional ALLOW-INs will be required.

The __allowed block size__ in VPC is between a /16 netmask (65,536 IP addresses) and /28 netmask (16 IP addresses).

 The following __VPC peering connection configurations__ are __not supported__ —

1. Overlapping CIDR Blocks
2. Transitive Peering
3. Edge to Edge Routing Through a Gateway or Private Connection

We can move part of our __on-premise address space to AWS__. This is called BYOIP. For this, we have to acquire a __ROA, Root Origin Authorization__ from the the regional internet registry and submit it to Amazon.



# Route53


__Main functions of Route53__ —
1. Register domain names.
2. Route internet traffic to the resources for your domain.
3. Check the health of your resources.

It's not used to _distribute_ traffic.

The __host (www) portion__ is not included in a __public zone's__ naming convention.

DNS is a domain name service that maintains a list of domain names and translates them to IP addresses.

__CNAME vs ALIAS__ —  

- For routing to S3 bucket // Elastic load balancer use A record with ALIAS.
- For routing to RDS instance use CNAME with NO ALIAS // without ALIAS.

ALIAS only supports the following services —
- API Gateway
- VPC interface endpoint
- CloudFront distribution
- Elastic Beanstalk environment
- ELB load balancer
- S3 bucket that is configured as a static website
- Another Route 53 record in the same hosted zone

ALIAS subdomains query is free within AWS and is only available for above mentioned services.

Route53 does not directly log to S3 bucket, we can forward that from Cloudwatch, but can't do it directly.

Types of __Route53 health checks__ —
1. Health checks that monitor __an endpoint__. This __can be on-premise__ too.
2. Health checks that monitor __other health checks__.
3. Health checks that monitor __Cloudwatch alarms__. 

__Multivalue answer routing policy__ responds with upto 8 healthy records selected at __random__.

__Weighted routing policy__ is a good fit for __blue-green deployments__.

__Private vs. Public Hosted zones__

- Public Hosted zone is created when registered new domain with Route 53 or migrated to Route 53, or create one manually
- Hosted zone will have "name servers" -- these are the IP addresses you can give to a domain operator, so Route 53 becomes "authoritive" for a domain.

- Private zones are created manually and associated with one or more VPCs -- they are only accessible from those VPCs
- Private zones need __enableDnsHostnames__ and __enableDnsSupport__ enabled on VPC
- Not all Route 53 features are supported --limits on health checks
- Split-view DNS is supported, using the same zone name for public and private zones -- providing VPC resources with different records(e.g. esting, internal versioning of websites).
    - with split-view, private is proferred -- if no matches, public is used
    - __split-view DNS__ means hosting domain in both private and public zones with different targets

__Record Set Types__

Below record types are available __within DNS__ and Route 53

- __A records__ -- IPv4 address for given host(www)
- AAAA records -- IPv6 address
- __CNAME records__ -- Allows aliases to be created (Like: www, ftp, images for subdomain)
- MX records -- Mail servers for a given domain
- NS records -- Used to set the authoritative servers for a subdomain.
- __TXT records__ -- Used for descriptive text in a domain -- often used to verify domain ownership (Gmail/Office365)
- __Alias record__  -- An extension of CNAME -- can be used like an A record, with the functionality of a CNAME and none of the limitations. Can refer to AWS logical services (Load balancers, S3 buckets), and AWS doesn't charge for queries of alias records against AWS resorces.

__Health Checks__

Health checks are __charged per month__ and are based on the __number of checks within the month__.

Route 53 provides health checks that allow endpoints to be evaluated based on user-definable criteria. Health checks are used for advanced Route 53 functionality, such as routing policies and failover.

__Tyes of Health checks__

- There is cost associated but gives better monitoring and used widly with different routing policies.
- __HTTP or HTTPS__, tcp/80 or tcp/443 in less than __4 seconds__
- Reponses 2XX or 3XX code within two seconds
- TCP health check: tcp connection within 10 seconds
- HTTP/S with string match: All the checks as with HTTP/HTPS but the body is checked for string match

- Records can be linked to health checks. If the check is unhealthy, the record isn't used
- Can be used to do failover and other routing architecturess

__Routing Policies__ (Advanced Route 53)

- __Simple__
    - only single record can be created with primary like www.example.com
    - Health chekcs can not be associated with failover routing ploicy
- __Failover__ based on health checks on primary record
    - www(Primary) (www.example.com to EC2 instance)
    - www(Secondary) ((www.example.com to Alias of ELB, S3, Lambda, API Gateway))
    - Health chekcs can be associated with failover routing ploicy
    - can create multiple record sets for hosted zone with each failover routing policy
    - Only single Primary and Secondary records
- __Weighted__ routing policies allow granular control over queries, allowing a certain percentage of queries to reach specific records
    - multiple records with weighted percentages in total of 100
    - resolutions at Route 53 happens based on above weightages. This is not a load balancer, only for new test feature deployments.
- __Latency__ routing policy allows clients to be matched to resources with the lowest latency means __based on Network conditions__
    - Based on regions the resource is deployed
    - Route53 maintains the table for incoming source region of request and desnination Region of the resource. And responses with low latency resource record.
- __Geolocation__ routing policy allows to chose the resources that serve your traffic based on the geographic region from which queries originate.
    - multiple record set is used and record set takes priority based on geolocation
    - One record set can be set as default that gets returned if the IP mathcing process fails or no record set is configured for the originating query region
    - GeoProximity allows a bias to expand a geographic area
    - Different customers gets different content based on geolocation



# ELB and Autoscaling

Classic LB(Old), Application LB(layer 7 recommended general purpose), NLB(Layer 4, low latency, packets unchanged)

__Patching an AMI for an auto scaling group__, the procedure is —  
1. Create an image out of the main patched EC2 instance
2. Create a new launch configuration with new AMI ID
3. Update auto scaling group with new launch configuration ID. 

Note that AMI ID is set during creation of launch configuration and cannot be modified, so we have to create a new launch configuration.

__Default metric types for a load balancer__ —
1. Request count per target.
2. Average CPU utilization.
3. Network in.
4. Network out.

__Monitoring Application Load Balancers__ —
1. Cloudwatch metrics
2. Access logs
3. Request tracing
4. Cloudtrail logs.

Adding __lifecycle hooks__ to ASGs put instances in __wait state__ before termination. During this wait state, we can perform custom activities. Default wait period is 1 hour.

__Cooldown Policy__ is The amount of time allowed for all scaling activities before any additional provisioning or de-provisioning can occur.

ASG __Dynamic Scaling Policies__ —
- Target tracking scaling. The __preferred__ one to use, this should be the first one we should consider.
- Step scaling
- Simple scaling
    - Simple scaling policy requires cooldown period to expire before responding to additional alarms

If you are scaling based on a utilization metric that increases or decreases proportionally to the number of instances in an Auto Scaling group, we recommend that you use target tracking scaling policies. Otherwise, we recommend that you use step scaling policies. 

The ELB service does not consume an IP address, it's the nodes that consume one IP address each.

__Auto-scaling__ ensures —
- Fault tolerance
- Availability

__ELBs__ can manage traffic within a region and not between regions.

__For unstable scaling behavior__, that is scaling multiple times frequently, the following things can be done —

- Increasing __auto-scaling cooldown timer__ value would give scaling activity sufficient time to stabilize.
- Modify the __cloudwatch alarm period__ that triggers scaling activity.

__Default cooldown period__ is 300 seconds.

__Port based routing__ is supported by __Application Load Balancer__.

__Network Load Balancer__ can be used to __terminate TLS connections__. For this, NLB uses a security policy which consists of protocols and ciphers. The certificate used can be provided by __AWS Certificate Manager__.

__Connection draining__ enables the load balancer to complete in-flight requests made to instances that are de-registering or unhealthy. 

ASG(Auto scaling group) __termination policy__ —

1. Oldest launch configuration.
2. Closest to next billing hour.
3. Random.

Load balancer does not create or terminate instances, that's done by auto scaling group.

__Classic load balancers__ (Old school)

- A Classic Load Balancer's listener accepts traffic based on Ports and Protocols
- Supports EC2 classic instances
- CLBs do not provide path-based routing because it is not a true layer 7 load-balancer like an ALB.

__Application Load Balancers__

- EC2 --> Load Balancers -> Targets -> Target Groups -> Add Targets of Instances -> Content Rules
- Works mainly with Listener Rules
- operate at layer 7 of the OSI model
- If a below rule match then target group is served
- Content rules can direct certain traffic to specific target groups
    - Host-based: based on the host used
    - Path-based: based on URL path
    - HTTP header
    - Http request method
    - Query string
    - Source IP
- __ALBs support Target types__ EC2, ECS, EKS, Lambda, HTTPS, HTTP/2 and WebSockets, and AWS WAF (Web Application Firewall)
- An ALB can host __multiple SSL cerificates using SNI__
- Health checks can be configured to custom port or default traffic post

__Network Load Balancers__ can scale to millions

- Does not support Host based and Path based routing
- Everything there is from ALBs and much more
- layer 4 on OSI model
- can support protocols other than HTTPS because it formawrds upper layer unchanged
- less latency because no processing above Layer 4 is required
- IP addressable -- static address
- Best load balancing performance within AWS
- Sends packates unouched to forwarded target group
- Targets can be addressed using IP address

__Launch Templates and Launch Configurations__

Allows to configure various configuration attributes that can be used to launch instances.

- AMI to use for EC2 launch
- Instance Type
- Storage
- Key pair
- IAM role
- User data
- Purchase options
- Network configuration
- Security groups

Extended features of __Lauch Templates__ are as below

- Versioning and inheritance
- Tagging
- More advanced purchasing options
- New instance features, including:
    - elastic graphics
    - T2/T3 unlimited settings
    - Placement groups
    - Capacity reservations
    - Tenancy options

__Neither can be edited after creation__ -- a new version of the template or a new launch configuration should be created

__Auto Scaling Groups__

- Min, Max and desired number of EC2 instances
- provided availablity zones subnets are used to evenly distribute the running instances
- Scale-in or scale-out automatically with associated lauch configuration or launch templates
- __Metrics__ such as CPU utilization or network transfer
- Scaling can be __manual, scheduled or dynamic__
- __colldowns__ can be defined to ensure rapid in/out events don't occur
- Scaling policies can be __simple, step scaling, or Target tracking__
- In simple, we can configure alarm for some metrics and then add remove instances from the scaling group
- __ALB__ can be configured with __ASG__ with __Target Group__
- evenly distributing EC2 instances over multiple AZs and their subnets.



# VPC VPN Direct Connect

__connect a VPC to a non-AWS network__, such as on-premises or data center. It is a highly available solution that can be configured to use either static or Border Gateway (BGW) routing.

- End to End encryption and cheaper HA service
- Latency and throughput issues may occur

Componenets:
- A virual Private Cloud(VPC)
- Virtual Private Gateway(VGW) attached to a VPC
- A Customer Gateway(CGW) - configuration for on-premises router
- VPN Connection (using 1 or 2 IPsec tunnels)

Best Practice & High Availablity
- Use dynamic VPNs(Uses BGP) where possible
- COnnect both tinnels o your CGW - VPC VPN is HA by Design
- Where possible use two VPN connections and two CGWs

__Routing Table priorities__
- Local route
- Longest prefix
- static routes always wins after local routes
- Direct connects always preffered over VPN
- __Direct connect BGP__ to __static VPN__ to __BGP VPN__


__Direct Connect(DX)__

high-speed, __low-latency physical connection__ providing access to public and private AWS services from your business premises

Dedicated __1 Gbps or 10 Gbps__ from DX locations.

It's not highly available or encrypted.

Provides __private VIF(VPCs)__ and __public VIF(S3, DynamoDB, SNS, SQS)__.

A private virtual interface allows you to __connect from your on-premises equipment to a specific VPC__.

Multiple VIFs can be configured. Also possible to use VPN along with DX.



# S3

In a newly created S3 bucket, everything // every additional option is turned off by default. Also, no bucket policy exists.

__S3 bucket properties__ are —
1. Versioning
2. Server access logging
3. Static website hosting
4. Object level logging // Essentially CloudTrail
5. Transfer acceleration
6. Events

__Object level properties__—  
Metadata and Storage class are object level properties. All object level properties are
1. Storage class
2. Encryption
3. Metadata
4. Tags
5. Object lock

__DELETE operation__ does not keep a copy unless you have versioning enabled. From the docs
> The DELETE operation removes the null version (if there is one) of an object and inserts a delete marker, which becomes the current version of the object. If there isn't a null version, Amazon S3 does not remove any objects. 

S3 is a __managed service__. It can't be part of a VPC.

__S3 object metadata__—
1. System metadata
2. User-defined metadata

User defined metadatas must start with `x-amz-meta`.

When you enable logging on a bucket, the console both enables logging on the source bucket and adds a grant in the target bucket's access control list (ACL) granting write permission to the Log Delivery Group.

__S3 bucket endpoints formats__ —
1. http://bucket.s3.amazonaws.com
2. http://bucket.s3.aws-region.amazonaws.com
3. http://s3.amazonaws.com/bucket
4. http://s3.aws-region.amazonaws.com/bucket

__Object sizes__ —
S3 can store objects of size 0 bytes to 5 TB.
A single PUT can transfer 5 GB max. For files larger than 100MB, multipart upload is recommended.

__Cross-region replication__ requires that versioning be enabled on both the source bucket and the destination bucket.

__AWS Glacier archive retrieval__ options —
- Expedited: Costly, 1-5 minutes.
- Standard: Default, 3-5 hours.
- Bulk: Cheapest, 5-12 hours.

To increase performance, we can __prefix each object name with a hash key__ along with the current date. But, according to the new S3 performance announcement, this is __not needed anymore__.

__Increasing performance in S3__ —
- If workload is mainly GET requests, integrate Cloudfront with S3.
- If workload consists of PUT requests, use S3 transfer acceleration.

In the __CORS__ configuration, the __exact URLs__ must be added, with the correct protocol, i.e. __http vs https__.

__S3__ does not support `OPTIONS`, `CONNECT` and `TRACE` __methods__. 

__S3 encryptions__ —
- SSE-S3: Data and master keys managed by S3.
- SSE-C: The user manages the encryption keys.
- SSE-KMS: AWS manages the data key, the user manages the master key.

We can create __event notification in S3__ to __invoke lambda__ function.

__Customer managed S3 encryption workflow__ —  
Generate a data key using Customer managed CMK. Encrypt data using data key and delete data key. Store encrypted data key and data in S3 buckets. For decryption, use CMK to decrypt data key into plain text and then decrypt data using plain text data key.

AWS S3 __performance__ —

- 3,500 requests per second to add data
- 5,500 requests per second to retrieve data

__Provisioned capacity__ should be used when we want to guarantee the availability of fast expedited retrieval from S3 Glacier within minutes.

For __S3 static website hosting__, the default provided __URL__ is https://bucket-name.s3-website-aws-region.amazonaws.com.

S3 server side encryption uses __AES 256__.

S3 __event notification targets__ —

- SQS
- SNS
- Lambda

For __static website hosting__ with S3, the name of the bucket must be the same as the domain or subdomain name.

__Preventing accidental deletion__ of S3 objects —

- Enable versioning
- Enable MFA delete

The following services enable us to __run SQL queries directly against S3 data__ —

- AWS Athena
- Redshift Spectrum
- S3 Select



# CloudFront

To make sure that S3 objects are only accessible from Cloudfront, create an __Origin Access Identity (OAI) for Cloudfront__ and grant access to the objects to that OAI.

The following __CloudFront events can trigger lambda function__ —
- Viewer request
- Viewer response
- Origin request
- Origin response

For __CloudFront query string__ forwarding, the parameter names and values used are __case sensitive__.

We can use __signed URLs and signed cookies with Cloudfront__ to protect resources.

Slower login time and 504 errors in front of Cloudfront can be optimized by —

- Lambda @ Edge.
- Setting up an Origin Failover Policy.

__AWS Shield__ is a service that protects resources against DDoS attacks to EC2, ELB, Cloudfront and Route53.

__Perfect Forward Secrecy__ is supported by —

- Cloudfront
- Elastic Load Balancing

Enabling __multiple domains to serve HTTPS__ over same IP address —- Generate an SSL cert with AWS Certificate Manager and create a Cloudfront distribution. Associate cert with distribution and enable Server Name Indication (SNI).

Classic Load Balancer does not support __SNI__, we have to use Application Load Balancer or Cloudfront.



# RDS

RDS is relational and OLTP(Online transactions processing) database

__RDS data size limits__ —
- Aurora: 64 TB
- Others: 16 TB

During automated backup, Amazon RDS performs a storage volume snapshot of entire Database instance. Also, it captures transaction logs every 5 minutes.

__AWS RDS is a service__ whereas __AWS Aurora is a database engine__.

__Encryption of RDS__ — Have to enable it on database creation. Also, not all instance classes support encryption, we have to choose one which supports it.

To enable __multi-region replication of RDS__, we have to use __Read Replicas__. Multi-AZ does not provide cross-region read replicas.

__RDS Read Replicas__ are __synced asynchronously__, so it can have __replication lag__.

We can't use auto-scaling with __RDS__. To improve __performance__, we should look to __sharding__ instead. Starting from __June 20__, we __can use auto-scaling__ with RDS instances.

We configure __RDS engine configurations__ using __parameter groups__.

To use __REDIS AUTH with ElastiCache__, __in-transit encryption__ must be enabled for clusters.

For RDS, __Enhanced Monitoring__ gathers its metrics from an __agent on the instance__.

In case of a __failover__, Amazon RDS flips the canonical name record (__CNAME__) for your DB instance to point at the standby in Multi-AZ. Failover in Read-replicas works differently and faster than Multi-AZ. __CNAME__ in both cases are retained. In RDS DB is __never overwritten__, always created new on from snapshots or promoted a read replicas to be a primary one.

__Aurora endpoints__, by default — 

- A reader endpoint. It load balances all read traffic between instances.
- A cluster endpoint. For write operations.

We can create __additional custom endpoints__ that load balance based on specified criteria.

The memory and processor __usage by each process__ in an RDS instance can not be monitored by Cloudwatch, we have to use __RDS Enhanced Monitoring__ for that. Because Cloudwatch monitors the hypervisor, not the individual instances.

__IAM DB authentication__ can be used with __MySQL and PostgreSQL__. With this, you don't need to use a password when you connect to a DB instance. Instead, you use an __authentication token__.

__RDS__ main components are:

- Managed DB
- Multi AZ (FT, RTO, Syncronous backup)
- Read Replicas (Availability, Read scale, Async backups)
- Automatic backups
- Security Groups, VPC Network, private-public Subnets
- DNS (CNames)

__RDS supports number of Database engines:__

- MySQL, PostgreSQL, MariaDB, Oracle, Microsoft SQL Server
- Aurora: An in-house developed engine with substantial features and peformance enhancements

__RDS can be deployed in single-AZ or Multi-AZ mode(for resilience) and supports the following instance types:__

- General Purpose(currently DB.M4 and DB.M5)
- Memory optimized(DB.R4, DB.R5 and DB.X1e, DB.X1 for Oracle)
- Burstable(DB.T2 and DB.T3)

__RDS Two types of storage are supported:__

- General Purpose SSD(gp2): 3 IOPS per GiB, burst upto 3,000 IOPS(pool architectire like EBS)
- Provisioed IOPS SSD(io1): 1,000 to 80,000 IOPS(engine dependent) size, and IOPS can be configured independently

__RDS instances are changed based on:__

- Instance size
- Provisioned storage(not used)
- IOPS if using io1
- Data transferred __out__
- Any backups/snapshots beyond the 100% that is free with each DB instance

__RDS supports encryption with the following limits/restrictions/conditions:__

- Encryption can be configured when creating DB instances
- Encryption can be added by taking a snapshot, making an encrypted snapshot, and creating a new encrypted instance from that encrypted snapshot.
- Encryption cannot be removed
- Rad replicas need to be the same state as the primary instance(encryption or not)
- Encryped snapshots can be copied between regions -- but a new destination region KMS CMK is used(because they are region specific)

__Network access__ to an RDS instnace is controlled by a __security group (SG)__ associated with the RDS instance.


# Aurora

Aurora is a custom-designed __relational database engine__ that forms part of RDS. Rather than an evolution, Aurora significantly replaces much of the traditional MySQL and PostgreSQL architecture in favor of __cluster__ and __shared storage architecture__, which is __more scalable and resilient__ with much higher performance.

- Location can be a regional cluster or global cluster
- A cluster contains a single primaryinstance and zero or more replicas
- Multiple Aurora Read replicas instances(Max 16) are like something in RDS and operates between __Standby and Read Replicas__
- __Automatic Failover promotions__ up to 16 tiers based on set priority on Read instances
- Has subnet groups, VPC, security groups just like RDS
- All read replicas can be multi-az

__cluster storage__

- All instances (primary and replicas) use the same shared storage -- the cluster volumes.
    - cluster volume is totally SSD based, which can scale up o 64 TiB in size
    - replicates data six times, accross three Availablity zones
    - Aurora can tolerate two failures without writes being impacted and three failures without impacting reads
    - Aurora storage is auto-healing

__Cluster Scaling and Availability__

- Cluster volumes scales automatically, only bills for __consumed data__, and is constanly backed up to S3
- Aurora replicas improves availablity, can be promotated to be a primary instance quickly, and allow for efficient read scaling
- Reads and writes use the cluster endpoint
- Reads can use the __read endpoint__, which balances connections over all replica instances.
- Backtracking an Aurora DB cluster can rollback upto 72 hours, It'll have down time

__Aurora parallel queries and Aurora Global__

- Multiple readers with parallel query operations, larger compute or big query operation
- Global DB locations where DB is replicated over another region in seconds and provides faster reads in global areas
    - expensive and only memory optimized are available with r4 and r5 series instances

__Aurora Serverless__

Aurora Serverless is based on the same database engine as Aurora, but instead of provising cerain resource allocation, Aurora Serverless handles this as a service.

With Aurora Serverless, you indicate your minimum and maxiumum load levels with __Aurora Capacity Units(ACUs)__, and the product scales based on the __incoming load__. Aurora Serverless is also able to scale down to zero, where the __only cost is storage__.

- Aurora serverless can use the Data API
- On demand auto-scaling. So, Aurora serverless is a Aurora as a service.
- No Admin overhead. Scales in capacity. Horizontal scaling.
- Scales using Proxy fleets.
- Can pause compute after consecutive minutes of inactivity
- Any Aurora snapshots can be restore as Aurora serverless
- AWS __Query Editor__ inside RDS only works with Aurora Serverless instances and connects with Data API.
- You can't access an __Aurora Serverless DB cluster's endpoint through an AWS VPN connection__ or an __inter-region VPC peering__ connection.
- Cluster volumes are billed for __consumed__ data



# DynamoDB

AWS __DynamoDB__ is NoSQL serverless database service which is durable, ACID compliant, can go through multiple schema changes, and changes to the database does not result in any database downtime.

__DynamoDB Global Tables__ can be used to deploy a multi region(paitioned regionally), multi AZ, fully managed database solution. (Resilient in single region only)

Table can have single partition key(PK) or a composite key (Partition key(hash key) and sort key(range key)

An item in a given Table can have __max size__ of 400 KB. Read and Write of a item is also 400 KB.

We can create __secondary indexes__ for __DynamoDB__ tables. Always choose DynamoDB when possible.

__DynamoDB streams__ can be used to monitor changes made to a database, and they can trigger lambda functions.

IAM roles needs to have access to Dynamo DB in order to query GET and PUT to tables.

By default we have three copies of DB with Dynamo DB. All PUT are persistance and return 200 OK status code while using the Dynaomodb APIs.

__Scan Operations__(reads every items, thuus comsumes more capacity units, can filters on different attributes) and __Query__(more efficient, allows to lookup without reading every items, Only search on partition key and filter on Sort key with ASC DSC sorting)

We can turn on __autoscaling for DynamoDB__.

For __write heavy__ use cases in __DynamoDB__, use partition keys with large number of distinct values.

__DynamoDB Accelerator, DAX__ is an __in-memory cache for DynamoDB__ that reduces response time from milliseconds to microseconds.

__Dynamo DB Properties__

- Primary Parttion key, Sort key
- Encryption type (Default enabled with AWS managed CMK(Customer Master Key) and KMS CMK)
- KMS master key ARN
- Backups with __Point in time recovery__ (Dynamodb creates automatic backups of tables for the __35 days__)
- __On-Demand backup and Restore__ (Create manual backup and manual restore DB)
    - restoring backup creates a new table
- __Global Tables__ feature needs to have enabled stream
    - table needs to be empty in order to enable Global tables feature for dynamo db table
    - add read replicas regions
    - Multi Regions and Multi-master replication process
    - All of the tables can be read and written to, last write wins and automatic replications happens in global replicas
- __Read capacity units__, __Write capacity units__
- Amazon Resource Name (ARN) for table resource type database
    - arn:aws:dynamodb:us-east-1:account_id:resource_type/db_name

__Dynamo DB Monitoring__

- has Metrics such as __Capacity__, __Latency__, __Streams__, __TTL__, __Scan and Query__, __Errors__ from __Cloud Watch__.

__Dynamo DB performances and Billing__

- Has two read/write capacity modes: __provisioned throughput__(default) and on-demand mode.
- When using on-demand mode, DynamoDB automatically scales to handle performance demands and bills a per-request charge.
- In provisioned throughput mode, each table is configured with __RCU__ and __WCU__ (each operation on table consumes atleast one RCU or WCU)
- RCU is 4 KB of data read per second in a stongly consistent way
- Atomic operations required 2x the RCU and 2x the WCU
- WCU is 1 KB of data or less written to a table
- Partitions are also raplicated into three availablity zones
- Three az nodes, one leader node, other nodes are eventually consistent, leader node is strongly consistence

__Dynamo DB Streams and Triggers__

- KEYS_ONLY
- NEW_IMAGE
- OLD_IMAGES
- NEW_AND_OLD_IMAGES

- Triggers from stream based on changes to invoke lambda function to perform certain actions

__Dynamo DB Indexes__

- LSI __local secondary indexes__ (maximum 5 per table)
    - must be created at the same time as creating table
    - use the same partition key but an alternative sort key
    - they share the RCU and WCU values for the main table

- GSI __global secondary indexes__ (Max 20 GSIs,  GSIs can be used to support alternative data access patterns, allowing efficient use of query operations.)
    - can be created at any point after the table is created
    - can use different partition and sort keys
    - have their own RCU and WCU values



# In-memory Caching

__DAX__

Mainly used with DynamoDB

__Elastic Cache__

Amazon __ElastiCache__ offers fully managed __Redis and Memcached__. 



# EC2

EC2 is IaaS (Infrastructure as a service) product. Provides virtual machines known as instances.

Creation components:
- OS Image (Linux 2 AMI, Ubuntu, Redhat, windows)
- Instance Type (Compute and Memory)
- Networking (VPC, Subnets)
- Key-pairs
- Tags
- Instance storage, EBS
- __Security Group__ (ENI - __Elastic Network Interface__ attached to EC2)
    - Instance's network card
    - Default __SSH 22 is allowed for All__

__Hibernation of EC2 instances__ —
- When EC2 instance is hibernated and brought back up, the public IP4 address is renewed. All the other IP addresses are retained.
- When EC2 instance is in hibernate, you are only charged for elastic IP address and EBS storage space.

__Reserved Instances that are terminated__ are **still billed** until the end of their term according to their payment option.

Upon __stopping and starting an EC2 instance__ —

- Elastic IP address is disassociated from the instance if it is an EC2-Classic instance. Otherwise, if it is an EC2-VPC instance, the Elastic IP address remains associated.
- The underlying physical host is possibly changed.

For __new accounts__, Amazon has a __soft limit of 20 EC2 instances per region__, which can be removed by contacting Amazon.

You can attach a __network interface (ENI)__ to an EC2 instance in the following ways —

1. When it's running. __Hot attach__.
2. When it's stopped. __Warm attach__.
3. When the instance is being launched. __Cold attach__.

__Instance Types and Sizes__

__EC2 instance Families__: General Purpose, Compute optimized, Memory optimized, Storage Optimized, Accelated Computing

__Types__ - certain set of features

- __T2, T3__ low-cost, burst capability
- __M5__ General workloads
- __C4__ CPU(Central processing unit) Optimized
- __X1, R4__ Large Memory optimized
- __I3, D2, H1__ IO optimized (storage optimized)
- __P3, P2, G3, F1__ GPU(Graphical processing unit) and FPGAs(Field programmable GAs, programmable hardware)

__Size__ - level of workload to cope with __nano, micro, small, medium, large, x.large, 2x.large, larger__

__a__ - Use AMD CPU
__A__ - ARM based
__n__ - Higher speed networking
__d__ - NVMe storage

__**Instance store volumes**__ 
- Physically attached while creating EC2 instance
- You cannot add instance store volume to an instance after it's launched.
- Not all EC2 instance types support instance store volume.
- Persistence — Instance store persists during reboots, but not stop or terminate. EBS volumes however persists accross reboot, stop, and terminate.
- Comes with the price of the Instance, They get mounted while creation only
- `df -h`
- `lsblk`

## EBS

Attached to EC2 via a storage network. Persistent, Attached & Removed, Replicated within a single AZ

__EBS volume types__ —
1. General purpose SSD. For web applications // most use cases.
2. Provisioned IOPS SSD. For critical high performing databases.
3. Throughput optimized HDD. For Big Data.
4. Cold HDD. For infrequently accessed data.

**__EBS volume types__** —

__IOPS__ -- the number of read and write operations per second
__throughput__ -- IOPS x block size

- General purpose *SSD* __gp2__
    - Default, balance of IOPS/MiB/s - burst pool IOPS per GB
    - 3 IOPS/GiB(100 IOPS - 16,000 IOPS)
    - Bursts up to 3,000 IOPS (credit based)
    - 1 GiB - 16 TiB size, max throughput p/vol of 250 MiB/s
- Provisioned IOPS *SSD* __io1__
    - Highest performance, For large number of transaction
    - Large database workloads
    - Volume size of 4 GiB - 16 TiB up to 64,000 IOPS per volume
    - Max throughput p/vol of 1,000 MiB/s
- For throughput, Throughput optimized HDD __st1__
    - Mechanical
    - Frequently accessed, thoughput intesive workloads
    - Cannot be a boot volume
    - used for streaming, big data
    - Volume size of 500 GiB - 16 TiB
    - Per-volume max throughput of 500 MiB/s and IOPS 500
- Cold HDD (__sc1__ Mechanical, lowest cost, infrequent access)
    - lowest cost
    - Infrequently accessed data
    - Cannot be a boot volume
    - Volume size of 500 GiB - 16 TiB
    - Per-volume max throughtput of 150 MiB/s and 250 IOPS
- Magnetic (standard)
    Legacy. Not used any more.


## EBS Encryption

__EBS Enrypion__ with DEK(KMS CMK):
    - Volume encryption uses EC2 host hardware to encrypt data __at rest__ and __in transit__ between EBS and EC2 instances. Encryption generates a __data encryption key (DEK)__ from a __customer master key(CMK)__ in each region.
    - A unique DEK encrypts each volume.
    - Encrypted EBS volumes are not supported on all instance types.
    - Encryption can be enabled on EC2 setting panel from the right-side of console.
    - When encrption is enabled, All new EBS instances are encrypted by default. And every snapshot created are also encrypted using the same KMS CMK key.
    - Encrypted DEKs stored with volume are decrypted by KMS using a CMK and given to the EC2 host.
    - Plaintext DEKs stored in EC2 memory and used to encrypt and decrypt data
    - KMS is regional service, so when Snapshot is copied to another region, new CMK has to be provided manually.

__EBS-optimized EC2 instances__ provide additional, dedicated capacity for EBS IO. Helps squeeze out the last ounce of performance.
    - Was __historically-optional__ and is __now the default__
    - Adds optimization and dedicated __communication paths for storage and tradition data networking__
    - This allows consistent utilization of both -- and is one required feature to support higher performance storage

When EBS volume is __created from a Snapshot__, all __data is copied in backgroud from S3__. This is slow process and impacts EBS performance.

Also, to note, __HDDs cannot be boot volumes__.

We can use Amazon __Data Lifecycle Manager__ to automate taking backups // snapshots of EBS volumes, and protect them from accidental or unwanted deletion.

EBS is __lower-latency__ than EFS.

The maximum ratio of __provisioned IOPS__ to requested volume size (in GiB) is 50:1.

__EBS snapshots__ are more efficient and cost-effective solution compared to __disk mirroring using RAID1__.

EBS snapshots can help acheive RPO(Recovery point objective) and RTO(Recovery time objective).

EBS volumes can only be attached to an EC2 instance in the __same Availability Zone__.

__EBS snapshot creation__ — In usual scenarios EBS volume snapshots can be created at the same time it's in usage. But when using RAID configurations, there are additional complexities and we should stop every IO operation and flush the cache before taking a snapshot.

With __scheduled reserved instances__, we can plan out our future usage and get reserved instances in those planned time-frame only.

__Throughput optimized HDD vs Cold HDD__ — Throughput optimized is used for frequently accessed data, whereas Cold HDD is used for infrequently accessed data. Also the later is more cost-effective.

__RAID0 vs RAID1__ —

- RAID1 is used for mirroring, high-availability and redundancy.
- RAID0 is used for higher performance, it can combine multiple disk drives together.

Larger EC2 instances have higher disk data throughput. This can be used in conjunction with RAID 0 to __improve EBS performance__.

## Enhanced Networking

Enhanced networking uses __SR-IOV__ Single Root Input-Output Virtulization, which allows a single physical network card to appear as multiple physical devices.
    - Each instance can be given one of these fake physical devices.
    - This results in faster transfer rates, lower CPU usage and lower consistent latency
    - EC2 delivers this via the __Elastic Network Adapter(ENA)__ or Inter 82599 Virtual Function(VF) interface

__To get more performance out of EBS volumes__ —
1. Use a more modern Linux Kernel.
2. Use RAID 0.
3. Placement groups
4. Enhanced Networking via ENA or Intel Virual Function

__VolumeRemainingSize__ is not an Cloudwatch metric for EBS volumes.

By default, __EBS volumes are automatically replicated within their availability zone__, and offers a significant high availability.

With __EC2 dedicated hosts__ we have control over __number of cores__, not anywhere else.

## Placement Groups

__Placement groups__ — Have to create PG first and then create EC2 instances in PGs
- __Cluster PG__
    - designed for performance improvement
    - only one availabilty zone
    - place instances physically near each other is a single AZ
    - Works with enhanced networking for peak performance
    - Might hit physical number of available slot for number of instances
- __Spread PG__
    - Maximum availability
    - small infrastructure system designs
- __Partitioned PG__
    - High availability
    - Multi-AZ patitions of instances
    - Sutable for large infrastructure
    - Maximum number of instances in an AZ is 7

The __console does not support placement groups__, have to do it from CLI.

__Cluster Placement groups__ have very __low inter-note latency__.


## Security Groups

- Software firewalls that can be attached to network interfaces of EC2 and VPCs
- Several instances can belong to the same security group even across different Availablity Zones in a VPC.
- stays in VPC
- Each SG have inbound and outbound rules
- A rule __allows traffic__ to or from a source(IP, network, named AWS entity) and protocol
- __Implicit/Defult deny__ rule but __cannot explicitly deny traffic__
- Apply multiple security groups within the same VPC to an EC2

Each ENI has max __5 SGs__ associated with it.

SGs are __stateful__ -- meaning for any traffic allowed in/out, the return traffic is automatically allowed.

SGs can reference AWS resources, other SGs, and even themselves.

Security groups can be shared across
- Two VPCs in the same region
- Multiple EC2s instances in a VPC
- AWS accounts in the same region


## EC2 Monitoring

__AWS Cloudwatch Logs__ can be used to __monitor and store__ logs from EC2 instances. The instance needs __awslogs log driver__ installed to be able to send logs to CloudWatch. We don't need any database or S3 for storage.

Cloudwatch logs agent is __more efficient__ than AWS SSM Agent.

__Cloudwatch alarm actions__ can automatically start, stop or reboot EC2 instances based on alarms.

__Cloudwatch monitoring schemes__ —
- Basic. 5 minutes.
- Detailed. 1 minute.
- Custom. Can be down to 1 second.

__Default Cloudwatch metrics__ —
- CPU utilization
- Disk reads and writes
- Network in and out

__Custom metrics__ —
- Memory utilization
 - Disk swap utilization
 - Disk space utilization
 - Page file utilization
 - Log collection


## Instance Metadata

Instance metadata is data relaing to the instance that can be accessed from within the instance itself using a utility capable of accessing HTTP and using the URL:

__http://169.254.169.254/latest/meta-data/__

AMI used to create the instance:
    - /meta-data/ami-id

Instance ID:
    - /meta-data/instance-id

Instance type:
    - /meta-data/instance-type

Instance metadata is a way that scripts and applications running on EC2 can get __visibility of data__ they would normally need API calls for.

Metadata can provide:
- External IPv4 address
- Avalability zones
- Security group
- Spot instance terminate time


## AMI (amazon machine images)

AMI is used to create immutable pre-baked(pre-installes instances) AMIs
    - complex application installation
    - static architecture
    - stored in pre-baked AMIs
    - AMI makes it alot Easier and faster to re-launch with existing configurations
    - Auto-scaling with auto-scaling groups can be achived using AMIs

Downside:
    - Bootstrapping(user data while configurations) can not be changed with AMI

AMI can be __public(AWS marketplace)__ or __private__ or __private with AWS account permissions__.

AMI is useful to maintain a state of existing EC2 instances and launch those exact instances when required with zero hassels.

1. Configure Instance
    - source instance and attached EBS volumes are configured with any required software and configurations
2. Create Image
    - Snapshots are created from volumes
    - AMI references snapshots, permissions and block device mappings
3. Launch Instances
    - With appropriate lauch permissions, instances can be created from an AMI
    - EBS volumes are created using snapshots as the source
    - An EC2 instance is created using the block device mapping to reference its new volumes


## Bootstrapping

process of providing "build" directives/command to an EC2 instance as part of __EC2 provisioning process__ like:
    - Configuring an EC2 instance
    - Performing software installation on an EC2
    - Configuring an application on an EC2

It uses user data and can take in __shell script-style commands__ or __cloud-init directives__

__Access keys__ are long term credentials and __shouldn't be used when bootstrapping__.

__AMI bake__ with __bootstrap__ is best way to quickly host multiple aaplications on EC2 instances with different configuration.


## EC2 Networking (ENI, IP, DNS)

Public and Private instances in public and private subnets.

Private DNS of EC2 instance pattern: __ip-X-X-X-X.ec2.internal__

Private IP released on an EBS backed EC2 instance when instance is terminated, Doensn't change when stopped.

Public IPS are dynamic and gets changed when Instance is stopped or terminated.

Elastic IPS can be taken are costly when not attached to an instance. They can be attached to an instance and detached to attach to a different instance. 

## Instance Roles

IAM role for EC2 instances. These are temporary and accessed via STS.

EC2 instance roles are IAM roles that can be "assumed/changed" by EC2 using an intermediary called an instance profile.

An instance profile is either created automatically when using the console UI or manually when using the CLI. It's container for the role that is associated with an EC2 instance.

The instance profile allows applications on the EC2 instance to access the credentials from the role using the instance metadata.

## EC2 Billing

Default billing per second and charged minimum for 60 seconds when provisioned.

1. __On-demand Billing__ for default or un-known demand
    - Anything between Reserved and Spot
    - SHort term workloads that can not tolerate interruptions
2. __Spot Billing__ for Spot-fleet instances
    - Spot instances allow consumption of spare AWS capacity for a given instance type and size in a __specific AZ__
    - Instances are provided for __as long as your bid is above the spot price__, and you only ever pay the spot price
    - If your bid is exceeded, instances are terminated with a __two-minute warning__
    - __Spot fleets__ are a container for "capacity needs". You can specify pools of instances of certain types/sizes aiming for a given "capacity". A minimum percentage of on-demand can be set to ensure the fleet is always active
    - Perfect fit for __non-critical workloads, burst workloads__, or consistent non-critical jobs that can tolerate interruptions without impacting functionality
    - By setting the __maximum price__, you'll ensure that the price of an EC2 never exceeds what you are willing to pay
    - __Not sutable for long-running workloads__ that require stability and cannot tolerate interruptions
3. __Reserved Billing__ (Zonal or Regional)
    - Reserved instance lock in a reduced rate for __one__ or __three years__
    - __Zonal__ reseved instances include a __capacity reservation__
    - Your commitment incurs costs even if instances aren't launched
    - Reserved purchases are used for long-running, critical systems/componenets, understood, and consistent workloads
    - Pay in all __up-front, partially up-front, no up-front__
    - __Regional is flexible but has no capacity reservations__


## Dedicated Hosts

Pick instance type and size. With that, there will be __some limits__ of available number of instances.

DHs are EC2 hosts for a __given type and size__ that cna be dedicated to you.
    - The number of instances that can run on the host is fixed -- depending on the type and size

Not a regional but __AZ specific physical hardware based AWS service__. Keep that in mind. Okay!

When __software is licenced per core/CPU__ and not compatible with running within a __shared cloud environment__.


# EFS

EFS supports cross availability zone mounting, but it is not recommended. The recommended approach is __creating a mount point in each availability zone__.

You can mount an EFS file system in only one VPC at a time. If you want to access it or mount it from another VPC, you have to create a __VPC peering connection__. You should note that all of these must be within the same region.

__NFS port 2049__ for EFS.

__Encryption__

1. Encryption at rest must be specified at the creation of file system. If you want to modify it later on, create a new EFS file system with encryption enabled and copy the data over.
2. Encryption at transit is supported by EFS // NFS, and must be enabled from the client side. It simply uses SSL to encrypt the connection.

__Performance mode__

1. General purpose must be used for most purposes, it has low latency, so ideal for web applications.
2. Max IO is ideal for big data and parallel connection and processing from a large number of hosts. It has higher latency but large throughput.

__Throughput mode__

1. Bursting is ideal for arbitrary large amount of data, because it scales properly.
2. But for cases with high throughput to storage ratio, such as common in web applications, provisioned mode is better.



# Lambda

Lambda functions __can be run within a private VPC__.

Lambda can __read events from__ —
- Amazon Kinesis
- Amazon DynamoDB
- Amazon Simple Queue Service

Services that can __invoke Lambda functions__ —
- Elastic Load Balancing (Application Load Balancer)
- Amazon Cognito
- Amazon Lex
- Amazon Alexa
- Amazon API Gateway
- Amazon CloudFront (Lambda@Edge)
- Amazon Kinesis Data Firehose
- Amazon Simple Storage Service
- Amazon Simple Notification Service
- Amazon Simple Email Service
- AWS CloudFormation
- Amazon CloudWatch Logs
- Amazon CloudWatch Events
- AWS CodeCommit
- AWS Config

AWS __CodePipeline__ and AWS __OpsWorks can't invoke lambda__ functions.

__For failures__ we can configure lambda to send non-processed payloads to __SQS Dead letter queue__. Then we can configure __SNS to send a notification__ if we want. Lambda __does not have an in-built mechanism__ for notification upon failure.

A policy on a role defines which API actions can be made on the target, it does not define whether the source can access the target or not.

Each lambda function has an __ephemeral storage of 512 MB__ in the `tmp` directory.  

AWS __CloudWatch rule__ can be configured to trigger a lambda function. While configuration, the following can be used as __input to the target lambda function__ —
- Matched event
- Part of the matched event
- Constant (JSON text)

The following __CloudFront events can trigger lambda function__ —
- Viewer request
- Viewer response
- Origin request
- Origin response

__Lambda function update has eventual consistency__. Which means, for a brief window of less than a minute, it may execute either the old version or the new version.

We can use __alias versions__ to point to another version. This can enable easier upgradation from the viewpoint of a consumer.

__Limits__ —
- Function memory allocation: 128 MB to 3008 MB, in 64 MB increments.
- Function timeout: 900 seconds.
- Deployment package: 50 MB * 5 layers.
- `tmp` directory storage: 512 MB.

To grant __cross-account permission to a function__, we have to modify the function policy, not the execution role policy.

The console doesn't support directly __modifying permissions in a function policy__. You have to do it from the CLI or SDK.

If we run __lambda functions inside a VPN__, they use __subnet IPs or ENIs__. There should be sufficient ones otherwise it will get throttled.

__ENI capacity__ = Projected peak concurrent executions * (Memory in GB / 3 GB).

The __lambda console__ provides __encryption and decryption helpers__ for encryption of environment variables. 

By default, the a KMS default service key is used for encryption, which makes the information visible to anyone who has access to the lambda console. For further restriction, create a custom KMS key and use that to encrypt.

__CloudWatch metrics for Lambda__ —
- Invocations
- Errors
- Dead Letter Error
- Duration
- Throttles
- IteratorAge
- ConcurrentExecutions
- UnreservedConcurrentExecutions

We can get the __function version__ within the function using —
- `getFunctionVersion` from the Context object.
- `AWS_LAMBDA_FUNCTION_VERSION` environment variable. 

__Lambda Retry upon Failure Behavior__ —
- Event sources that aren't stream-based
    - Synchronous invocation — Returns error with __status code 200__. Includes __FunctionError__ field and __X-Amz-Function-Error__ header.
    - Asynchronous invocation — __Retry twice__, then sent to __Dead Letter Queue__.
- Poll-based and stream-based event source (Kinesis or DynamoDB) — Lambda keeps __retrying until the data expires__. The exception is __blocking__, this ensures the data are processed in order.
- Poll-based but not stream-based event source (SQS) — On unsuccessful processing or if the function times out of the message, it is __returned to the queue__, and ready for further reprocessing after the visibility timeout period. If the function errors out, it is sent to __Dead Letter Queue__.

__Lambda traffic shifting__ —

- Canary
- Linear
- All at once


# Step Functions

A serverless visual workflow service that provides state machines. State machines can be defined using Amazon States Language(ASL).

Up-to one year long. Multiple lambda functions.

Chain of long running tasks in a serverless way.


# API Gateway

API Gateway can __integrate with any HTTP based operations__ available on the public internet, as well as other AWS services.

__Endoint Types__
- Regional
- Edge Optimized
- Private

__Integration types__ —
- Lambda function, can be from __another AWS account__ as well.
- HTTP
- Mock
- AWS Service
- VPC Link

For connecting API Gateway to a set of services hosted in an __on-premise network__, we can use
1. __DirectConnect__ to connect the private network to AWS directly.
2. Then use __VPCLink__ to set up API Gateway connection.

Creation Flow: API -> Resource -> Method

__API Gateway Throttling__ —

- __Burst limit__ refers to the first millisecond.
- __Steady-state limit__ refers to an one second interval.

__Throttling behaviors__ —
- If an user exceeds the burst limit but not the steady-state limit, the rest of the requests are throttled over the one second steady-state interval. 
- If an user exceeds the steady-state limit, AWS returns `429 Too Many Requests` error.

When it comes to throttling settings, you can __override stage settings on an individual method__ within the stage. That is, there is an option for method level throttling to override stage level throttling.

__Access control mechanisms__ for API Gateway —
- Resource policies
- AWS IAM roles and policies
- CORS or Cross-origin resource sharing
- Lambda authorizers
- Amazon Cognito user pools
- Client side SSL certs
- Usage plans

API Gateway __automatically protects the backend systems from DDoS__ attack.

__Cache properties and settings__ —
- Cache status
- Flush entire cache
- Enable API cache
- Cache capacity
- Encrypt cache data
- Cache TTL
- Require authorization
- Handle unauthorized requests

__Monitoring__ API Gateway usage — we can use __CloudWatch__ or __Access logging__. Access logging logs who accessed the API and how the caller accessed the API, CloudWatch does not include this data.

__Protect backend systems__ behind API gateway from __traffic spikes__ —

- Enable throttling.
- Enable result caching.



# ECS

managed container engine. Related services are Amazon EKS (Elastic Kubernetes service) and Amazon ECR (Elastic container registry)

Infrastructure clusers Launch types —

- Fargate (AWS managed)
- EC2 (AWS partually managed)

All types of instances, i.e. __on-demand, spot and reserved can be used with ECS__.

Docker containers and ECS are particularly __suited for batch job workloads__ as they can get embarassingly parallel. 

Amazon ECS enables you to __inject sensitive data into your containers__ by storing your sensitive data in either —
- AWS Secrets Manager secrets
- AWS Systems Manager Parameter Store parameters

ECS main components:
- Scheduling and Orchestration
- Cluster Manager
- Placement Engine

__Cluster__ A logical collection of ECS resources - either ECS EC2 instances or a logical representation of managed Fargate infrastructure

__Tasks Definition__(give image name or URL, Dockerfile) is used to create a ECS task(Dines application) (__Task role__ is required to access resources required for ECS to run given container). Can have multiple containers.

__Container Definitions__ managed inside the tasks definition. Memory, CPU and Post mappings for the containers can be defined.

__Tasks__ single running copy of the task definition, like running image. e.g. DB and Web container

__Service__ Allows task definition to  be scaled by adding additional tasks. Defines Minimum and maximum values.

__Registrey__ Storage for container images i.e. ECS container Registry or DockerHub. used to download image to create  containers.



# Identity Federation

different types of Identity Federation

- Cross-account roles
    - Allows an external AWS identity access to your AWS account by assuming a specifically created IAM Role.
- Web Identity Federation
    - Web Identity Federation allows web identity providers like Google, Amazon, and Facebook to assume roles and access resources in an AWS account.
- SAML 2.0 IDF
    - SAML 2.0 is a way to re-use identities, from on-premises systems (such as Microsoft Active Directory or AWS Directory Service), to assume a role in an AWS account.

Amazon Cognito is not a type of Identity Federation. Amazon Cognito is a service that saves user identity and data synchronously across different devices such as smartphones and tablets.


Placement groups have the placement strategies of Cluster, Partition and Spread. With the Partition placement strategy, instances in one partition do not share the underlying hardware with other partitions. This strategy is suitable for distributed and replicated workloads such as Cassandra.



# STS (Security Token Service)

STS is a web service that enables you to __request temporary, limited-privilege credentials for AWS Identity and Access Management (IAM) users__ or for users that you __authenticate (federated users)__

-  available as a global service (default https://sts.amazonaws.com us-east)
- regional STS is available for low latency, build in redundancy, and increase session token validity
- CloudTrail logging and store to Amazon S3 bucket for later querying request logs(Who, What, When, Status)



# SNS (Simple Notification Service)

provides a __pub/sub-based notification system__, which supports a wide range of __subscriber endpoint__ types. Highly reliable, Can scale

Create Topic, Publishers send messages to Topic and Subscribers receives messages from Topic.

__Available protocols(Subscribers) for AWS SNS__ —
- HTTP(S)
- Email and Email-JSON
- SQS(message can be added to one or more queues)
- Application
- Lambda functions
- SMS (cellular message)
- Mobile push notificationsIiOS, Android, Amazon, MSO

__SNS message attributes__ (Topic)are —
- Name, Type, Value
- Messages size <= 256 KB>
- Subscribers to the topic receive messages

__Publishers__
An entity that publishes/sends messages to queues
- Application
- AWS services, including S3(S3 events), CloudWatch, CloudFormation, etc

We can add __filter policies__ to individual subscribers in an SNS topic.

With __Amazon SNS__, there is a possibility of the client receiving __duplicate messages__.

Encryption at-rest and in-transit available with SNS. Resource Access policy can be applied. Delivery status logging is available.

To access the SNS service from __inside a VPC__
- NAT gateway
- IGW
- VPC endpoint



# SQS (Simple Queue Service)

helps applications scale by allowing decoupling of application components and inter-process, -service, and -server messaging.

Consumers must __delete an SQS message__ manually after it has done processing the message. To delete a message, use the __ReceiptHandle__ of a message, not the MessageId.

Incoming messages can __trigger a lambda function__.

We can use __dead letter queues__ to isolate messages that can't be processed right now.

SQS does not __encrypt__ messages by default.

Default __visibility timeout__ for SQS is __30 seconds__. __Retention period__ is max __4 days__(345600 seconds)

__Standard Queues__ (High throughput, multiple delivery may occur, un-ordered delivery)

Each __FIFO Queue__ uses —
- Message Deduplication ID 
- Message Group ID. Message Group ID helps preserve order.
- 3000 messages per second with batching and 300 by default

For application with identical message bodies, use unique de-duplication ID, while for unique message bodies, use content-based deduplication ID.

Both the default and maximum batch size for `ReceiveMessage` call of SQS is 10.

Reducing SQS API calls —

- Use long polling (opp. to short polling)
- Send `DeleteMessage` requests in batch using `DeleteMessageBatch`. Other batch actions are SendMessageBatch and `ChangeMessageVisibilityBatch`. 

__Message retention period__ in SQS — 1 minute to 14 days. The default is 4 days.

Limit on number of __inflight messages__ — 120,000 for standard queue and 20,000 for FIFO queue.

Resource Policies can be applied and Monitoring is available. CloudWatch can listen and ac upon these monitoring metrics. AutoScaling group can also be enabled to scale-up or in.

For Rapid scaling, use lambda triggers. They are highly available and integrates with SQS efficiently.



# Elastic Transcoder

Elastic Transcoder is a service that performs "serverless" transcoding of media between formats. By default, it works using a manual job submission system but can be expected to operate in an event-driven way. Offers low admin overhead.

Componenets:
- __Pipelines__ (One or more Queues for processing needs)
- __Job__ (gets added to pipiline in the same region, one input object and upto 30 Output objects/formats)
- __Presets__ (contains transcoding settings and can be applied to job to ensure output compatible with various devices(Iphone, ables, TVs, Kindle))

Notifications:
- Pipilnes can send notifications(SNS topics) as jobs progresse through various states(Error, Finished)



# Elastic Beanstalk

AWS __Elastic Beanstalk__(Paas Product) can be used to create —

- Web application using DB
- Capacity provisioning and load balancing of websites
- Long running worker process
- Static website

It should not be used to create tasks which are run once or on a nightly basis, because the infrastructure is provisioned and will be running 24/7.

__Elastic Beanstalk__ can be used to host __Docker containers__.



# Opsworks

Implemetation of the Chef configuration management and deployment platform.

OpsWorks moves away from the low-level configurability of CloudFormation but not as far as Elastic Beanstalk

- __Cookbooks (aka recipes) come from a repository__
- Uses IAM permissions to interact with diff components of AWS

lets you create a stack of resources with layers and manage resources as a unit

__AWS Opsworks__ is a configuration management service for Chef and Puppet. With __Opsworks Stacks__, we can model our application as __a stack containing different layers__.

__OpsWorks Componenets__
- __Stacks__
    - A unit of managed infrastructure
    - can use stacks per application or per platform
    - could use stacks for development, staging, or production
- __Layers__
    - Comparable to application tiers within stack
    - e.g. database layer, application layer, proxy layer
    - Recipes are generally associated with layers and configure what to install on instances in that layer
- __Instances__
    - Instances are EC2 instances associated with a layer
    - Configured as 24/7, load based, or time based
- __Apps__
    - Apps are deplyed to layers from a source code repo or S3
    - Actual deployment happens using recipes on a layer
    - Other recipes are run when deployments happen, potentially to reconfigure oher instances
- __Recipes__
    - __Setup__ Executed on an instance when first provisioned
    - __Configure__ Executed on all instances when instances are added or removed
    - __Deploy__ and Undeploy Executed when apps are added or removed
    - __Shutdown__ Executed when an instance is shut down but before it's stopped

# Storage Gateway

__AWS Storage Gateways__—

1. File gateway
2. Volume gateway: Cached volumes
3. Volume gateway: Stored volumes
4. Tape gateway



# IAM, Cognito and Directory Services

__Amazon Cognito__ has two __authentication methods__, __independent__ of one another —

- Sign in via third party federation
- Cognito user pools

__AWS Directory Service__ options —

- AWS Managed Microsoft AD
- AD Connector
- Simple AD
- Amazon Cloud Directory
- Amazon Cognito

There is no __default policy__ ever, anywhere. When permissions are checked, roles and policies are considered together, and in the default case there is no policy, so only the role is considered.

We can configure __IAM policies__ that allows __access to specific tags__.

__Connecting AWS SSO to On-Premise Active Directory__ —

- __Two-way trust relationship__: __Preferred__. Users can do everything from both portals.
- __AD connector__: SSO does not cache user credentials. Users can't reset password from SSO portal, have to do it from on-premise portal.

For __two-step verification__, SSO sends __code to registered email__. It can set to be either —

- Always-on
- Context-aware

__Cross-account IAM roles__ allow customers to securely grant access to AWS resources in their account to a third party.

If our identity store is not compatible with SAML, we can develop a custom application on-premise and use it with STS.

__Microsoft Active Directory__ supports __SAML__. 


**Monitoring Services**
- CloudWatch (__logging metrics and events, alarms, actions__)
- CloudWatch Logs (__logs ingestion and logs management tool__)
- Cloud Trails (__management and data access logs__)
- VPC Flow Logs (__Network traffic metadata__)


# CloudWatch

Provides near real-time monitoring of AWS products.

Data retention is based on granularity:
- One-hour aggregated metrics - 455 days
- Five-minutes aggregated metrics - 63 days
- One-minute aggregated metrics - 15 days

__Metrics --> alarms --> take actions__ (Show data to dashboard)

__CloudWatch Metrics__ is a set of data points over time. e.g. CPU utilization of an EC2 instance

Alams cna be created on metrics, taking an action if the alarm is triggered.

__Alams states__-
- __INSUFFICIENT__ -- Not eough data o judge the state
- __ALARM__ -- the alarm threshold has been breached (e.g. > 90% CPU)
- __OK__ -- threshold has not been breached

__Alarm components__-
- Metric -- data points over time being measured
- Threshold -- exceeding this is bad (static or anomaly)
- Period -- how long threshold should be bad before an alarm is generated
- Action -- what to do when alarm triggers - SNS, Auto Scaling, EC2 reboot/stop

# CloudWatch Logs

Provides functionality to store, monitor and access logs from EC2, on-premises servers, Lambda, Route53, CloudTrail, VPC Flow Logs, custom applications. 

__Metrics filters__ can be used to analyse logs and creae metrics (failed SSH logins)
    - Metrics filter pattern matches text in all log events in all log streams of whichever log group it's created on, creating a metric 

1 __Log Events__ is a timestamp and a raw message

2 __Log stream__ is a sequence of log events with the same source

3 __Log Group__ is a container for log streams. It contains retention, monitoring, and access control



# CloudTrail

__Enabled by default__ with retention for __90 days__ and stored in Event History.

Provides __full API/account activity logging across all regions__ in an account and (optionally) all accounts within an AWS Organization.

CloudTrail is a __governance, compliance, risk management, and auditing__ service that records account activity within an AWS account.

Any actions taken by users, roles, or AWS services are recorded to the service. Activity is recorded as a __CloudTrail event__, and by default you can __view 90 days via event history__.

__Trails__ can be created, giving more control over logging and allowing __events to be stored in S3 and CloudWatch Logs__.

By default, __CloudTrail logs are encrypted__ using S3 server-side encryption (SSE). We can also choose to encrypt with AWS KMS. SNS notifications for every log file delivery.

Changes to __CloudTrail global service event logs__ can only be done via the CLI or the SDKs, not the console.

__CloudTrail Events__

- __Management events__ insights into the management operations that performed on resources
- __Data Events__ insights into the resource operations performed on or within a resource

CloudTrail can __deliver logs to CloudWatch Logs__ with given __IAM role__.

Cloud Trail is not real-time and only sends data in batches.


# VPC Flow Logs

Manually enabled on a VPC, subnet, or ENI level and __monitor traffic metadata__ for any included interfaces. Flow Logs monitors Network meta-data __(account-id, interface-id, srcaddr, dstaddr, protocol, packets, bytes, start, end, action, log-status)__

- Source and destination IP addresses/ports
- Protocol
- Bytes
- Start and end
- ALLOW or REJECT status

(Every EC2 instances gets a network interface id, and we can find traffic meta-data for that network interface at VPC Flow logs)

__Does not sniff network traffic content__ for network in-out activiies..

VPC Flow Logs __Can be integrated with CloudWatch Logs or S3__ (with IAM role)

Doesn't capture traffic, including Amazon DNS server, Windws licence activation, 169.254.169.254, DHCP traffic, and VPC router.

__VPC Flow Logs aren't real-time__ and don't capture actual traffic -- only metadata on the traffic.


**operations**


# CloudWatch Events

Has a near real-time visibility of changes that happens within an AWS account. Using rules, you can match against certain events within an account and deliver those events to a number of supported targets.  A service that supports the creation of event-driven responses in AWS.

Examples:
- Set scheduled invole lambda function
- On CloudTrain some actions -> invoke lambda function -> Perform some action on CloudTrail again

__Automatically responds to AWS services actions__ can be done by CloudWatch events

__Rules__: Source and Target


# KMS and CloudHSM

AWS key management service for encryptions. __Regional__ secure key management and encrytion/decryption services. (uses base64 encryption)

KMS is __FIPS 140-2 level 2 validated__ and certain aspects support __level 3__.

Everything in KMS is regional, KMS can use __CloudHSM via Custom Key store(FIPS 140-2 level 3)__

- __KMS__ manages Customer Master Key (__CMK__)
- can __encrypt/decrypt and re-encrypt__ up to __4 KB with a CMK__ 
    - supply the data and specify the key to use
    - provide ciphertext, and it returns the plaintext

KMS can generate Data Encryption Key(__DEK__) using CMK.
    - KMS can store encrypted DEK and encrypted data
    - KMS first decrypt the DEK, which can then decrypt the data
    - called __envelope encryption__
    - Plain Data encrypted using -> plainext encryption key(DEK) -> encrypted DEK -> provide back encrypted DEK to get plainext encryption key -> decrypt the data

Customer Master Keys(CMK)
- __AWS managed CMKSs__ (can view, can't manage)
    - used by default when pick encryption within most AWS services
    - formatted as aws/service-name
    - only the service they belong to can use them directly
- __Customer managed CMKs__ (can view, can manage)
    - Certain services allow to pick
    - allowed key rotation configuration
    - can be controlled via key policies and enabled/disabled
- __AWS Owned CMKs__
    - used by AWS on a shared basis across many accounts - you normally don't see these

__CloudHSM(hardware security modules) backup procedure__ — Ephemeral backup key (EBK) is used to encrypt data and Persistent backup key (PBK) is used to encrypt EBK before saving it to an S3 bucket in the same region as that of AWS CloudHSM cluster.

With __AWS CoudHSM__, we can control the entire lifecycle around the keys.

AWS KMS API can be used to encrypt data.

KMS new product called __Custom key store__ like CloudHSM.


**Analytics Services**


# Kinesis

scalable and resilient sreaming service from AWS. Designed to __ingest large amounts of data__ from hundreds, throusands, or even __millions of producers__. Consumers can access a __rolling window__ of that data, or it can be stored in persistent storage of database products.

Stream can be used to collect, process and analyze a large amount of incoming data. A stream is a public service accessible from inside VPCs or from the public interne by an unlimited number of producers.

__Billed__ for shard/per hour. And Every millon payload PUTs(per 25 KB of data).

__Kinesis stream data retention period__ — 24 hours (default) to 168 hours(7 days). 

For __Kinesis__, we have to use __VPC Interface Endpoint__, powered by __AWS PrivateLink__.

Amazon __Kinesis Scaling Utility__ is a __less cost-effective__ solution compared to doing it with __Cloudwatch alarms + API Gateway + Lambda function__.

__Kinesis data streams__ store the data, by default for 24 hours and upto 7 days. Whereas __Kinesis Firehose__ stream the data directly into either —
- S3
- Redshift
- Amazon Elasticsearch Service
- Splunk
- persisntanty store large scale streaming data

__Kinesis Shard__
- Shards are added or removed to/from streams to allow them to scale.
- A stream starts with at least one shard, which allows __1 MiB of ingestion__ and __2 MiB of consumption__.
- If ShardIterator expires immediately and data is lost, we have to increase the write capacity assigned to the Shard table.
- Linear performances with Number of shards and stream capacity.

__Kinesis is very different from SQS__ When needed many producers and many consumers as well as a rolling window of data. SQS is a queue; Kinesis allowss lots of independent consumers reading the same data window.

__Kinesis Data Record__
- Basic entity written to and read from Kinesis streams, a data record can be up o 1 MiB in size.

__Kinesis Streams__ are __deleted based on their rolling retention window__, while SQS messages should be deleted once consumed to avoid re-polling.



# Redshift

Column based database designed for analytical workloads. __OLAP__(Online analytical processing) retrival and analytics.

Petabyte-scale data warehouse product. spot instances are not an option. It's capable of being used for ad-hoc warehouseing/analytics or long-running deployments.

__Redshift automated snapshot retention period__ — 1 day to 35 days.

With __Redshift Spectrum__, we can run complex queries on __data stored in S3__.

We can use __WLM in the parameter group configuration__ of Redshift to define number of query queues and how queries are routed to those queues.

__Cross-region replication__ can be setup for __Redshift Clusters__. automatically copy snapshots (automated or manual) for a cluster to another AWS Region.

Amazon __Redshift Enhanced VPC Routing__ provides VPC resources the access to Redshift.

__Redshift encryption__ —
- Using AWS KMS to encrypt the underlying data.
- Using S3 and its encryption.

Redshift main components __Leader Node, Slices, Compute Node__



# Athena

__Serverless query engine__ capable of reading wide range of data formats from S3.

Uses __Schema on read__ approach to allow SQL-like queries against reletional data.

__AWS Athena__ is a managed service which can be used to make interactive __search queries to S3 data__.

Structured, Un-structured or semi-structured data can be queried by Athena (XML, JSON, CSV/TSV, AVRO, ORC, PARQUET)

AWS __Athena pricing__ is based upon per query and amount of data scanned in each query. To __reduce price__ —
- Partition data based on different parameters so that amount of data scanned gets reduced.
- Create separate workgroups based upon user groups.


# EMR

__AWS EMR__ — AWS Elastic MapReduce, Hadoop based big data analytics.

__AWS EMR__ is preferred for __processing log files__.

__EMR__ can use __spot instances__ as underlying nodes.

We can access the underlying EC2 instances in AWS EMR cluster. It is an __AWS-managed__, EC2 cluster-based product.

Main componets are __Master Nodes, Core nodes, Task Nodes, S3 or HDFS__


# Misc

An 80 TB __Snowball__ appliance and 100 TB Snowball Edge appliance only have 72 TB and 83 TB of __usable capacity__ respectively. 

AWS STS — The __policy of the temporary credentials__ generated by STS are defined by the intersection of your IAM user policies and the policy that you pass as argument.

AWS __VM Import__ // Export can be used to transfer virtual machines from local infrastructure to AWS and vice-versa.

AWS __Trusted Advisor__ is a resource that helps users with cost management, performance and security.

__CloudFormation Drift Detection__ can be used to detect changes in the environment. Drift Detection only __checks property values which are explicitly set__ by stack templates or by specifying template parameters. It does not determine drift for property values which are set by default.

__AWS Server Migration Service (SMS)__ is an agentless service which makes it easier and faster for you to migrate thousands of on-premise workloads to AWS.

__Amazon Inspector__ is a security assessment service, which helps improve security and compliance of applications.

__Amazon ECS for Kubernetes (EKS)__ exists, it's a managed service.

__AWS Polly__ — Lexicons are specific to a region. For a single text appearing multiple times, we can create alias using multiple Lexicons.

Amazon __Quicksight__ is a managed service for __creating dashboards__ with data visualization.

__AWS CloudSearch__ helps us add search to our website or application. __Like Elasticsearch__.

__AWS Glue__ is a fully __managed ETL service__ for data. It __keeps a track of processed data using Job Bookmark__. Enabling Job Bookmark will help to __scan only changes since last bookmark__ and prevent processing of whole data again.

__AWS X-Ray__ — Helps debug and __analyze microservices architecture__.

__Reducing cost with AWS X-Ray__ — Sampling at a lower rate.

__Amazon WorkDocs__ has a __poweruser__ facility, which on enabling restricts sharing of documents to that user only.

__AWS Data Pipeline__ can automate the movement and transformation of data for data-driven workflows. For example, transferring older data to S3 from DynamoDB.


__Disaster recovery solutions__ —
- Backup and Restore. Cheapest.
- Pilot Light
- Warm Standby
- Multi-Site
- Multiple AWS Regions. Costliest.

With __AWS Config__, we can get a snapshot of the current configuration of our AWS account.

For __queue based processing__, scaling EC2 instances based on the size of the queue is a preferred architecture.

It's best practice to launch Amazon __RDS instance outside an Elastic Beanstalk environment__.

__AWS Athena is simpler__ and requires less effort to set up __than AWS Quicksight__.

__RI Coverage Budget__ reports number of instances that are part of Reserved Instance. For an organisation using default IAM policy, each member account owner needs to create a budget policy for individual accounts and not by master account.

__Consolidated Billing__ in AWS Organisations combines usage from all accounts and billing is generated based upon total usage. Services like __EC2 and S3 have volume pricing tiers__ where with more usage volume the overall charge decreases.

To automatically trigger __CodePipeline__ with changes in source __S3__ bucket, use __CloudWatch Events rule__ and __CloudTrail trail__.

__Amazon Data Lifecycle Manager__ can be used for creation, retention and deletion of EBS snapshots.

With __AWS Organizations__, we can centrally manage policies across multiple AWS accounts. With __Service Control Policies (SCPs)__, we can ensure security policies are in place.

__AWS WAF__ is a web application firewall.

In __AWS Managed Blockchain network__, the format for __resource endpoint__ is — `ResourceID.MemberID.NetworkID.managedblockchain.us-east-1.amazonaws.com:PortNumber`.

When you want to keep your expenditure within a budget, use __AWS Budgets__, not AWS Cost Explorer.

__Transferring data__ from an EC2 instance to Amazon S3, Amazon Glacier, Amazon DynamoDB, Amazon SES, Amazon SQS, or Amazon SimpleDB __in the same AWS Region has no cost at all__.

__Amazon MQ__ is a message queue which supports industry standard messaging protocols.

__AWS IoT Core__ is a managed service that lets IoT devices connect and interact with AWS applications and resources.

The following storage have __encryption at rest by default__ —

- AWS Glacier
- Storage Gateway in S3

By default, each workflow execution can run for a __maximum of 1 year__ in Amazon SWF. 

In __AWS SWF__, a __decision task__ tells the decider the state of the workflow execution.

__Third party SSL cert__ can be imported into —

- AWS Certificate Manager
- IAM Certificate Store
