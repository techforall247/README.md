# Automatically Revert Changes on EC2 Security Groups

Set up an automated process to revert unauthorized changes to your EC2 security groups using AWS CloudTrail, Lambda, and CloudWatch.

### 1. Create a Custom VPC Security Group
- Navigate to the **VPC Dashboard** in the AWS Management Console.
- Click on **Security Groups** and then **Create security group**.
- Tag your security group with the name `prod-sg`.

### 2. Create a CloudTrail Trail
- Go to the **CloudTrail Dashboard**.
- Click on **Trails** and then **Create trail**.
- Choose a name for your trail and ensure it logs events for the region where your security group resides.

### 3. Create a Lambda Execution Role
- Go to the **IAM Dashboard**.
- Click on **Roles** and then **Create role**.
- Choose the **Lambda** service and attach `AmazonEC2ReadOnlyAccess` and `CloudWatchLogsFullAccess` policies.

### 4. Create a Lambda Function
- Go to the **Lambda Dashboard** and click **Create function**.
- Choose **Author from scratch** and configure the function to use the execution role you created.
- Use the following Python code for the Lambda function:

```python
import os
import json
import boto3

# Set the global variables
globalVars = {
    'Owner': "tamilcloud",
    'Environment': "Test",
    'REGION_NAME': "xxxxxxxxx",
    'tagName': "prod-sg",
    'security_group_id': os.environ['security_group_id']
}

def lambda_handler(event, context):
    print(event)
    if 'detail' not in event or 'eventName' not in event['detail']:
        return {"Result": "Failure", "Message": "Lambda not triggered by an event"}

    if (event['detail']['eventName'] == 'AuthorizeSecurityGroupIngress' and
            event['detail']['requestParameters']['groupId'] == globalVars['security_group_id']):
        result = revoke_security_group_ingress(event['detail'])
        message = f"AUTO-MITIGATED: Ingress rule removed from security group: {result['group_id']} that was added by {result['user_name']}: {json.dumps(result['ip_permissions'])}"
        print(message)

def revoke_security_group_ingress(event_detail):
    request_parameters = event_detail['requestParameters']
    ip_permissions = normalize_paramter_names(request_parameters['ipPermissions']['items'])
    boto3.client('ec2').revoke_security_group_ingress(
        GroupId=request_parameters['groupId'],
        IpPermissions=ip_permissions
    )
    return {
        'group_id': request_parameters['groupId'],
        'user_name': event_detail['userIdentity']['arn'],
        'ip_permissions': ip_permissions
    }

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
        else:
            ipv_range_list_name = 'ipRanges'
            ipv_address_value = 'cidrIp'
        ip_ranges = [{ipv_address_value.capitalize(): item[ipv_address_value]} for item in ip_item[ipv_range_list_name]['items']]
        new_ip_item[ipv_range_list_name.capitalize()] = ip_ranges
        new_ip_items.append(new_ip_item)
    return new_ip_items
```

### 5. Add CloudWatch Event Trigger
- Go to the **CloudWatch Dashboard** and click **Rules**.
- Click **Create rule** and set the event source to **Event Source**.
- Choose **EC2 Security Group** and set the detail type to **AWS API Call via CloudTrail**.
- Add your Lambda function as the target.

### 6. Test the Setup
- Manually change the rules in your security group.
- Verify that the changes are reverted automatically.
- Monitor CloudWatch Logs and Lambda execution results to confirm functionality.


Feel free to reach out us if you have any questions or need further assistance!

[![Automatically-Revert-SG-Rule](https://img.shields.io/badge/GitHub-Automatically--Revert--SG--Rule-blue?logo=github)](https://github.com/techforall247/Automatically-Revert-SG-Rule)