Create your python function: .
Note: For this example we are naming the file: lambda.py,
which is important to sync up with the terraform setup:

import json

def handler(event, context):
    print("Received event: " + json.dumps(event, indent=2))




-----------------------------------
Zip the file up:

zip lambda lambda.py
-------------------------------------------
Define your terraform setup:

In a file named anything ending with .tf*
For Example: main.tf

provider "aws" {
  region = "us-east-1"
  access_key = ""#key here
  secret_key = ""#key here

}
variable "function_name" {
  default = "minimal_lambda_function"
}

variable "handler" {
  default = "lambda.handler"
}

variable "runtime" {
  default = "python3.6"
}

resource "aws_lambda_function" "lambda_function" {
  role             = "${aws_iam_role.lambda_exec_role.arn}"
  handler          = "${var.handler}"
  runtime          = "${var.runtime}"
  filename         = "lambda.zip"
  function_name    = "${var.function_name}"
  source_code_hash = "${base64sha256(file("lambda.zip"))}"
}

resource "aws_iam_role" "lambda_exec_role" {
  name        = "lambda_exec"
  path        = "/"
  description = "Allows Lambda Function to call AWS services on your behalf."

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
}
----------------------------------
Create your lambda with your function and your IAM role:

terraform apply
