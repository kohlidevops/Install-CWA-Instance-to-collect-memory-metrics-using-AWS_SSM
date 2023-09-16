# Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM

******To Collect the Memory Metrics from Amazon Linux EC2 Instance using AWS System Manager with the help of Run Command and Parameter Store******

AWS CloudWatch does not collect memory metrics from EC2 Instances by default. To obtain the memory utilisation, we first install the CloudWatch Agent on an EC2 instance. In this section, we will install the AWS CloudWatch agent using AWS System Manager and gather memory metrics in the AWS CloudWatch dashboard. We will then build an alarm in AWS CloudWatch that will send an alert email message when the memory goes skyrockets.

<img width="477" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/8b5e3eb4-2d81-4cb7-80e8-f345e82ffff1">

I assuming that you have launched Amazon Linux EC2 instance and its up and running state.

****Step -1: Create IAM Role and associate role to EC2 Instance****

Navigate to IAM and create new Role with EC2 as Trusted Identity.
<img width="680" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/583c98a1-58c3-4e75-bc6a-a7a5b40b9e40">

Make ensure your IAM Role should have below policies

<img width="742" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/0cac5252-8cb9-4549-b56f-bb7805a4aafe">

Then associate this IAM Role to your EC2 instance.

****Step -2: Install SSM Agent on Amazon EC2 Instance****

SSH into your EC2 Instance and follow the below steps to install the SSM agent.

Download the SSM agent and run the agent installer using the below command.

#sudo yum install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm

**To start the amazo-ssm-agent service, run the below command.**

#sudo systemctl start amazon-ssm-agent

**To enable the amazon-ssm-agent service to start on system reboot or boot up.**

#sudo systemctl enable amazon-ssm-agent

**To check the status of the ssm agent.**

#sudo systemctl status amazon-ssm-agent

****Step -3: Install CloudWatch Agent on Amazon Linux EC2 Instance****

To install the CloudWatch agent to the EC2 Instance go to the AWS Systems Manager service on the left side scroll down under Node Management select "Run Command".

<img width="959" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/a10d14d3-0514-4cd6-9896-fda647705eea">

Once you selected the Run Command you will be promoted to the Run Command Dashboard then, on the right side click on Run Command Button.

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/bc56f5e1-a58d-4641-8149-7c987a5b2cf6)

On the Command section select the “AWS-ConfigureAWSPackage” to install the Cloudwatch agent to the EC2 Instance.

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/600d74bd-0568-4c7f-9c27-bfa3ed8dbc6d)

Now, Select the Target scroll down to the Target selection and "Choose instances manually" option, and leave everything default.

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/c7183e25-2329-4619-9d49-0853eb786f6d)

We don’t need the command output to store in an S3 bucket So, In the Output options deselect the options shown in the image below.

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/462ab2e5-9721-49a8-a6df-6c7b12576105)

Now you click on Run Command then it will show the Successful status which means CloudWatch Agent has been successfully installed on Amazon Linux EC2 Instance. 
Perfect!

****Step -4: Setup CloudWatch agent configuration to the AWS SSM Parameter Store****

Now, We are going to add the CloudWatch agent configuration on the Parameter Store to collect memory metrics from our target EC2 instance. 

Go back to the System Manager service, on the left side under Application Management click on Parameter store.

<img width="926" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/25268eb7-e38a-4b3a-9db8-f9050cc55eb3">

Click on the Create parameter.

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/85a79efd-7a41-4d25-abe1-3a550590c809)

Give a unique name "AmazonCloudWatch-MemoryConfig" and add a description select Standard Tier, Type "String", and Data type "Text".

<img width="852" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/d47eb397-7423-4c10-b3db-8903881ea938">

Just add the below Json file to the Value section.

     {
         "agent": {
           "metrics_collection_interval": 30,
           "logfile": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log"
         },
       "metrics": {
    "metrics_collected": {
       "mem": {
         "measurement": [
           {"name": "used", "rename": "MemoryUsed"},
           {"name": "mem_available", "rename": "MemoryAvailable"}
         ]
         }
        },
      "append_dimensions": {
        "InstanceId": "${aws:InstanceId}"
       }
       }
      }

