## How to Automatically Revert Changes on EC2 Security Groups

we'll walk you through setting up an automated process to revert unauthorized changes to your EC2 security groups.

This setup involves creating a custom VPC security group, setting up AWS CloudTrail, Lambda, and CloudWatch, and using a Lambda function to revert any changes. 

Follow these steps to ensure your security groups remain consistent and secure.

### 1. Create a Custom VPC Security Group

First, you need to create a custom VPC security group. This group will be monitored for changes, and if any unauthorized changes are detected, they will be automatically reverted.

1. Navigate to the **VPC Dashboard** in the AWS Management Console.
2. Click on **Security Groups** and then **Create security group**.
3. Give your security group a tag with the name `prod-sg`.

### 2. Create a CloudTrail Trail

AWS CloudTrail will help you monitor API calls made to your security groups.

1. Go to the **CloudTrail Dashboard**.
2. Click on **Trails** and then **Create trail**.
3. Choose a name for your trail and ensure it’s set up to log events for the region where your security group resides.

### 3. Create a Lambda Execution Role

You need to create an IAM role that grants Lambda the necessary permissions to interact with EC2 and CloudWatch.

1. Go to the **IAM Dashboard**.
2. Click on **Roles** and then **Create role**.
3. Choose the **Lambda** service and attach the following policies: `AmazonEC2ReadOnlyAccess` and `CloudWatchLogsFullAccess`.

### 4. Create a Lambda Function

Create a Lambda function that will be triggered by CloudWatch Events when changes are detected in your security group.

1. Go to the **Lambda Dashboard** and click **Create function**.
2. Choose **Author from scratch** and configure the function to use the execution role you created.
3. Use the following Python code for the Lambda function:

```python
import os
import json
import boto3

# Set the global variables
globalVars = {}
globalVars['Owner'] = "tamilcloud"
globalVars['Environment'] = "Test"
globalVars['REGION_NAME'] = "xxxxxxxxx"
globalVars['tagName'] = "prod-sg"
globalVars['security_group_id'] = os.environ['security_group_id']

def lambda_handler(event, context):
    print(event)
    print(globalVars['security_group_id'])

    if 'detail' not in event or 'eventName' not in event['detail']:
        return {"Result": "Failure", "Message": "Lambda not triggered by an event"}

    if (event['detail']['eventName'] == 'AuthorizeSecurityGroupIngress' and
            event['detail']['requestParameters']['groupId'] == globalVars['security_group_id']):
        result = revoke_security_group_ingress(event['detail'])

        message = f"AUTO-MITIGATED: Ingress rule removed from security group: {result['group_id']} that was added by {result['user_name']}: {json.dumps(result['ip_permissions'])}"

def revoke_security_group_ingress(event_detail):
    request_parameters = event_detail['requestParameters']
    ip_permissions = normalize_paramter_names(request_parameters['ipPermissions']['items'])

    response = boto3.client('ec2').revoke_security_group_ingress(
        GroupId=request_parameters['groupId'],
        IpPermissions=ip_permissions
    )

    result = {}
    result['group_id'] = request_parameters['groupId']
    result['user_name'] = event_detail['userIdentity']['arn']
    result['ip_permissions'] = ip_permissions

    return result

def normalize_paramter_names(ip_items):
    new_ip_items = []

    for ip_item in ip_items:
        new_ip_item = {
            "IpProtocol": ip_item['ipProtocol'],
            "FromPort": ip_item['fromPort'],
            "ToPort": ip_item['toPort']
        }

        if 'ipv6Ranges' in ip_item and ip_item['ipv6Ranges']:
            ipv_range_list_name = 'ipv6Ranges'
            ipv_address_value = 'cidrIpv6'
            ipv_range_list_name_capitalized = 'Ipv6Ranges'
            ipv_address_value_capitalized = 'CidrIpv6'
        else:
            ipv_range_list_name = 'ipRanges'
            ipv_address_value = 'cidrIp'
            ipv_range_list_name_capitalized = 'IpRanges'
            ipv_address_value_capitalized = 'CidrIp'

        ip_ranges = []

        for item in ip_item[ipv_range_list_name]['items']:
            ip_ranges.append(
                {ipv_address_value_capitalized: item[ipv_address_value]}
            )

        new_ip_item[ipv_range_list_name_capitalized] = ip_ranges
        new_ip_items.append(new_ip_item)

    return new_ip_items
```

### 5. Add CloudWatch Event Trigger

Configure CloudWatch Events to trigger your Lambda function whenever a change is made to security groups.

1. Go to the **CloudWatch Dashboard** and click **Rules**.
2. Click **Create rule** and set the event source to **Event Source**.
3. Choose **EC2 Security Group** and set the detail type to **AWS API Call via CloudTrail**.
4. Add your Lambda function as the target and configure any additional settings as needed.

### 6. Test the Setup

To ensure everything is working correctly, manually change the rules in your security group and verify that the changes are reverted automatically. Monitor CloudWatch Logs and Lambda execution results to confirm that the process is functioning as expected.

---

Automating the management of security groups with AWS Lambda and CloudWatch Events can help maintain the integrity of your infrastructure by automatically reverting unauthorized changes. This setup not only saves time but also ensures that your security policies are consistently enforced.

For more details and the complete code, check out our GitHub repository [Automatically-Revert-SG-Rule](https://github.com/techforall247/Automatically-Revert-SG-Rule).

Feel free to reach out if you have any questions or need further assistance!