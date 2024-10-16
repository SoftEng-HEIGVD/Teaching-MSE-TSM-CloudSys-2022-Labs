# Lab: Cloud Provisioning

_Duration of this lab: 2 sessions_

#### Pedagogical objectives

* Become familiar with AWS's proprietary tool for Infrastructure-as-Code, CloudFormation
* Use Terraform to provision cloud resource in a provider-independent way

#### Tasks

In this lab you will perform a number of tasks and document your progress in a lab report (PDF file). Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

#### Allocating and freeing cloud resources

MSE is paying for your use of Amazon cloud. Please use the service responsibly:

* Launch virtual machines and other resources only when you are ready to work with them.
* Shut down virtual machines when not in use, for example when you finish working for the day.
* Once you are done with your work free all resources, except when indicated otherwise.

#### Lab convention for naming cloud resources

You will work together with the other students in the same AWS account, and you will see the cloud resources created by other students. To avoid chaos it is therefore important that everybody follows the same convention for naming the resources. This includes public/private key pairs, security groups, EC2 Instances, load balancers, etc.

We will use the following __lab naming convention:__ the name of every cloud resource starts with the group, followed by your last name. You can append more information if you want. For example when student Nicollier in Group F creates an EC2 Instance "master", he names it

    GrF-Nicollier-master

#### A note on regions

By selecting a region an AWS user is able to control where in the world his virtual machines or other cloud resources are located, for example in a particular country or close to the majority of users.

To simplify the administration of your account we impose a limitation in that you can only create resources in one region, _Northern Virginia (us-east-1)_.

You can see the regions when you click on the drop-down menu in the upper right corner. Make sure that you always have __N. Virginia__ selected.

If you select another region and create a resource, you will get the error "You are not authorized to perform this operation".

# Part 1 - CloudFormation

In this part you will explore AWS's proprietary tool, CloudFormation, using the AWS Management Console.

## Task 1.1: Explore predefined templates with the Composer

In this task you will explore ready-made templates developed by AWS for common scenarios and explore the graphical designer.

1. In the AWS Management Console navigate to CloudFormation.

2. Pretend that you want to create a new stack: Click on __Create stack__, then __With new resources (standard)__.

    * __Prerequisite - Prepare template__: Select _Use a sample template_
    * __Select a sample template__: _Simple > LAMP stack_
    
    To view the template in Composer click on __View in Infrastructure Composer__.
   
    Composer has several features displayed on three panes:
    
    * __Canvas__: Located on the central and largest pane, this is where your template resources display as a diagram. Changes that you make here automatically modify the template's code.

    * __Template__: Next to the Canvas button, this is the editor where you can edit your entire template by using JSON or YAML code.
        
    * __List__: Located on the left side of the page, this lists all the template resources that you can add to your template, categorized by the AWS service name. Add resources to the template by dragging them from this pane to the canvas.

    * __Resources__: Next to the List tab, this lists all the currently added resources to the template.

    * __Resources properties__: When you select an item in the canvas and click on *Details*, Composer opens the related code in a right pane. Here you can specify the details of your template, such as resource properties or template parameters. If you edit the code, you must save to update the diagram.

3. Hover over each node on the flow-of-information loop of the icons. Analyze which types of resources were used to create the necessary cloud architecture.
    
   Experiment with adding or removing resources, and analyze how the code changes. You can also look at the other templates.
   
Deliverables: none
   
## Task 1.2: Launch a stack

In this task you will launch a simple stack and observe how CloudFormation creates resources.

Prerequisites:

* You need a [key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html).

1. Choose __Create stack__, then __With new resources (standard)__ and select the sample template __Simple__ > __LAMP Stack__.

    On the screen __Specify stack details__ provide these values:

    * __Stack name__: use the lab naming convention
    * __DBPassword__, __DBRootPassword__, __DBUser__: enter some values
    * __InstanceType__: t2.micro
    * __KeyName__: select your EC2 key pair
    
    On the screen __Configure stack options__ provide these values:
    
    * __Tags__: add a tag with key CloudFormationTest and value true
    
    On the last screen, click on __Create change set__.
    
2. Examine the change set and verify that it conforms to your expectations. In a second window open the EC2 console and verify that no resources have been created yet. Then click on __Execute__ and watch what happens in the EC2 console. The state of the stack should eventually become __CREATE_COMPLETE__.

   * What tags did CloudFormation add to the EC2 Instance? Which tag is used by CloudFormation to uniquely identify the resource?
   * How did CloudFormation name the Security Group and what tags did it add?
   * The template specified an output parameter _WebsiteURL_. What is its value?
   
