# AWS-CodeDeploy-Demo

Demo tutorial using AWS CodeDeploy via GitHub. Two types of stacks are provided:

1. An Amazon Linux nested stack (low flexibility)
2. An Ubuntu Server stack (high flexibility, but also lengthy configuration)

Consult the corresponding section to deploy one of the stacks.

# 1. Amazon Linux Stack (NOT RECOMMENDED)

This stack is located in [`al-stack.yaml`](./al-stack.yaml) and features a nested stack provided by AWS.

The CF stack internally wraps an AWS-provided CF stack for provisioning an AWS EC2 Amazon Linux instance with the
CodeDeploy agent installed. For more details
see [here](https://docs.aws.amazon.com/codedeploy/latest/userguide/instances-ec2-create-cloudformation-template.html#instances-ec2-create-cloudformation-template-cli).

It also creates necessary IAM resources, such as:

- a Service Role that allows CodeDeploy to access EC2;
- an [EC2 instance profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)
that allows our EC2 instance to access S3 to obtain our application bundle

## 1.1. Push the AL stack to AWS

Example command:

Ensure that you pass in a valid EC2 Key Pair name when creating the stack so that SSH access is possible.

```bash
aws cloudformation create-stack --stack-name=CodeDeployEC2StackWrapper  --template-body=file:///home/akshay/Desktop/Dev/Tutes/AWS-CodeDeploy-Demo/al-stack.yaml --capabilities=CAPABILITY_IAM --region us-east-1 --parameters ParameterKey=EC2KPName,ParameterValue=adesk
```

## 1.2. Create the CodeDeploy Application

The AWS Console may be the easiest way to achieve this. Create a CodeDeploy EC2 application that has the name
`TestApp` (the GitHub Actions CI expects this name) and a Deployment Group called `TestAppDepGrp`. In that deployment
group:

- Attach the Service Role created from the previous stack creation
- Choose EC2 as the Environment Configuration and use the Tag key: `Name` and the Tag value `CodeDeployEC2Stack`
- Disable load balancing
- Use an in-place, all-at-once deployment

# 2. Ubuntu Server Stack

In this stack, we are on our own. We create everything needed to configure CodeDeploy from scratch for an Ubuntu Server.
This includes manually provisioning the CodeDeploy agent using `cfn-init` as well as creation of associated IAM resources.
We also automate creation of CodeDeploy resources (Application + Deployment Group) for maximal automation!

See [`ubuntu-server-stack.yaml`](./ubuntu-server-stack.yaml).

The stack covers:

- An Ubuntu Server EC2 Instance configured with CodeDeploy agent
  using [cfn-init](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-init.html) helper script
    - Associated [Security Group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
    - Associated [EC2 Instance Profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)
    that allows our EC2 instance to access S3 to obtain our application bundle
    - An [IAM Role](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) attached to the
      Instance Profile mentioned above
- A CodeDeploy Application, and
  a [DeploymentGroup](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-groups.html#deployment-group-server)
    - Associated [Service Role](https://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-create-service-role.html)
    to give the CodeDeploy service access to EC2

## 2.1. Push the Ubuntu Server stack to AWS

One command, that's it! No fiddling around the CodeDeploy console!

Example command:

**Ensure that you pass in a valid EC2 Key Pair name when creating the stack so that SSH access is possible.**

```bash
aws cloudformation create-stack --stack-name=CodeDeployEC2StackWrapper  --template-body=file:///home/akshay/Desktop/Dev/Tutes/AWS-CodeDeploy-Demo/ubuntu-server-stack.yaml --capabilities=CAPABILITY_IAM --region us-east-1 --parameters ParameterKey=EC2KPName,ParameterValue=adesk
```
