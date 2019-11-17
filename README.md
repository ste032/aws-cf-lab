# Disclaimer

This repository is my attemp at trying to come up with a solution for the CMS-kata (https://github.com/xpeppers/cms-kata) by Xpeppers 
using my current expertise level on AWS and Linux Administration.

I am going to be completely honest: it was brutal yet fun, and it was worth doing if only for the incredible amount of knowledge and opportunities to learn this kata put me through. 

The reason why I had to split it into two templates is quite simple: I couldn't make autoscaling and elb to work, so I had to fall down to a more simple solution that implemented a single EC2 instance + RDB and served on http.

I realize the solution offered doesn't comply with the kata requirements, and for that I apologize. I poured myself into it nonetheless and tried to tackle this new challenge at the best of my abilities. I learned a lot of new concepts and techniques, also got to take a peek of what Infrastructure as code really means and what it has to offer. 

For this I thank you, regardless of the outcome.

<br>
  
# Technicalities

Setting up and making centos7 work with cloudformation init was a challenge in itself. SElinux didn't cooperate either, but managed to not disabled it since I am a grown adult. The comments in the templates go in detail on what approach I applied in coding them.

The AWS cloudformation documentation has a set of very specific Joomla examples, almost too specific. My work is based on these templates, and much more work was done adapting the concepts to centos rather than amazon linux (the distro used in the examples). 

Also, json vs YAML: what a match! YAML every day.


# Requirements

Both templates require the following:

1. **AMI:** the official CentOS 7 ami provided by the CENTOS Foundation, which is only available after subscribing to it:
    https://aws.amazon.com/marketplace/pp/B00O7WM7QW#pdp-overview ;
    
2. **VPC:** an existing VPC;

3. **Subnets:** one or more subnets already configured in your vpc;

4. **Key Pair:** an existing key pair used for ssh access;

<br>


# Templates dive-in

There are a number of parameters used in both templates, some of which need to be manually set at runtime.

### Parameters used in both templates:


| Name                      | Type                     | Description   | Default Value         |  Possible Values |
| ------------------------- |:------------------------:|--------------|----------------------:|-----------|
| VpcId                     | AWS::EC2::VPC::Id        | ID of anexisting VPC |   ||
| Subnets                   | AWS::EC2::Subnet::Id     | ID(s) of one or more subnets |  ||
| KeyName | AWS::EC2::KeyPair::KeyName | Existing EC2 KeyPair to enable SSH |  ||
| InstanceType | String | EC2 instance(s) class | t2.micro ||
| JoomlaUsername |  String | Joomla admin username | admin||
| JoomlaPassword |  String | Joomla admin password  | adminjoomla ||
| DBName |  String | RDS Mysql instance name  | joomladb ||
| DBUsername |  String | Mysql account username  | admin ||
| DBPassword |  String | Mysql account password  | password ||
| DBClass |  String | RDS Mysql class  | db.t2.micro ||
| DBAllocatedStorage |  Number | RDS Mysql initial size in GB  | 5 to 8 ||
| SSHLocation |  String | IP addr range for ssh  | 0.0.0.0/0 ||
| JoomlaVersion |  String | Joomla version to be installed   | 3-9-13 | 3-9-13, 3-9-12, 3-9-11, 3-9-10, 3-9-9|

<br>

### Template specific parameters for "multi.yaml" :

| Name                      | Type                     | Description   | Default Value         |  Possible Values |
| ------------------------- |:------------------------:|--------------|----------------------:|-----------|
| WebServerCapacity        | Number      | # of EC2 instances to deploy |  2 | 1 to 3|

<br>

# Deploy

Validate the template via AWS CLI:

```
aws cloudformation validate-template --template-body file:///path/to/single.yaml

aws cloudformation validate-template --template-body file:///path/to/multi.yaml

```

then deploy specifying only the parameters that don't have a default value, which coincidentally are the same for both templates:

```
aws cloudformation create-stack --template-body file:///path/to/any_of_the_templates.yaml --stack-name YOUR_STACK_NAME --parameters ParameterKey=KeyName,ParameterValue=YOUR_VALUE ParameterKey=Subnets,ParameterValue=YOUR_VALUE ParameterKey=VpcId,ParameterValue=YOUR_VALUE

```

