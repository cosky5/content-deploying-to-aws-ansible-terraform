# Deploying to AWS with Terraform and Ansible Repo

## The Senario
You were recently hired as an Infrastructure Automation Engineer at a SaaS company. The company is trying to move away from cloud-provider-specific infrastructure as code. They want to test out Terraform for infrastructure deployment as it is cloud agnostic and Ansible as it is OS agnostic and also a hybrid IaC tool.
Your first task is to use Terraform and Ansible to deploy a distributed Jenkins CI/CD pipeline and put it behind one of the company's DNS domains for testing. It sounds easy enough but there's quite some planning which will go into this and you're already on top of it.

We have to sping up a Jenkins Master node (EC2) in us-east-1 and Jenkins Worker (EC2) in us-west-2. These nodes needs to communicatate therefore, we'll need to set up appropriate VPC peering, route table enteirs, and security group rules to allow that. The Jenkins node's web server listens on port `8080/tcp`and the Jenkins Worker will need to be able to communicate with the Jenkins Master on all ports over TCP.

Username and password for our Jenkins Application have been provided in plaintext file in the following URLs:

 a. [Jenkins Master Ansible Playbook](https://raw.githubusercontent.com/linuxacademy/content-deploying-to-aws-ansible-terraform/master/aws_la_cloudplayground_multiple_workers_version/ansible_templates/install_jenkins.yaml)
 
 b. [Jenkins Worker Ansible Playbook](https://raw.githubusercontent.com/linuxacademy/content-deploying-to-aws-ansible-terraform/master/aws_la_cloudplayground_multiple_workers_version/ansible_templates/install_worker.yaml)
 
 c. [Ansible Jinja template for Jenkins Worker Setup](https://raw.githubusercontent.com/linuxacademy/content-deploying-to-aws-ansible-terraform/master/aws_la_cloudplayground_multiple_workers_version/ansible_templates/node.j2)
 
 d. [Ansible Jinja template for Jenkins Worker credential setup](https://raw.githubusercontent.com/linuxacademy/content-deploying-to-aws-ansible-terraform/master/aws_la_cloudplayground_multiple_workers_version/ansible_templates/cred-privkey.j2)
 
 e. [Plaintext Jenkins Auth file](https://github.com/linuxacademy/content-deploying-to-aws-ansible-terraform/blob/master/aws_la_cloudplayground_multiple_workers_version/ansible_templates/jenkins_auth)

We also have to create a directory inside the our ansible_templates called `ìnventory_aws` and store the Ansible dynamic inventory fecthing config file there. The URL to the file is [Ansible Inventory Config for AWS](https://raw.githubusercontent.com/linuxacademy/content-deploying-to-aws-ansible-terraform/master/aws_la_cloudplayground_multiple_workers_version/ansible_templates/inventory_aws/tf_aws_ec2.yml)

### Log in to the Node EC2 Instance
Since we have our setup in place, it time to start working on the project. First, let configure our SSH:
```
ssh cloud_user@<IP-OF-TERRAFORM-CONTROLLER>
```
Note: This Instance already has an EC2 instance profile (role) attached to it and has all necessary AWS API permissions required for this lab. It also has the AWS CLI set up and is configured with the AWS account attached to this lab, for which the console login credentials are also provided in the lab interface page once the lab spins up.

After logging in, check the version of Terraform that is installed. Execute the following command to check:
```
 terraform version
```

### Clone the GitHub Repo for Terraform Code

Use the git command to clone the GitHub repo which has the Terraform code to deploy to complete this lab. [GitHub repo URL](https://github.com/linuxacademy/content-deploying-to-aws-ansible-terraform.git).

1. Then we execute the following command:
```
git clone https://github.com/linuxacademy/content-deploying-to-aws-ansible-terraform.git
```
2. And move into the `aws_deployment_folder`
```
cd content-deploying-to-aws-ansible-terraform/aws_la_cloudplayground_multiple_workers_version
```

3. Execute `ls`and examine the contents of the directory you're in.

4. View the contents of `backend.tf` to view the backend information for storing state files.
```
less backend.tf
```

### Configure an S3 Bucket
1. Create the S3 bucket, providing a unique bucket name:
```
aws s3api create-bucket --bucket <UNIQUE_BUCKET_N1. AME> mynewstatebucket105
```

2. Copy the unique bucket name after it is successfully created. Edit the backend.tf file and replace "" for the bucket variable with your unique bucket name.

3. Save and close the file
```
:wq
```

### Configure Route 53 Public DNS
1. Display the domain name:
```
aws route53 list-hosted-zones | jq -r .HostedZones[].Name | egrep "cmcloud*"
```
2. Copy the result of the previous command and edit the `variables.tf` file.
```
vim variables.tf
```
3. Navigate to the variable `dns-name` stanza and replace the text "" with the domain name we copied earlier. Be sure to include the `. ` at the end of the domain name.

4. Save and close the file:
```
:wq
```

### Create an SSH Key Pair
1. Create the key pair, pressing **Enter** three times after running the command to accept the defaults:
```
ssh-keygen -t rsa
```

### Deploy the Terraform Code
1. Initialize the Terraform directory you changed into to download the required provider:
```
 terraform init
 ```

2. Ensure Terraform code is properly formatted:
```
 terraform fmt
```
 
3. Ensure code has proper syntax and no errors:
```
 terraform validate
```
4. See the execution plan and note the number of resources that will be created:
```
terraform plan
```
5. Deploy resources
```
terraform apply
```
6. Enter `yes` when prompted.

After terraform apply has run successfully, you can use the AWS CLI on the Controller node to list, describe created resources, and additionally also log in to the AWS Console to verify and investigate created resources.

### Test Out Your Deployment
1. Test out the URL of your website returned in Terraform outputs.

In a new web browser tab, navigate to the URL provided in the results of the terraform apply command run previously. Use the following Jenkins credentials:

```
Username: admin
Password password
```
2. Changing the workers count by modifying workers-count variable in the variables.tf file and ensure that Terraform apply is successful.

Click Manage Jenkins from the menu on the left of the page.
Click Manage Nodes and Clouds from the System Configuration section.
From your terminal, edit the variables.tf file and increase the workers-count variable from **1** to **0**.
```
 vim variables.tf
```
Run `terraform apply` again to apply this configuration change. Enter yes when prompted.
Back in your web browser, refresh the page to verify the number of worker nodes is updated.


Finally, on the Terraform Controller node CLI, execute terraform destroy and enter yes when prompted to delete all resources which were created and ensure that it runs through successfully.

This repo will contain code used and taught in the A Cloud Guru course named above.

This course was originally created before Terrafom 0.13 was released however I have updated it to work with version 0.13 in the `terraform_v13_compatible_code` folder.


------

For following along using Terraform 0.12 refer to the `aws_la_cloudplayground_multiple_workers_version`.

Again, for following along using Terraform 0.13 refer to the `terraform_v13_compatible_code`.

Although the folder naming convention in this repository should make sense for the most part, it was primarily created to be referred through the lessons on the A Cloud Guru website.


For queries and feedback please reach out to ACG support: https://help.acloud.guru

## Warning
1. Parts of this repository expect users to obtain a Route53 domain name, which is available with ACG Playground tier subscription.
2. Following along and deploying resources in AWS as taught by this course WILL incur charges!!! Be sure to destroy any infrastructure that you do not need.

---


Copyright 2020 A CLOUD GURU

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
