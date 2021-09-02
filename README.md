# AWS-CodeDeploy-Demo

Demo tutorial using AWS CodeDeploy via GitHub.

## Prerequisities
- AWS CLI
- EC2 Key Pair with the name `adesk` (used in the CF stack)
- Calling IAM user with necessary permissions

## Push the CF stack to AWS

The CF stack internally wraps an AWS-provided CF stack for provisioning an AWS EC2 Amazon Linux instance with the
CodeDeploy agent installed. For more details
see [here](https://docs.aws.amazon.com/codedeploy/latest/userguide/instances-ec2-create-cloudformation-template.html#instances-ec2-create-cloudformation-template-cli).

 It also creates necessary IAM resources, such as:
- a Service Role that allows CodeDeploy to access EC2;
- an [EC2 instance profile](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html)
that allows our EC2 instance to access S3 to obtain our application bundle

Example command:

```bash
aws cloudformation create-stack --stack-name=CodeDeployEC2StackWrapper  --template-body=file:///home/akshay/Desktop/Dev/Tutes/AWS-CodeDeploy-Demo/stack.yaml --capabilities=CAPABILITY_IAM --region us-east-1
```

## Create the CodeDeploy Application

The AWS Console may be the easiest way to achieve this. Create a CodeDeploy EC2 application that has the name
`TestApp` (the GitHub Actions CI expects this name) and a Deployment Group called `TestAppDepGrp`. In that deployment
group:
- Attach the Service Role created from the previous stack creation
- Choose EC2 as the Environment Configuration and use the Tag key: `Name` and the Tag value `CodeDeployEC2Stack`
- Disable load balancing
- Use an in-place, all-at-once deployment