3. Delete the stack. Verify in the EC2 console that the instance, its EBS volume, and the security group have been deleted.

Deliverables:

* Respond to the questions in points 2 and 3.
* The LAMP Stack template by AWS makes things appear easier than they are. They use an ugly hack to make the selection of the instance type easy for the user of the template, but it makes the template difficult to maintain. Explain the hack.

# Part 2 - Terraform

In this part you will use the provider-independent tool Terraform to provision cloud resources. You will also learn how to use LocalStack, a mock of AWS services, to test Terraform scripts.

#### Prerequisites:

You need to have credentials for AWS, both console and AWS API access.

You need to have installed on your local machine

* Python v.3.7 or later
* AWS Command-Line Interface (CLI)
* Docker (for LocalStack)

## Task 2.1: Create a Terraform project and provision an EC2 instance

In this task you will use Terraform to provision AWS cloud resources.

1. Configure the AWS CLI with the credentials you have received. Terraform will look for the credentials in the same places as the AWS CLI, so it will be configured as well. So that you don't have to overwrite existing credentials, follow the instructions in [Named profiles for the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html) to create several profiles, let's say `cloudsys1` for your existing credentials and `cloudsys2` for the new ones.

   Set the profile to the new credentials:
   
        export AWS_PROFILE=cloudsys2
        
2. Test your access to the AWS API endpoints by executing the following commands with the AWS CLI:

        aws sts get-caller-identity # should display your user ID, account name and ARN
        aws ec2 describe-regions # should display all region names
   
3. Install Terraform on your local machine by following the instructions in the Terraform documentation: [Install Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli). This lab was tested with Terraform v.1.3.3. If you run Windows it is recommended you use a Linux installation to run Terraform (Windows Subsystem for Linux with Debian/Ubuntu is fine).

    Verify that Terraform is installed correctly by running:

        terraform --version

4. Create a Terraform project directory named `terraform`. In it, create a Terraform configuration file `main.tf` with the following content:

        terraform {
          required_providers {
            aws = {
              source  = "hashicorp/aws"
              version = "~> 4.16"
            }
          }
        
          required_version = ">= 1.2.0"
        }
        
        provider "aws" {
          region  = "us-east-1"
        }
        
        resource "aws_instance" "app_server" {
          ami           = "ami-0149b2da6ceec4bb0" # Ubuntu Server 20.04 LTS (HVM), SSD Volume Type
          instance_type = "t2.micro"
        
          tags = {
            Name = "ExampleAppServerInstance"
            Course = "TSM-CloudSys"
            Year = "2023"
            Lab = "Terraform"
            Group = "F" # Put your group letter here
          }
        }
        
   In the `tags` section of the `app_server` resource update the `Group` tag with the letter of your group.

5. You can now initialize Terraform:

        cd terraform
        terraform init
        
   You should see the message:
   
        Terraform has been successfully initialized!
    
   What files were created in the `terraform` directory? Make sure to look also at hidden files and directories (`ls -a`). What are they used for?

6. Verify that your Terraform configuration file is valid:

        terraform validate
        
   You should see the message:
   
        Success! The configuration is valid.

7. Create an execution plan to preview the changes that will be made to your infrastructure and save it locally:

        terraform plan -input=false -out=.terraform/plan.cache

8. If satisfied with your execution plan, let Terraform create the resources by applying it:

        terraform apply -input=false .terraform/plan.cache

    If no errors occur, you have successfully managed to create an EC2 instance using Terraform. Open the AWS Management console to verify that the instance exists. You can also use the AWS CLI to list the instances:
    
        aws ec2 describe-instances
        
9. In the Terraform configuration file modify the AMI of the instance from Ubuntu Server 20.04 LTS to 22.04 LTS (ami-08c40ec9ead489470). Use again `plan` to preview the changes and `apply` to apply them. Copy the output of `plan` to the report and explain it. How does Terraform implement the change of AMI?

10. Destroy the infrastructure you have created:

        terraform destroy
        
    Verify that the instance was terminated by using the AWS Management Console or the AWS CLI.

Deliverables:

* What files did Terraform create when you called `init` and what is their purpose?
* After you modified the AMI, copy the output of the command `plan` into the report and explain it.
* How did Terraform implement the change of AMI?


## Task 2.2: Develop and test Terraform configurations by using LocalStack

In this task you will use LocalStack, a mock of AWS services, to further develop your Terraform configuration file.

To access AWS services programmatically, the AWS CLI and Terraform call the endpoints of the AWS API. To replace AWS services with LocalStack, you need to tell Terraform to use the LocalStack endpoints instead. There are two ways to do this:

