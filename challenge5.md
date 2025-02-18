# CloudFoxable - First Flag Writeup

## Challenge Overview
**Challenge Name:** First Flag  
**Author:** Seth Art (@sethsec)  
**Default State:** Enabled  
**Estimated Cost:** No cost  
**Starting Point:** [GitHub Repository](https://github.com/BishopFox/cloudfoxable)

The "First Flag" challenge in CloudFoxable involves deploying a Terraform-based AWS environment designed for security testing and capturing flags. This write-up provides a step-by-step guide to deploying the environment, finding the flag, and cleaning up resources.

---

## Step 1: Setting Up the Environment
### 1.1 Select or Create an AWS Account
- Use an AWS account that does NOT contain any production resources or sensitive data.
- If you do not have an AWS account, create one at [AWS Signup](https://aws.amazon.com/).
- Ensure that billing alerts are configured to avoid any unexpected charges.

### 1.2 Create a Non-Root User with Administrative Access
- In the AWS Management Console, navigate to **IAM > Users**.
- Click "Add user" and provide a username.
- Select "AWS Management Console access" and "Programmatic access".
- Assign "AdministratorAccess" via an IAM policy.
- Complete the user creation process and securely store the credentials.

### 1.3 Install and Configure AWS CLI
- Install AWS CLI following the instructions [here](https://aws.amazon.com/cli/).
- Verify installation with:
  ```bash
  aws --version
  ```
- Configure AWS CLI with the newly created user:
  ```bash
  aws configure
  ```
  Enter the Access Key, Secret Key, Region, and output format as prompted.
- Verify the configuration:
  ```bash
  aws sts get-caller-identity
  ```
  This should return details about your IAM user, confirming successful setup.

### 1.4 Install Terraform
- Download and install Terraform from [here](https://developer.hashicorp.com/terraform/downloads).
- Add Terraform to your system path if necessary.
- Verify the installation:
  ```bash
  terraform version
  ```

### 1.5 Clone the CloudFoxable Repository
```bash
git clone https://github.com/BishopFox/cloudfoxable
cd cloudfoxable/aws
```

### 1.6 Configure Terraform
- Copy the example Terraform variables file:
  ```bash
  cp terraform.tfvars.example terraform.tfvars
  ```
- Edit `terraform.tfvars` using a text editor like Vim or Nano:
  ```bash
  nano terraform.tfvars
  ```
  Add your AWS profile:
  ```
  aws_local_profile = "YOUR_PROFILE"
  ```
  Save and exit the file.

### 1.7 Deploy the CloudFoxable Environment
- Initialize Terraform:
  ```bash
  terraform init
  ```
- Apply the Terraform configuration:
  ```bash
  terraform apply
  ```
- If you want to preview the changes before applying, run:
  ```bash
  terraform plan
  ```
- Confirm deployment by typing `yes` when prompted.

---

## Step 2: Finding the First Flag
- Once Terraform completes deployment, carefully **review the output**.
- The final Terraform output provides details about credentials or AWS CLI setup instructions related to **ctf-starting-user**.
- Set up your AWS CLI with the `ctf-starting-user` profile:
  ```bash
  aws configure --profile ctf-starting-user
  ```
- Run AWS commands to explore resources and locate the first flag:
  ```bash
  aws s3 ls --profile ctf-starting-user
  ```
- The flag format follows this pattern:
  ```
  FLAG{challengeName::CamelCaseText}
  
  ```
  Flag of this challenge: FLAG{congrats_you_are_now_a_terraform_expert_happy_hunting}
- Locate and submit the full flag string in the CTFd instance to complete the challenge.

---

## Step 3: Cleaning Up
- To remove all CloudFoxable resources, navigate to the deployment directory and run:
  ```bash
  terraform destroy
  ```
- Confirm deletion when prompted by typing `yes`.
- Verify that all resources are removed to avoid any lingering AWS charges.

---

## Conclusion
By following this guide, you successfully:
- Set up an AWS environment for CloudFoxable
- Deployed Terraform configurations
- Found and submitted the first flag
- Cleaned up AWS resources efficiently

This challenge serves as an excellent introduction to AWS security testing and penetration testing using cloud infrastructure. If you encounter any issues, revisit each step carefully or consult the [CloudFoxable GitHub page](https://github.com/BishopFox/cloudfoxable) for troubleshooting tips.