Now, click on Create parameter.

****Step -5: Install CloudWatch agent configuration using Amazon System Manager****

To install the CloudWatch agent configuration on the EC2 Instance. Go back to the System manager Run Command.

Search for “AmazonCloudWatch-ManageAgent” and select it to install the CloudWatch manage agent which will install the configuration.

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/65a436ce-2413-4bf5-b8d7-4a250641a2b2)

In the Command parameters add the "Optional Configuration Location" to the parameter name that we have created in the previous step. Make ensure the Configuration Location name should be same as the parameter name that is "AmazonCloudWatch-MemoryConfig".

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/223b2176-cc7e-400a-8052-9cdfeda532c4)

Under Target Selection “Choose instances manually” option and select your instances.

<img width="730" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/bdc757b6-01bb-4419-a917-c4d2875d43c8">

Then Click on Run Command to install the CloudWatch Agent Configuration on target EC2 Instance using System Manager. Once successful the command, then you can see the Memory metrics in AWS CLoudWatch.

****Step -6: Check the Memory Metrics On AWS CloudWatch****

After successfully installing the CloudWatch agent and CloudWatch memory metrics configuration then go to the AWS CloudWatch service on the left side under Metrics and select "All metrics" you will see "CWAgent" under custom namespaces click on that.

<img width="954" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/b5a731e7-fea1-4361-88f1-05d0c46ca766">

Now, Click on the InstanceId and select your instances to view metrics.

<img width="778" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/e841bb3d-ea6d-4922-bb41-4a53b75875c7">

<img width="803" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/017ea81e-75a7-4e16-a746-5f0620095c7e">

****Step -7: Create an SNS Topic****

Go to the SNS service and select topics on the left side and click on Create Topic.

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/8379e35f-4f08-4566-b432-deed1960b7ae)

Select "Standard" Type and give a name to the SNS topic like "CWAgent- Alert" leave the rest of the options default and click on Create Topic.

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/28b4d051-5d11-478f-8cff-aa6c0a5664cf)

Now, Subscribe to the SNS topic with the email address and click on Create subscription.

<img width="800" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/ee7f3303-6054-450b-b385-fd4ea887458d">

Go to your mail domain you can see a mail to confirm the subscription click on confirm subscription link to confirm your subscription.

****Step -8: Create a CloudWatch Alarm****

Go back to the CloudWatch service and on the right side select All "alarms" and then click on "Create alarm"

![image](https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/3b5b8f80-c57f-4485-9b92-f5d528e54479)

Under "Specify metric and conditions" Select custom namespaces "CWAgent" and then click on "InstanceId". Select the "MemoryUsed" metric which will only monitor the memory usage of the EC2 Instance.

<img width="876" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/f5c2afcc-dbd2-4272-b74d-0f5fc0c53701">

Now, select the metric period to one minute.

<img width="803" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/2c9f2c81-05bc-412d-8a98-3938723e85e1">

Select the Threshold type "Static" add the condition "Greater/Equal" and then enter the threshold value of 70. This means if the memory usage is more than 70% then it will be in an alarm state. Leave the Additional configuration default and click Next.

<img width="740" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/9f6b96dd-e901-4373-a311-5dd2d3a996e0">

In Configure Action Step select Alarm trigger state to "In alarm" select an existing SNS topic and choose the SNS topic from the drop-down menu that we have created in the previous step. In our case, it is "CWAgent-Alert" and then click on Next.

<img width="661" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/fea06869-d482-4297-a420-f59bfe886c78">

Add the Alarm name and then click on Next.

<img width="649" alt="image" src="https://github.com/kohlidevops/Install-CWA-Instance-to-collect-memory-metrics-using-AWS_SSM/assets/100069489/d588bc2c-c45d-4c33-991d-470e183e849c">

Scroll down preview the steps and click on Create alarm.

**Here we go!
Now we have successfully created the cloudwatch alarm which will trigger when the Amazon Linux EC2 instance memory goes above 70 %.**

That's it.


