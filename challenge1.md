#Challenge 1: An S3 bucket was configured with a public IAM policy, without permission to  access the list and download the objects. My challenge was to identify the security risks associated with the bucket by analyzing available access permissions. I also need to identify a flag stored in one of the accessible objects within the files/ directory. Once I identify the flag, I will submit the flag to complete the challenge.

Analysis of the challenge policy:
I have public access to list the objects within the files/ directory due to the s3:ListBucket action with the s3:prefix condition, download any object in the bucket (s3:GetObject) regardless of its directory.
I don’t have access to list objects outside the files/ directory, perform other actions like deleting, uploading, or modifying objects.
Granting public access to list and download files in a specific directory is a clear security risk. Although the s3:prefix limits listing to the files/ directory, the s3:GetObject permission allows public downloads of all objects in the bucket. I find there was some flaws here. An attacker could also list the contents of the files/ directory and download all files, analyze the data for sensitive information, leading to data leaks or misuse
This misconfiguration could have been avoided by, Avoiding the use of Principal: * in the policy. Limiting access to specific authenticated users or IP addresses. Regularly auditing S3 bucket permissions and using AWS tools like S3 Block Public Access or IAM Access Analyzer to identify misconfigurations.

Steps I used to solve the challenge:
Used the AWS CLI to list files in the files/ directory with code aws s3 ls s3://thebigiamchallenge-storage-9979f4b/files/ --no-sign-request which command displayed the list of files in the files/ directory.
From the list, I noted filenames that could potentially store the flag. For instance, files with names like flag.txt were candidates
I downloaded a file that I suspected contained the flag using command aws s3 cp s3://thebigiamchallenge-storage-9979f4b/files/flag.txt ./ --no-sign-request
After downloading i openend a file using cat and was able to find a flag and I submitted the flag and it was shown success.

In this challenge, I followed a step-by-step approach to understand the IAM policy, list accessible files, and retrieve the flag. The biggest challenge was figuring out the limitations of the policy, particularly the restrictions imposed by the s3:prefix condition. To overcome this, I carefully analyzed the policy and tested commands to stay within the allowed permissions. The key breakthrough was realizing that I needed to focus only on the files/ directory. This challenge reinforced the importance of avoiding public access in S3 buckets and using security best practices like limiting permissions, enabling logging, and conducting regular audits to prevent similar misconfigurations.
