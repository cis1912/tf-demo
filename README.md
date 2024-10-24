
> In this in-class exercise, you will create an S3 bucket and upload a file to it and alter its permissions to toggle public/private access using Terraform. This exercise consists of 3 parts: Setting up S3 manually, Setting up S3 using Terraform, and Altering S3 permissions using Terraform.

# Part I: Setting up S3 manually

## Step 1: Create an S3 Bucket

You will need an AWS access key and secret key to create an S3 bucket. We will provide one you can use during class. Please do not share this key with anyone. Once you have the credentials, place it in `~/.aws/credentials` file.

Go to the AWS Management Console and search for S3. Click on the S3 service and create a new bucket. Your bucket name should be `org.cis1912.<pennkey>`. For example, if your PennKey is `esinx`, your bucket name should be `org.cis1912.esinx`.

Disable "Block all public access" and acknowledge that the bucket will be public. You want your bucket to be accessible to the public so that we can view the file you upload!

Once your done creating the bucket, you will need to add a new bucket policy. The policy should look like this:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::org.cis1912.<pennkey>/*"
        }
    ]
}
```

This will allow anyone to access the objects in your bucket.

## Step 2: Upload a file to the bucket

Upload a picture of yourself to the bucket. You can use any picture you like! Place it under the key `profile.jpg`.

Now, try navigating to `https://s3.us-east-1.amazonaws.com/org.pennlabs.<pennkey>/profile.jpg`. You should be able to see the picture you uploaded!

## Step 3: Alter the permissions of the file

Now you will try to make the bucket private by modifying the bucket policy and altering the bucket properties. Remove the bucket policy you added in Step 1 and try to access the file again. You should not be able to access the file anymore.

For additional security, alter the bucket properties to block all public access.

# Part II: Setting up S3 using Terraform

## Step 0: Install Terraform

Follow steps on [Terraform's official website](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) to install Terraform on your machine.

## Step 1: Setup Terraform with AWS

Create a new directory and create a new file called `main.tf`. This file will contain the Terraform code to create an S3 bucket.

```
provider "aws" {
  region = "us-east-1"
}
```

This code will tell Terraform to use the AWS provider in the `us-east-1` region.

Now, run `terraform init` to initialize Terraform. This will install all the relevant provider modules for AWS.

## Step 2: Create an S3 bucket

Add the following code to `main.tf`:

```
resource "aws_s3_bucket" "cis1912_bucket" {
  bucket = "org.cis1912.<pennkey>"
  acl    = "public-read"
}
```

This code will create an S3 bucket with the name `org.cis1912.<pennkey>` and set the bucket to be public.

While the bucket has been created, it is not accessible to the public yet. You will need to add a bucket policy to allow public access.

```
resource "aws_s3_bucket_policy" "cis1912_bucket_policy" {
  bucket = aws_s3_bucket.cis1912_bucket.bucket
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = "*"
        Action    = "s3:GetObject"
        Resource  = aws_s3_bucket.cis1912_bucket.arn
      }
    ]
  })
}
```

This code will add a bucket policy to the bucket you created in the previous step. The policy will allow anyone to access the objects in the bucket.

Run `terraform apply` to create the bucket and the bucket policy.

## Step 3: Upload a file to the bucket

But how would you upload a file to the bucket? You can use the `aws_s3_bucket_object` resource to upload a file to the bucket.

```
resource "aws_s3_bucket_object" "profile_picture" {
  bucket = aws_s3_bucket.cis1912_bucket.bucket
  key    = "profile.jpg"
  source = "path/to/your/picture.jpg"
}
```

This code will upload a file to the bucket you created in the previous step. Replace `path/to/your/picture.jpg` with the path to the picture you want to upload.

Run `terraform apply` to upload the file to the bucket.

## Step 4: Getting the object URL

You can get the URL of the object you uploaded using the `aws_s3_bucket_object` resource.

```
output "object_url" {
  value = aws_s3_bucket_object.profile_picture.id
}
```

Run `terraform apply` to get the URL of the object you uploaded. You should be able to access the object using the URL.

# Part III: Altering S3 permissions using Terraform

## Step 1: Make the bucket private

Now that you have created the bucket and uploaded a file to it, you will try to make the bucket private by modifying the bucket policy and altering the bucket properties, all within Terraform.

We will alter the bucket properties to block all public access.

```
resource "aws_s3_bucket_public_access_block" "cis1912_bucket_block" {
  bucket = aws_s3_bucket.cis1912_bucket.bucket
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

And delete all the bucket policies you added in the previous steps.

Run `terraform apply` to make the bucket private. You should not be able to access the file anymore.