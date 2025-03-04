# Day 1 : Streamlined Guide: Using Claude as AI Assistant to Terraform
## Step 1: Use Claude to Generate Terraform Code
- Start a conversation with Claude.
- Ask Claude to create Terraform code for an S3 bucket. Use a prompt like:
  "Act as a Terraform expert and AWS Engineer provide Terraform code to create an S3 bucket in AWS with a unique name."
- Claude should generate code similar to this:

      # Configure the AWS Provider
      provider "aws" {
        region = "us-west-2"  # Choose your preferred region
      }
      
      # Generate a random string to ensure unique bucket name
      resource "random_pet" "bucket_name" {
        length    = 2
        separator = "-"
      }
      
      # Create the S3 Bucket
      resource "aws_s3_bucket" "example_bucket" {
        bucket = "my-unique-bucket-${random_pet.bucket_name.id}"
      
        # Enable versioning for additional data protection
        versioning {
          enabled = true
        }
      
        # Configure server-side encryption
        server_side_encryption_configuration {
          rule {
            apply_server_side_encryption_by_default {
              sse_algorithm = "AES256"
            }
          }
        }
      
        # Add lifecycle rule to manage bucket objects (optional)
        lifecycle_rule {
          enabled = true
      
          expiration {
            days = 90  # Automatically delete objects after 90 days
          }
        }
      
        # Bucket tags for better resource management
        tags = {
          Name        = "Example Bucket"
          Environment = "Development"
          ManagedBy   = "Terraform"
        }
      }
      
      # Block public access to the bucket
      resource "aws_s3_bucket_public_access_block" "bucket_public_access" {
        bucket = aws_s3_bucket.example_bucket.id
      
        block_public_acls       = true
        block_public_policy     = true
        ignore_public_acls      = true
        restrict_public_buckets = true
      }


- Save this code for use in Step 5.

## Step 2: Create IAM Role for EC2
- Log in to the AWS Management Console.
  - ![Image1](https://github.com/user-attachments/assets/a9a3af8f-c123-41b0-a3b4-a85073310e96)

- Navigate to the IAM dashboard.
  - ![Image2](https://github.com/user-attachments/assets/7856c19f-bcc2-4df3-806d-4dcdbaeec5ce)
    
- Click "Roles" in the left sidebar, then "Create role".
  - ![Image3](https://github.com/user-attachments/assets/6edc0e41-416b-4d3f-a080-3520d09b9281)

- Choose "AWS service" as the trusted entity type and "EC2" as the use case.
  - ![Image4](https://github.com/user-attachments/assets/99c0fbdb-f0fb-44ad-8805-7c2620992630)

- Click Next on the down left orange button and then Search for and attach the "AdministratorAccess" policy.
  - ![Image5](https://github.com/user-attachments/assets/ac940206-6fca-47cd-bef2-4dc1c01ff31d)

**Note**: In a production environment, use a more restricted policy.
- Name the role "EC2Admin" and provide a description.
  - ![Image6](https://github.com/user-attachments/assets/ff38f41f-8e32-4efd-a2e1-76c4a596392e)

- Review and create the role.

## Step 3: Launch EC2 Instance
- Go to the EC2 dashboard in the AWS Management Console.
- Click "Launch Instance".
  - ![Image7](https://github.com/user-attachments/assets/2ae80801-6f48-42ac-b5e9-1c4165ec79b4)
- Choose an Amazon Linux 2 AMI.
  - ![Image8](https://github.com/user-attachments/assets/e2f62e9f-7793-4125-9855-548880b7b0c6)

- Select a t2.micro instance type.
- Configure instance details:
    - Network: Default VPC
    - Subnet: Any available
    - Auto-assign Public IP: Enable
    - IAM role: Select "EC2Admin"
  - ![Image9](https://github.com/user-attachments/assets/b197e9d7-e51f-479a-83b9-5859de4b02d7)

- Keep default storage settings.
- Add a tag: Key="Name", Value="workstation".
- Create a security group allowing SSH access from EC2 Connect IP.
- Review and launch, selecting or creating a key pair.
  - ![Image10](https://github.com/user-attachments/assets/b6c12dff-4fd7-4500-b1fc-0ae6c9d672a7)
  - ![Image11](https://github.com/user-attachments/assets/aff27f60-9bcf-462e-b4a8-db341ef28761)
  -  ![Image13](https://github.com/user-attachments/assets/9f92e086-ea91-405e-89c1-e4e2265d7ae4)




































































