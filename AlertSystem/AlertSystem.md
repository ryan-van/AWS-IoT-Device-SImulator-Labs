# Alert System Lab

## Introduction
In this lab you will get acquainted with the IoT Device Simulator, which creates a web interface that allows you to simulate devices that send data to IoT Core. You will make an alert system that notifies you of extremities in temperature, water level, and air quality.

## Architecture
![Architecture](/images/architecture.jpg)

## Prerequisites
* AWS Account
* Basic understanding of Cloud Services

## Step 1 - Create the Device Simulator Interface
Let’s first setup IoT Device Simulator using a CloudFormation template:
* Login to the AWS Console.
* Follow this [link](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?templateURL=https:%2F%2Fs3.amazonaws.com%2Fsolutions-reference%2Fiot-device-simulator%2Flatest%2Fiot-device-simulator.template)
* Change the region to match your region (currently only tested to work in US-West-2 or US-East-1)
![Device Simulator 1](/images/devicesim1.jpg)
* For Specify Details, follow this image. Make sure to use your own admin name and insert your aws email for Administrator email under parameters
![Device Simulator 2](/images/devicesim2.jpg)
* On the options page, select next.
* For the Review page, confirm your settings and check the acknowledgment box for IAM resources
    * Click create stack
 ![Device Simulator 3](/images/devicesim3.jpg)
* Here, it will take about 10 minutes to prepare the CloudFormation template. You should receive an email in the meantime.

## Step 2 - Create an AWS Simple Notification Service (SNS) Topic
While waiting for the CloudFormation Template to be created, we will focus on creating the SNS Topic.
* Go to the SNS website (make sure your region is correct)
* Create Topic
* Under name, use any name you want (we used iotSNS)
* Leave the rest of the settings default and Create Topic
* Under our created topic, we can create our subscription to an end user device, here you may choose which device you want to send our alerts to
    * Select the protocol of your choice (here, we’re going to use email)
 ![SNS 1](/images/sns1.jpg)
    * You must confirm on the device / protocol that you want to subscribe
    * NOTE: Charges may apply for SNS on mobile devices

## Step 3 - Create a device type and simulate an alert device
Setup your Device Simulator Account by following the email you received. Once that is done, we’ll get started with creating our device type.
* After creating your account and setting the admin user, you should enter a screen similar to this:
![Device Type 1](/images/devicetype1.jpg)
* We’re going to create a Device Type now.
    * Click on Create a Device Type
    * Name it iotAlert
    * for Data Topic, make the topic: /iot/alert
    * Data Transmission can be left as default
        * \*Note: Duration is how long the device is simulated, Interval is how often a sample is collected
    * For Message Payload, we’re going to add the following attributes:
        * Name:
            * Attribute Name: Name
            * Data Type: DEVICE ID
        * Time:
            * Attribute Name: Time
            * Data Type: UTC TIMESTAMP
        * Temperature:
            * Attribute Name: Temperature (C)
            * Data Type: Float
            * Integer Min: -5
            * Integer Max: 55
        * Air Quality:
            * Attribute Name: Air Quality Index
            * Data Type: Integer
            * Min: 0
            * Max: 350
        * Water Level:
            * Attribute Name: Water Level (cm)
            * Data Type: Float
            * Integer Min: 0
            * Integer Max: 22
    * Save the Device Type:
 ![Device Type 1](/images/devicetype2.jpg)
* Click Widgets on the left
    * Add a widget
* You can click on view to see the data being transmitted
You’re first device is created and data is being simulated! Let’s move onto creating our lambda function to filer out events in the case of extremities.

## Step 4 - Create an AWS Lambda function
We’re going to now create our Lambda function and send data to our SNS topic
* Go to the lambda console
* Create a new lambda function from scratch:
![Lambda 1](/images/lambda1.jpg)
    * Under permissions, create a new role from AWS policy templates;
    * Name the role and under policy templates, choose SNS Publish Policy
 ![Lambda 2](/images/lambda2.jpg)
* Edit the code in the lambda function:
```python
from __future__ import print_function
import json
import boto3
def lambda_handler(event, context):
    # Parse the JSON message, convert it into a string
    eventText = json.dumps(event)

    response = event
    on = False

    temperature = response['Temperature (C)']
    air_quality = response['Air Quality Index']
    water_level = response['Water Level (cm)']

    if temperature < 0 or temperature > 50:
        eventText = "Extreme temperature warning \n" + eventText
        on = True
    if air_quality > 300:
        eventText = "Unhealthy Air Quality \n" + eventText
        on = True
    if water_level > 20:
        eventText = "Flash Flood possbile \n" + eventText
        on = True

    if on:
        # Create an SNS client
        sns = boto3.client('sns')

        # Publish a message to the specified topic
        response = sns.publish (
          # Should look like 'arn:aws:sns:us-east-1:1111111111111:alertTest'
          TopicArn = 'arn:aws:sns:region:accountid:alertTest',
          Message = eventText
        )
```
* Make sure to change your TopicArn to the one from your SNS Topic

## Step 5 - Connect your devices to IoT Core and AWS Lambda
We’re now going to create a trigger for our Lambda function, which will be data being sent into Lambda from our Device Simulator
* Go to AWS IoT Core → Act → Rule
![IoT Core 1](/images/iotcore1.jpg)
* Name it iotAlert
* For the SQL statement, change it to our topic: SELECT \* FROM '/iot/alert'
* Add Action:
    * Select “Send a message to a Lambda function”
    * Select the lambda function we created, iotAlert
* Create the Rule
![IoT Core 1](/images/iotcore2.jpg)
We have now connected our IoT Device to our Lambda function, which is connected to SNS

## Step 6 - Test
Now we’re going to test our alert system!
* Enter the Device Simulator Console and under widgets, click view for the device we created
* Start the device, and note that you may either receive 0 or multiple notifications depending on the random data simulated and depending on the conditions we restrained our lambda function to. If you didn’t receive any, try running the simulation again.
* If you don’t see any data going to your device, you can check the CloudWatch metrics in Lambda under Monitoring to see if an error has occurred or if your data is being sent correctly.
Congratulations, you’ve completed our first Device Simulator Lab. Remember to clean up your resources so you won’t be charged for  these anymore:
* CloudFormation template (Keep this for more Device Simulator Labs)
* S3 Bucket created (Keep this for more Device Simulator Labs)
* SNS Topic
* Lambda Function
