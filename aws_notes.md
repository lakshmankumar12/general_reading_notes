# Accessing AWS

* AWS management console - browser based interface
* AWS commandline tools
* AWS SDKS - software development kits
    * code in various programming languages

* ARN

    Amazon Resource Names (ARNs) uniquely identify AWS resources. We require an ARN
    when you need to specify a resource unambiguously across all of AWS, such as in
    IAM policies, Amazon Relational Database Service (Amazon RDS) tags, and API
    calls.

    Eg: `arn:aws:iam::123456789012:user/Development/product_1234/*`
        `arn:aws:iam::304109252636:user/abhishek.goyal`


# User

* IAM - AWS identity and access management

root user
    - the account with which the AWS account was created.
IAM user
    - policies associate what the user can do
    - has a password/access/key
IAM usergroups
    - policies can be associated with user groups

# awscli

* Have a `~/.aws/credentials` that has
```
[default]
aws_access_key_id=ACCESS_KEY
aws_secret_access_key=SECRET_KEY
```
* Have a `~/.aws/config` that has
```
[default]
regions=us-east-1
```

# boto3 library

## iam

```python

import boto3

## Doubts: Not sure how the credentials are picked.
## May be from ~/.aws/credentials
iam_client = boto3.client('iam')

## No policy will be attached to this user yet.
response = iam_client.create_user(UserName=username)

# list all users
paginator = iam_client.get_paginator('list_users')
for response in paginator.paginate():
    for user in response['Users']:
        username = user['UserName']
        Arn = user['Arn']
        print('Username : {} Arn : {}'.format(username, Arn))

#update a user
response = iam_client.update_user(
    UserName=old_username,
    NewUserName=new_username
)

# policy creation
user_policy = {
    "Version":"2012-10-17",
    "Statement":[
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
response = iam_client.create_policy(
    PolicyName = 'pyFullAccess',
    PolicyDocument=json.dumps(user_policy)
)

# policy listing.
# paginator -- typically lists 100 per page
paginator = iam_client.get_paginator('list_policies')

## attach policy
response = iam_client.attach_user_policy(
    UserName=username,
    PolicyArn=policy_arn
)
response = iam_client.detach_user_policy(
    UserName=username,
    PolicyArn=policy_arn
)

##  note the Scope
##     AWS   - amazon added policies
##     Local - user created policies
##     All   - Both (default)
##  https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iam.html#IAM.Client.list_policies
##  https://docs.aws.amazon.com/cli/latest/reference/iam/list-policies.html
for response in paginator.paginate(Scope="AWS"):
    for policy in response['Policies']:
        policy_name = policy['PolicyName']
        Arn = policy['Arn']
        print('Policy Name : {} Arn : {}'.format(policy_name, Arn))


# attach user to a policy
response = iam_client.attach_user_policy(
    UserName=username,
    PolicyArn=policy_arn
)

# create user groups
iam_client.create_group(GroupName=group_name)


```


## ec2

```python
import boto3
cli = boto3.client('ec2',
                   region_name=regName,
                   aws_access_key_id=access_key
                   aws_secret_access_key=secret_key

cli.describe_regions()
cli.describe_availability_zones()


```

# Networking

## links

* https://medium.com/@mda590/aws-routing-101-67879d23014d
* https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html

## Notes

* VPC
    * Has a local subnet.
        * If you intend to connect 2 VPC (VPC-peering), they should have non overlapping local subnets
    * Has a implicit main route-table.
    * Can have multiple local subnets.
        * Each subnet should be associated with a route-table.
            * By default, subnets are associatd with the main VPN-route-table.
            * We can create explicit route-table and make that the subnet-route-table as well.
        * One Route-table can be the subnet-route-table for many subnets.
            * However, one subnet is associated with only one route-table at a time.
* Route
    * has a destination(cidr) and target
        * default dest is the `0.0.0.0/0`
        * choice of targets are
            * local
            * internet-gateway
            * nat-gateway
            * vpc-peer-connection
            * vpc-gateway


# Dump

* There are
    * subnets,
    * subnet-association-to-route-tables,
    * routes,
    * route-tables
    * security groups (associated to interfaces of EC2)
