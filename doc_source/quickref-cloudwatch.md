# Amazon CloudWatch Template Snippets<a name="quickref-cloudwatch"></a>

**Topics**
+ [Billing Alarm](#cloudwatch-sample-billing-alarm)
+ [CPU Utilization Alarm](#cloudwatch-sample-cpu-utilization-alarm)
+ [Recover an Amazon Elastic Compute Cloud Instance](#cloudwatch-sample-recover-instance)
+ [Create a Basic Dashboard](#cloudwatch-sample-dashboard-basic)
+ [Create a Dashboard with Side\-by\-Side Widgets](#cloudwatch-sample-dashboard-sidebyside)

## Billing Alarm<a name="cloudwatch-sample-billing-alarm"></a>

In the following sample, Amazon CloudWatch sends an email notification when charges to your AWS account exceed the alarm threshold\. To receive usage notifications, enable [billing alerts](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/monitor-charges.html)\.

### JSON<a name="quickref-cloudwatch-example-1.json"></a>

```
"SpendingAlarm": {
  "Type": "AWS::CloudWatch::Alarm",
  "Properties": {
    "AlarmDescription": { "Fn::Join": ["", [
      "Alarm if AWS spending is over $",
      { "Ref": "AlarmThreshold" }
    ]]},
    "Namespace": "AWS/Billing",
    "MetricName": "EstimatedCharges",
    "Dimensions": [{
      "Name": "Currency",
      "Value" : "USD"
    }],
    "Statistic": "Maximum",
    "Period": "21600",
    "EvaluationPeriods": "1",
    "Threshold": { "Ref": "AlarmThreshold" },
    "ComparisonOperator": "GreaterThanThreshold",
    "AlarmActions": [{
      "Ref": "BillingAlarmNotification"
    }],
    "InsufficientDataActions": [{
      "Ref": "BillingAlarmNotification"
    }]
  }
}
```

### YAML<a name="quickref-cloudwatch-example-1.yaml"></a>

```
SpendingAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmDescription: !Sub >
      "Alarm if AWS spending is over $${AlarmThreshold}"
    Namespace: AWS/Billing
    MetricName: EstimatedCharges
    Dimensions:
    - Name: Currency
      Value: USD
    Statistic: Maximum
    Period: '21600'
    EvaluationPeriods: '1'
    Threshold:
      Ref: "AlarmThreshold"
    ComparisonOperator: GreaterThanThreshold
    AlarmActions:
    - Ref: "BillingAlarmNotification"
    InsufficientDataActions:
    - Ref: "BillingAlarmNotification"
```

## CPU Utilization Alarm<a name="cloudwatch-sample-cpu-utilization-alarm"></a>

The following sample snippet creates an alarm that sends a notification when the average CPU utilization of an Amazon EC2 instance exceeds 90 percent for more than 60 seconds over three evaluation periods\.

### JSON<a name="quickref-cloudwatch-example-2.json"></a>

```
 "CPUAlarm" : {
   "Type" : "AWS::CloudWatch::Alarm",
   "Properties" : {
     "AlarmDescription" : "CPU alarm for my instance",
     "AlarmActions" : [ { "Ref" : "logical name of an AWS::SNS::Topic resource" } ],
     "MetricName" : "CPUUtilization",
     "Namespace" : "AWS/EC2",
     "Statistic" : "Average",
     "Period" : "60",
     "EvaluationPeriods" : "3",
     "Threshold" : "90",
     "ComparisonOperator" : "GreaterThanThreshold",
     "Dimensions" : [ {
       "Name" : "InstanceId",
       "Value" : { "Ref" : "logical name of an AWS::EC2::Instance resource" }
     } ]
   }
 }
```

### YAML<a name="quickref-cloudwatch-example-2.yaml"></a>

```
CPUAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmDescription: CPU alarm for my instance
    AlarmActions:
    - Ref: "logical name of an AWS::SNS::Topic resource"
    MetricName: CPUUtilization
    Namespace: AWS/EC2
    Statistic: Average
    Period: '60'
    EvaluationPeriods: '3'
    Threshold: '90'
    ComparisonOperator: GreaterThanThreshold
    Dimensions:
    - Name: InstanceId
      Value:
        Ref: "logical name of an AWS::EC2::Instance resource"
```

## Recover an Amazon Elastic Compute Cloud Instance<a name="cloudwatch-sample-recover-instance"></a>

The following CloudWatch alarm recovers an EC2 instance when it has status check failures for 15 consecutive minutes\. For more information about alarm actions, see [Create Alarms That Stop, Terminate, or Recover an Instance](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/UsingAlarmActions.html) in the *Amazon CloudWatch User Guide*\.

### JSON<a name="quickref-cloudwatch-example-3.json"></a>

```
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Parameters" : {
    "RecoveryInstance" : {
      "Description" : "The EC2 instance ID to associate this alarm with.",
      "Type" : "AWS::EC2::Instance::Id"
    }
  },
  "Resources": {
    "RecoveryTestAlarm": {
      "Type": "AWS::CloudWatch::Alarm",
      "Properties": {
        "AlarmDescription": "Trigger a recovery when instance status check fails for 15 consecutive minutes.",
        "Namespace": "AWS/EC2" ,
        "MetricName": "StatusCheckFailed_System",
        "Statistic": "Minimum",
        "Period": "60",
        "EvaluationPeriods": "15",
        "ComparisonOperator": "GreaterThanThreshold",
        "Threshold": "0",
        "AlarmActions": [ {"Fn::Join" : ["", ["arn:aws:automate:", { "Ref" : "AWS::Region" }, ":ec2:recover" ]]} ],
        "Dimensions": [{"Name": "InstanceId","Value": {"Ref": "RecoveryInstance"}}]
      }
    }
  }
}
```

### YAML<a name="quickref-cloudwatch-example-3.yaml"></a>

```
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  RecoveryInstance:
    Description: The EC2 instance ID to associate this alarm with.
    Type: AWS::EC2::Instance::Id
Resources:
  RecoveryTestAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: Trigger a recovery when instance status check fails for 15
        consecutive minutes.
      Namespace: AWS/EC2
      MetricName: StatusCheckFailed_System
      Statistic: Minimum
      Period: '60'
      EvaluationPeriods: '15'
      ComparisonOperator: GreaterThanThreshold
      Threshold: '0'
      AlarmActions: [ !Sub "arn:aws:automate:${AWS::Region}:ec2:recover" ]
      Dimensions:
      - Name: InstanceId
        Value:
          Ref: RecoveryInstance
```

## Create a Basic Dashboard<a name="cloudwatch-sample-dashboard-basic"></a>

The following example creates a simple CloudWatch dashboard with one metric widget displaying CPU utilization, and one text widget displaying a message\.

### JSON<a name="quickref-cloudwatch-sample-dashboard-basic.json"></a>

```
{
    "BasicDashboard": {
        "Type": "AWS::CloudWatch::Dashboard",
        "Properties": {
            "DashboardName": "Dashboard1",
            "DashboardBody": "{\"widgets\":[{\"type\":\"metric\",\"x\":0,\"y\":0,\"width\":12,\"height\":6,\"properties\":{\"metrics\":[[\"AWS/EC2\",\"CPUUtilization\",\"InstanceId\",\"i-012345\"]],\"period\":300,\"stat\":\"Average\",\"region\":\"us-east-1\",\"title\":\"EC2 Instance CPU\"}},{\"type\":\"text\",\"x\":0,\"y\":7,\"width\":3,\"height\":3,\"properties\":{\"markdown\":\"Hello world\"}}]}"
        }
    }
}
```

### YAML<a name="quickref-cloudwatch-sample-dashboard-basic.yaml"></a>

```
BasicDashboard:
  Type: AWS::CloudWatch::Dashboard
  Properties:
    DashboardName: Dashboard1
    DashboardBody: '{"widgets":[{"type":"metric","x":0,"y":0,"width":12,"height":6,"properties":{"metrics":[["AWS/EC2","CPUUtilization","InstanceId","i-012345"]],"period":300,"stat":"Average","region":"us-east-1","title":"EC2 Instance CPU"}},{"type":"text","x":0,"y":7,"width":3,"height":3,"properties":{"markdown":"Hello world"}}]}'
```

## Create a Dashboard with Side\-by\-Side Widgets<a name="cloudwatch-sample-dashboard-sidebyside"></a>

The following example creates a dashboard with two metric widgets that appear side by side\.

### JSON<a name="quickref-cloudwatch-sample-dashboard-sidebyside.json"></a>

```
{
    "DashboardSideBySide": {
        "Type": "AWS::CloudWatch::Dashboard",
        "Properties": {
            "DashboardName": "Dashboard1",
            "DashboardBody": "{\"widgets\":[{\"type\":\"metric\",\"x\":0,\"y\":0,\"width\":12,\"height\":6,\"properties\":{\"metrics\":[[\"AWS/EC2\",\"CPUUtilization\",\"InstanceId\",\"i-012345\"]],\"period\":300,\"stat\":\"Average\",\"region\":\"us-east-1\",\"title\":\"EC2 Instance CPU\"}},{\"type\":\"metric\",\"x\":12,\"y\":0,\"width\":12,\"height\":6,\"properties\":{\"metrics\":[[\"AWS/S3\",\"BucketSizeBytes\",\"BucketName\",\"MyBucketName\"]],\"period\":86400,\"stat\":\"Maximum\",\"region\":\"us-east-1\",\"title\":\"MyBucketName bytes\"}}]}"
        }
    }
}
```

### YAML<a name="quickref-cloudwatch-sample-dashboard-sidebysidequickref-cloudwatch-sample-dashboard-sidebyside.yaml"></a>

```
DashboardSideBySide:
  Type: AWS::CloudWatch::Dashboard
  Properties:
    DashboardName: Dashboard1
    DashboardBody: '{"widgets":[{"type":"metric","x":0,"y":0,"width":12,"height":6,"properties":{"metrics":[["AWS/EC2","CPUUtilization","InstanceId","i-012345"]],"period":300,"stat":"Average","region":"us-east-1","title":"EC2 Instance CPU"}},{"type":"metric","x":12,"y":0,"width":12,"height":6,"properties":{"metrics":[["AWS/S3","BucketSizeBytes","BucketName","MyBucketName"]],"period":86400,"stat":"Maximum","region":"us-east-1","title":"MyBucketName bytes"}}]}'
```
