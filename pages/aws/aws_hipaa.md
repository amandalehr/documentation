---
title: AWS HIPAA
keywords: aws, hipaa, procedures
last_updated: January 25, 2017
tags: [AWS, HIPAA, medicine, case_studies, storage, research_computing, data_science]
summary: "A HIPAA-compliant research system on AWS"
sidebar: mydoc_sidebar
permalink: aws_hipaa.html
folder: aws
---

## Introduction

[This document](aws_hipaa.html) presents a secure data management system, specifically using the 
motivation of managing Private Health Information (PHI) under HIPAA regulations on the public cloud. 
This template applies more generally to secure data management in research computing.

HIPAA regulations require data be encrypted both at rest (e.g. on a storage device) and in 
transit (e.g. moving from one storage device to another).  Our hypothetical situation is described 
in further detail below; it is well to keep these rest/transit encryption notions in mind. 

Please note that while this study uses AWS we are also building out the construct on the 
Microsoft Azure public cloud. 

## Links

- [HIPAA home page](https://www.hhs.gov/hipaa/index.html/)
- [HIPAA on Wikipedia](https://en.wikipedia.org/wiki/Health_Insurance_Portability_and_Accountability_Act)
- [HIPAA Privacy Rule Summary](https://www.hhs.gov/hipaa/for-professionals/privacy/laws-regulations/index.html)
- [AWS HIPAA template](https://aws.amazon.com/quickstart/architecture/accelerator-hipaa/)


## Warnings

- ***AWS has nine HIPAA-aligned technologies. A HIPAA-compliant system on AWS means that only these technologies can 
come into contact with PHI.  Other technologies can be used in an ancillary capacity provided they do not interact
directly with PHI data.***
- ***HIPAA compliance is an obligation placed on the data system builder and the medical researcher.
The public cloud is very secure: physically and technologically. 
We contend that data system failure or compromise is most likely to be caused by human error.***
- ***File names may not include PHI. They may include identifier strings that would be indexed in a secure table.***

## User story

- A scientist *K* receives approval from an IRB to work with PHI data. 
  - The intent is to analyze these data in a secure, HIPAA-compliant data system (herein HCDS) 
- *K* contacts a technology expert *J* and a research data provider *S* 
  - Presenting a request for the approved data and a request for a cloud-based HCDS 
- Some due diligence and preparation ensue
- *K* is given an ssh key to her HCDS
  - *K* logs on to the HCDS through a secure gateway 
  - *K* carries out the data analysis over a period of time
    - The system logs all activity 
  - IOT devices distributed to patients report health data to the HCDS 
    - These data supplement the research
- The study concludes 
  - Sensitive data are stored to a secure archive 
  - The HCDS is deleted 
  - Log files are archived to preserve a complete record of operation for the HCDS

## Plan of action for this HCDS Proof of Concept

0. Follow the existing notes/guidelines and translate into this document
1. Create a system architecture and diagram  
  - Anticipate <new data to EMR> pipeline
  - Anticipate <IOT to VM> pipeline
  - Anticipate changes to <PHI > VM> process
2. Create artificial data 
  - data manufacturing software 
  - generate source artificial data (from EMR)
  - generate IOT stream
  - anticipate study-to-clinical pipeline
3. Complete system including analytical tools
  - include R, Python, Jupyter 
4. Review with IT personnel (J first)
5. Review with management
6. Review with researchers

## Details to incorporate

- Logging: CloudWatch and CloudXXXXX are AWS logging services; and this is frequently parsed using Splunk
- Intrusion detection! Jon Skelton (Berkeley AWS Working Group) reviewed use of Siricata (mentions 'Snort' also) 
- Include an encryption path for importing clinical data 
- Include a full story on access key management
- The IOT import will -- I think -- be a poll action: The secure VM is polling for new data
- This system should include a very explicit writeup of how the human in the loop can break the system
- Acceptable for data on an encrypted drive to moves through an encrypted link to another encrypted drive? 
To clarify: Must the data be further encrypted at rest first? 
- Filenames may not include PHI. Hence there is an obligation on the MRs to follow this and/or build it into file generation.
- CISO approval hinges on IT, Admin and Research approvals. 

## Terminology and Concepts
* Ansible
* Regions and Availability Zones on AWS
* Bastion Server 
* Siricata / Snort
* CIDR
* Direct tools like **ssh** and third party apps like Cloudberry 
* Dedicated Instance 
* Lambda Service
* NAT Gateway

## PHI contact model

We identify tools and technologies that do / do not interface directly with PHI.

- Tools that do not come in contact with PHI can be thought of as 'triggers and orchestration'.
- Services that may come in contact with PHI can be described as 'data and compute'

### HIPAA-aligned technologies

Nine AWS Technologies under the AWS BAA that are HIPAA-aligned:

- S3 storage
- EC2 compute instances (VMs)
- EBS elastic block storage: Attached filesystem
- RDS relational database service
- DynamoDB database
- EMR elastic map reduce: Hadoop/Spark engine support
- ELB elastic load balancer
- Glacier archival storage
- Redshift data warehouse

### Useful Trigger/Orchestration Tools

- Lambda
- Virtual Private Cloud (VPC)

### Encryption

- HIPAA requires data be encrypted at rest, i.e. on a storage device
- HIPAA requires data be encrypted in transit, e.g. moving from one storage device to another

## Architecture

- Placeholder
- Diagrams

### Multiple sources of data

- Electronic Medical Records (EMR)
- A Data Warehouse (tables) 
- A Clinician 
- Sensors stream data from patient

### Destinations
- S3 bucket
- EBS/EFS 

### Configuring the system

#### Overview

- Initial steps: Build out the VPC and demonstrate dropping a file into S3 from the outside
and then manually going inside the VPC to pull that file through onto a private EC2 instance.
- Next steps: Around services, encryption, ...
- Logging
- Pseudo Data
- Transition to operation
- VPC from Template
- Bastion from AMI 
- EC2 Master from AMI
- EC2 Workers from AMI

We punctuate the procedural steps needed to build the HIPAA-Compliant Data System (HCDS) 
with short notes on rationale, how things fit together. We also make extensive use of 
very short abbreviations (just about every entity gets one, indicated by **boldface** 
and obsessive re-naming of everything using the Project Identifier Tag (PIT)

#### Initial steps

- Absolute First Priority Step 0: Designate to AWS that this account involves PHI/HIPAA data
  - kilroy action to Aaron: How to do find this out?!

- Keep in mind: Suppose this is a multi-day effort: Shut down instances to save money

Our objective is (see Figure below kilroy) to use a Laptop or other cloud-external data source
to feed data into a HIPAA-Compliant Data System (**HCDS**) wherein we operate on that data. The 
data are assumed to be Private Health Information (PHI).

- Write down or obtain a Project Identifier Tag (PIT) to use in naming/tagging everything
  - In our example here PIT = 'hipaa'. Short, easy to read = better

- Identify our source computer as **L**, a Laptop sitting in a coffee shop 
  - This is intentionally a *non-secure* source
  - We return to this issue later (kilroy make sure we do please)
  - **L** does *not* have a PIT
  - **L** could also be a secure resource operated by Med Research / IRB / data warehouse

We are starting with a data source and will return to encryption later. The first burst 
of activity will be the creation of a Virtual Private Cloud (VPC) on AWS per the diagram
above (kilroy). We assume you have done Step 0 above and are acting in the capacity of a
system builder; but you may not be an experienced IT professional. You are building this 
environment, you may or may not necessarily be doing research.

The easiest way to create a VPC is using the console Wizard. That method is covered in a 
section below.  The method we use here is manual to help illuminate the components.

#### Diagram notes

- AWS is the big box
- VPC is inside
- Public and Private subnets are inside VPC
- NAT gateway inside the Public subnet box
- Internet Gateway on boundary of VPC
- S3 Endpoint on boundary of VPC

#### To continue on first steps

CIDR block syntax: The specification 10.0.0.0/16 has two parts: w.x.y.z and /N. 
The first part defines an ip address with three wildcards, the zeros.  N indicates 
number of addresses available: 2**(32 - N). When N = 16 this is 2**16 or 65536. 
This is two bytes so y and z are usable to cover.  Hence 10.0.0.0/16 means 
"addresses of the form 10.0.y.z". Any subnets we place within the VPC will be limited 
by this address space.  We will define two subnets within the VPC each with respective 
CIDR ranges: Subranges of the VPC CIDR block.  A subnet with CIDR = 10.0.1.0/24 has 256 
addresses of the form 10.0.1.z.  From each subnet AWS grabs a few ip addresses: .0, .1, 
.2, .3, .255; so on a /24 only 251 addresses remain; for example 10.0.1.4, 10.0.1.5,
10.0.1.6, ..., 10.0.1.254.  

- On AWS create a VPC **V**
  - Give **V** a PIT name 
  - **V** will not use IPv6v.  This makes matters simpler.
  - **V** will have a CIDR block defining an ip address space
    - We use 10.0.0.0/16
  - **V** automatically has a routing table **RT**
    - After creating **V**: Select Routing Tables, sort by VPC and give **RT** a PIT name
      - Here: 'hipaa_routingtable'
      - The routing table is a logical mapping of ip addresses to destinations
  - **V** automatically has a security group **SG**
    - After creating **V**: Select Security Groups, sort by VPC and give **SG** a PIT name
      - Here: 'hipaa_securitygroup'
  - **V** will have both public and private subnets 
    - The private subnet **Si** is where work on PHI proceeds
      - CIDR block 10.0.1.0/24
      - **Si** will be firewalled behind a NAT gateway
    - The public subnet **Su** is for internet access
      - CIDR block 10.0.0.0/24
      - **Su** will connect with the internet via an Internet Gateway
      - **Su** will be home to the Bastion server **B** 
      - **Su** will be home to the NAT Gateway **NG**
        - **B** and **NG** are on the public subnet but also have private subnet ip addresses 
          - That is: Everthing on the public subnet also has a private ip address in the VPC. 
          - This will obviously chew up the private ip address range capacity
          - Public names will resolve to private addresses within the VPS at need.
          - kilroy need a remark on how this works, collision avoidance
  - After creating **V** create an associated Flow Log **FL**
    - As of March 2017 the UI is a little tetchy so be prepared to go around twice
    - Click the Create Flow Log button
      - Assuming permissions are not set: Note the Hypertext to set up permissions 
        - This is because we need to define the proper Role
        - So click on this hypertext
        - On the Role creation page: Give the Role a PIT name; Create new; Allow
        - You now have an IAM Role for FlowLogs
          - This gives the account the necessary AWS permissions to work with Flow Logs
          - In so doing we fell out of the Create Flow Log dialog so... back around
      - Return to the VPC in the console
      - Click on Create Flow Log
        - Filter = All is required (not "accepted" and not "rejected" traffic)
        - Role: Select the role we just created above
        - Destination log group: Give it a PIT name 
          - We use hipaa_loggroup


![kilroy figures]()

- On **V** create subnets **Su** and **Si**
  - Note that Create VPC does give us the option of creating a VPC with a pre-built pub/priv... 
    - See below for more on this faster template-driven approach

Note: The console column for subnets shows "Auto-assign Public IP" and this should be Yes for our 
public subnet (note column name includes *Public IP*). The Private subnet should have this set
to No. If necessary change these entries using the "Subnet Actions" button. 

Note: We must be able to make new EC2 instances on the private subnet that *by default* do not have
a public IP address. In this way we do not accidentally expose a **Wi** instance (worker) to public
access; so this tremendously important. kilroy need a link to where we come back to this and solve.
In the subnet table there is a "Default Subnet" column which is a boolean. In this example both
**Su** and **Si**  have this set to No so there is no default subnet. We fix this later.
The fix happens in the routing table **RT** which is built into the VPC.

- Give **RT** a PIT-name hipaa_routingtable
  - This is a global VPC-wide routing table; it can be superseded by a subnet's routing table

The VPC routing table **RT** reads:  
```
10.0.0.0/16 points to the VPC "local" stuf
0.0.0.0/0 points to the NAT gateway and to the Internet
```

**Su** has a routing table (PIT name is hipaa_public_routingtable)  which has 
10.0.0.0/16 
0.0.0.0/0 Internet gateway

Before creating more RTs we need an Internet Gateway IG
This is easily done and also attach hipaa_internetgateway to hipaa the VPC


so Create Route Table: hipaa_publicsubnet on the hipaa VPC
Go to Subnet Associations tab for this RT
edit this to do subnet association with the public subnet. Obviously. So RT > Subnet > VPC
  call this hipaa_publicroutes 
Go to Routes tab for this RT
  For this RT Edit (under Routes) and add 0.0.0.0/0 pointing to the IG

Now add the NAT Gateway; and then we will modify the main RT to point to this
  - There may be some Elastic IP assignment voodoo 

WE now have two RTs. VPC and Public. VPC 0-entry points at the NAT Gateway: All internet-traffic
will route through the NAT. 

Public subnet has a custom RT which says "all non-local traffic goes out to the IG." = The Internet. 

This has precedence over the main table which sends to NAT. 

By default the Private subnet will use the VPC RT to go to the NAT. Like one's Router at home. 
This allows the private network to talk to the Internet and the Internet can't talk to my private subnet. 




  


- Create an Internet Gateway **IG**

- Create a NAT Gateway **NG**

- On **V** create a public-facing Bastion Server **B**
  - **B** has only port 22 open (ssh) 
  - **B** uses Secure Groups on AWS to limit access to only a subset of URLs
  - kilroy establish that a second key **K2** provides secure access to **B**
  - kilroy establish the chain of custody of **K2**
  - might **B** be from an AMI?


m4.large running AWS Linux: Created. DO NOT USE T instances because they are not going to connect to our 
Dedicated Tenancy VPC: Not supported.

Go through all the config steps: Memory, tags, etc etc; Security group is important

Create a new security group with title hipaa_bastion_ssh_securitygroup
Description = allow ssh from anywhere
Notice in the config table "Source" is 0.0.0.0/0 which is "anywhere"; but best practice is to restrict...

IF we allowed only UW but included the UW-VPN then someone could log in from anywhere VIA the UW VPN... 


Key pair: hipaa_bastion_keypair: Generate new, download to someplace safe on **L** and Launch

- Assign **B** an Elastic IP address that will persist, assumed public
  - kilroy we did not do this but it needs doing; so must free one up from account


**Su** is subnet public **Si** is subnet private


- On **Si** install a small Dedicated EC2 instance **E**
  - kilroy establish that **E** has an access key **K3** 
  - kilroy follow the chain of custody of **K3** to **B**
  - kilroy might **E** be from an AMI?


We are now configuring the S3 Endpoint. 
  In so doing we selected the VPC RT (not the public subnet RT; could use that)
  We are concerned about S3 traffic
  S3 Endpoint is a new type of Gateway that gives S3 access from Private subnet direct into S3 without any traverse of the internet.
  

WARNING: Kilroy: After I added the S3 Endpoint the NAT gateway entry had vanished from my VPC routing table. 
This is bad. Right now the procedure is going to be: After S3 EP go examine RT and re-add NAT gateway if it 
is not properly present. holy cow!!!!!!!!!!!!!!!


Created S3 bucket

## Encryption
Suppose on EC2 we create an EBS volume for the PHI
So is the "comes with" EBS volume encrypted? No. Therefore: Keep data on /hipaa
Volume type = General Pupose and the size does affect IOPS; I went for 64GB

For costing and performance include this link: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSVolumeTypes.html

This gets back into optimization; it does not affect our HIPAA story 

Notice the EBS volume has an AZ which **must match** the target EC2 where it is going 

Notice the volume can by encrypted with a check box; and then there is a Master Key issue. 

kilroy use the default which has some Key Details listed (can these be shown publicly???) or...

Using the default key technically encrypts this data at rest. So that's done. 

Working with your own set of keys would be part of risk management: Should the keys be compromised etcetera.
So there is some additional hassle and so on but some potential risk management. 
Due diligence: kilroy check the DLT account T&C; and generally "who is responsible for the risk?" 

The question is: Do we want to use one key across multiple environments (the default) or created new keys for 
each new envvironment? How much hassle, etc.

ok done: hipaa_ebs_ec2_private 

Next log in to EC2 on the private subnet:
- log in to the bastion server
- used copy-paste to edit a file on the bastion called ec2_private_keypair.pem
  - I did a bad paste and was missing the -----BEGI
  - on this file we ran chmod 600 on the pem file
  - ssh -i <this file> 10.0.1.248
    - notice this ip address is in the console Description tab when highlighting the private subnet EC2 instance
  - Notice that the pem file on the bastion would immediately compromise the EC2 on the private subnet
    - That sure sucks for you

Using (sudo) fdisk to create the partition on the raw block device (fast: writing the partition table) will blow 
away anything already there. 

Now we did the simple

% aws s3 cp s3://bucket-name/keyname .

The keyname that I used was the filename that I uploaded to this S3 bucket from **L**. 
That file was pushed from **L** using the console but can also be done using the CLI. 

We are intentionally not going to encrypt the boot volume. It can be done; goes on the DD pile. 
  - related: kilroy action: How to think about swap space; as part of encryption; 

We implement server side encryption on S3 next. 
- The file may be unencrypted on **L**
- We upload it to S3 and stipulate "encrypt this when it comes to rest in S3"
- S3 manages this
- We create an associated policy that *only* allows this type of upload
  - Therefore a not-encrypted-at-rest request will be denied

Best way is follow "S3 AWS encryption" links to http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingEncryption.html
Click Server-Side Encryption
Click on Amazon S3-Managed Encryption keys in the left bar to get to 
http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingServerSideEncryption.html

Copy the code box contents
Go to console
S3 bucket
Permissions tab
Bucket Policy button
Paste
Replace in two places: actual bucket name

PutObject command: must have server-side-encryption set to AES256 (that is the name of the encryption algorithm) 
  AND
must have server-side-encryption set to true

What about the inbound files: They must be "encrypted in transit" so we get that with an https endpoint to S3: Done. 

Transferring to the instance: scp 

Last encryption note: SSL is used by the CLI be default; so our EC2-private command 'aws s3 ... etc': Look at cli/latest/reference for the link on this. This means that S3 to EC2private is encrypted in transit. The EBS /hipaa is encrypted at rest. Done. 



 



## Auditing

DR
Indicate awareness; up to CISO to provide hurdles






- Move **K** to **B**
  - kilroy how? Using scp / ssh? 
- Log in to **B** and move **K** to **E** 
  - Observe that material encrypted 
  - Maybe instead we should be tunneling directly to **E** from **L**
- Use NAT Gateway to install encryption s/w **ENC** on **E**
  - Or should this already exist kilroy via AMI?
- Use an AMI to create a processing EC2 **W**

  - Also with **ENC**
- Create S3 buckets 
  - **S3D** for data
  - **S3O** for output
  - **S3L** for logging
  - **S3A** for ancillary purposes (non-PHI is the intention)

  - Such S3 buckets only accept http PUT; not GET or LIST
- Create an S3 bucke **S3O** for output
- Create an S3 bucke **S3L** for logging
- Create an S3 Endpoint **S3EP_D** in **V**
- Create an S3 Endpoint **S3EP_O** in **V**
- Create an S3 Endpoint **S3EP_L** in **V**
  - "S3 buckets have a VPC Endpoint included... ensure this terminates inside the VPC"
- Create role **R** allowing **W** to read data at **S3EP** from **S3**

#### Advanced steps

- Set up a DynamoDB table to track names of uploaded files
- Set up a Lambda service 
  - Triggered by new object in bucket in the S3 input bucket
  - This Lambda service is managed using a role
- Set up an SQS Simple Queue Service **Q** (kilroy... ?)
- Create an SNS to notify me when interesting things happen


#### Logging 

- Configure everything above to log to **S3L** using CloudWatch/CloudFront

#### Creating Pseudo Data

#### Transition to operation

#### Residual notes

- Set up Ansible-assisted process for configuring and running jobs on EC2 instances
  - kilroy what is Ansible?
- Pushing data to S3
  - Console does not seem like a good mechanism
  - Third party apps such as Cloudberry are possible...
  - AWS CLI with scripts: Probably the most direct method
- Compute scale test: Involves setting up some substantial processing power
  - Implication is that the HCDS can intrinsically fire up EC2 instances as needed
  - Launch **W** x 5 Dedicated instances, call these **Wi**
  - Assign S3 access role 
  - Encrypted volumes 
  - S/w pre-installed (e.g. genomics pipelines)
  - Update issue: Pipeline changes, etcetera; kilroy NAT Gateway to pull updates from GitHub? 
- **Wi** can be pre-populated with reference data: Sheena Todhunter operational scenario
  - Assumes that a HCDS exists in perpetuity to perform some perfunctory pipeline processing
  - On **B**
    - Create SQS queue of objects in S3
    - Start a **Wi** for each message in queue...
  - Go
    - Latest pipeline... EBS Genomes... chew
    - If last instance running: Consolidate / clean-up
  - SNS topic notifies me when last instance shuts down.
    - What is the difference between SNS and SQS? kilroy
    - Run Ansible script to configure **Wi** (patch, get data file names from DynamoDB table, etcetera)
    - Get Ws the Key from E
    - The Ws send an Alert through the NAT gateway to Simple Notification Service (SNS) 
    - Which uses something called SES to send an email to the effect that the system is working with PHI data 
      - Ws pull data from S3 using VPC Endpoint; thanks to the Route table
      - Ws decrypt data using HomerKey
      - Ws process their data into result files: Encrypted EBS volume. 
      - Optionally the result files are encrypted in place in the EBS volume.
  - Through **S3EP_O** the results are moved to S3.
  - **Wi** sends an Alert through the NAT gateway to SNS 
    - which uses something called SES to send another email: Done
  - **Wi** evaporates completely leaving no trace

## Procedure Log

Create a VPC **V**

![pic0001](/documentation/images/aws/hipaa0001.png)

![hipaa0002](/documentation/images/aws/hipaa0002.png)

- Use the name 'hipaa'; should be unique.
- CIDR as shown is typical.
- Dedicated Instance means: Nobody else allowed here. 

![hipaa0003](/documentation/images/aws/hipaa0003.png)

- I added a tag indicating that I originated the VPC.
- I do not yet have a Flow Log (kilroy) associated with the VPC

### Pro Tip
You can be more cost-effective by not making this Dedicated but then your PHI-Using instances will have to be launched Dedicated. 
This is carte blanche Dedicated and so is more expensive. We do not consider this option in this tutorial because we are erring on 
the side of caution rather than cost. 


### Create a subnet

![hipaa0004](/documentation/images/aws/hipaa0004.png)

![hipaa0005](/documentation/images/aws/hipaa0005.png)

Public subnet addresses will be of the form: 10.0.0.2, .3, .4, ... .254

![hipaa0006](/documentation/images/aws/hipaa0006.png)

Take note of the AZ: 

![hipaa0007](/documentation/images/aws/hipaa0007.png)

We could do multiple public sub-nets by creating more than one in multiple Azs; that is a big-time concept.

Now the private one: 

![hipaa0007](/documentation/images/aws/hipaa0007.png)

![hipaa0008](/documentation/images/aws/hipaa0008.png)

![hipaa0009](/documentation/images/aws/hipaa0009.png)

![hipaa0010](/documentation/images/aws/hipaa0010.png)

Attach it to the VPC: 

![hipaa0011](/documentation/images/aws/hipaa0011.png)

![hipaa0012](/documentation/images/aws/hipaa0012.png)

Now for the NAT Gateway

![hipaa0013](/documentation/images/aws/hipaa0013.png)

Choose the public one in czarhipaa

![hipaa0014](/documentation/images/aws/hipaa0014.png)

![hipaa0015](/documentation/images/aws/hipaa0015.png)

Notice we Created a New EIP

![hipaa0016](/documentation/images/aws/hipaa0016.png)

Now the Route Table

![hipaa0017](/documentation/images/aws/hipaa0017.png)

![hipaa0018](/documentation/images/aws/hipaa0018.png)

![hipaa0019](/documentation/images/aws/hipaa0019.png)

Here they are (including the default main one): 

![hipaa0020](/documentation/images/aws/hipaa0020.png)

![hipaa0021](/documentation/images/aws/hipaa0021.png)

Edit and modify as shown:

![hipaa0022](/documentation/images/aws/hipaa0022.png)

Save

And then under Subnet Associations tab: Edit: 

![hipaa0023](/documentation/images/aws/hipaa0023.png)

Now let us go back to the Route Table selector

![hipaa0024](/documentation/images/aws/hipaa0024.png)

![hipaa0025](/documentation/images/aws/hipaa0025.png)

![hipaa0026](/documentation/images/aws/hipaa0026.png)

Subnet Associations tab: 

![hipaa0027](/documentation/images/aws/hipaa0027.png)

![hipaa0028](/documentation/images/aws/hipaa0028.png)

Now for the Endpoint

![hipaa0029](/documentation/images/aws/hipaa0029.png)

Notice that this has Full Access; we will restrict access at a later step.

![hipaa0030](/documentation/images/aws/hipaa0030.png)

end as of Jan 27 2017.

{% include links.html %}