* LocalStack provides a thin wrapper script around Terraform called `tflocal` that re-configures Terraform to use the LocalStack endpoints and then calls `terraform`. 
* If you want to use the "real" `terraform` command, you need to configure Terraform manually yourself.

The same idea applies to the `aws` command, for which there is a wrapper script `awslocal`.

1. LocalStack is packaged as a PyPi package. Before installing it, create a virtual environment called `localstack`. If you already have `miniconda` or `virtualenv` use them to create it. Otherwise follow the instructions in [Python Virtual Environments: A Primer](https://realpython.com/python-virtual-environments-a-primer/). 

2. Activate the `localstack` virtual environment. Then install  [localstack](https://docs.localstack.cloud/get-started/#localstack-cli). We will also need the package `terraform-local` which provides a wrapper script around `terraform` called `tflocal`.
    ```shell
    pip install localstack
    pip install terraform-local
    ```

3. The LocalStack mock services run inside a Docker container. Launch the container with

        localstack start -d
        
    You can later stop it with `localstack stop`.
    
4. Use the AWS CLI to explore the LocalStack mock services. The wrapper script `awslocal` has the downside that it only works for the v.1 AWS CLI. So we will instead use the real `aws` command and configure it to talk to the LocalStack endpoint.

   The AWS CLI needs to know the Access Key ID/Secret Access Key and the default region. These can be set in environment variables. LocalStack does not need the keys, so in principle they can be set to anything, but to make pre-signed URLs for S3 buckets work, always use the value `test` for both keys:
   
        export AWS_ACCESS_KEY_ID="test"
        export AWS_SECRET_ACCESS_KEY="test"
        export AWS_DEFAULT_REGION="us-east-1"
       
   To redirect the AWS CLI to the LocalStack endpoint instead of the AWS endpoint, use the `--endpoint` option:
   
        aws --endpoint-url=http://localhost:4566 ec2 describe-regions
       
   Unfortunately, the endpoint URL cannot be set permanently via an environment variable. A workaround is to use a shell alias:
   
        alias aws='aws --endpoint-url=http://localhost:4566'
       
   To remove the alias later, use
   
        unalias aws
        
   Repeat the AWS CLI commands from task 1 to query the caller identity and to list the available regions.
    
4. Repeat the Terraform commands of task 1, but replace the `terraform` command with `tflocal`, i.e.

        tflocal init
        tflocal plan
        tflocal apply
        
    Verify that everything works the same as before by using the AWS CLI. Some useful commands:
    
        aws ec2 describe-instances
        aws ec2 describe-volumes
        aws ec2 describe-security-groups
        
    To reduce typing you may want to enable autocomplete for the AWS CLI: [Command completion](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-completion.html).
    
Deliverables:

* When switching from AWS services to LocalStack, why do you need to do an `init` again? Think about the role of the Terraform state file.
* Show in the report how you verified that the instance was "created" in LocalStack.

## Task 2.3: Add a security group

In this task you will add to the EC2 instance a security group and test your configuration using LocalStack.

1. Modify the Terraform configuration file to add a security group. The security group shall allow incoming traffic for SSH and HTTPS from any IP source address, and allow any outgoing traffic.

2. Test your configuration by going through `plan`, `apply` cycles.

3. Add [output values](https://developer.hashicorp.com/terraform/language/values/outputs) to your Terraform configuration file. Output values allow you to see at a glance certain important properties of the created resources. Add output values that show:

    * The public IP address of the instance
    * The instance ID
    
   When you run `apply`, the output values should appear at the end of the command's output like this:
   
            Apply complete! Resources: x added, x changed, x destroyed.
            
            Outputs:
    
            instance_public_ip = "xxx.xxx.xxx.xxx"
            instance_id = "i-xxxxxxxxxxxxxxxx"

4. Once everything works as planned, run `tflocal destroy` to destroy the mock resources and switch back to AWS services. To switch back you need to unset the `AWS_*` environment variables and the `aws` alias:

        unset AWS_ACCESS_KEY_ID
        unset AWS_SECRET_ACCESS_KEY
        unset AWS_DEFAULT_REGION
        unalias aws
        
  Tip: You can save these instructions in a script, say `switch2aws`, and execute the script with `source switch2aws`.

   Provision your configuration on AWS and test that everything is OK. Finally, destroy the resources.

Deliverables:

* Copy your Terraform configuration file into the report.
* Copy the output of `apply` into the report.
* Copy a screenshot of the AWS Management Console showing the details of the security group into the report.

