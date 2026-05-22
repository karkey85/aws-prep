
Switching from an EC2 instance to AWS Lambda transitions the architecture into a completely serverless layout. 
This is a highly recommended design pattern because it eliminates server management, auto-scales instantly, and only incurs costs while code is executing.

# The Architectural Blueprint
[ API Gateway ] ---> [ AWS Lambda ] ---> [ ENI / Subnet ] ---> [ RDS Database ]
                     (Needs PyMySQL)     (Inside your VPC)       (SG-Database)

## Lambda inside the VPC: 
By default, Lambda runs in an isolated AWS-managed network.
It cannot touch your private RDS instance unless you explicitly configure the Lambda function to run inside your Default VPC.

## Security Group Pass: 
Lambda will automatically receive a dynamic Elastic Network Interface (ENI) inside your subnet. 
We will attach a security group to Lambda so your database trusts it.

# Step 1: Package Your Code and Dependencies

## 1. Create a clean project folder
mkdir lambda_release && cd lambda_release

## 2. Target installation directly into this directory
pip3 install pymysql -t .

## 3. Create your script file
nano lambda_function.py

Paste the following Lambda handler code into lambda_function.py:

```
import json
import os
import pymysql

# Best Practice: Use Environment Variables instead of hardcoding strings
DB_HOST = os.environ.get("DB_HOST", "your-rds-endpoint.amazonaws.com")
DB_USER = os.environ.get("DB_USER", "admin")
DB_PASS = os.environ.get("DB_PASS", "your_password")
DB_NAME = os.environ.get("DB_NAME", "exam_practice")


def lambda_handler(event, context):
    """Main execution point targeted by AWS Lambda."""
    connection = None
    try:
        # Establish the database connection socket
        connection = pymysql.connect(
            host=DB_HOST,
            user=DB_USER,
            password=DB_PASS,
            database=DB_NAME,
            port=3306,
            cursorclass=pymysql.cursors.DictCursor,
        )

        with connection.cursor() as cursor:
            sql = "SELECT id, first_name, last_name, email, created_at FROM users"
            cursor.execute(sql)
            results = cursor.fetchall()

            # Clean timestamp formats for JSON serialization compatibility
            for row in results:
                if "created_at" in row and row["created_at"]:
                    row["created_at"] = row["created_at"].strftime(
                        "%Y-%m-%d %H:%M:%S"
                    )

        # Return structured response formatted for API Gateway integrations
        return {
            "statusCode": 200,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps({"total_records": len(results), "data": results}),
        }

    except Exception as e:
        return {
            "statusCode": 500,
            "body": json.dumps({"error": f"Database interaction failed: {str(e)}"}),
        }

    finally:
        if connection:
            connection.close()
```

Now, compress the directory into a zip deployment archive:

zip -r ../lambda_deployment.zip .

# Step 2: Configure Permissions & Security Groups

Before creating the function, we need a security group for Lambda so the database can recognize it.

1. Go to the EC2 Console -> Security Groups -> Create security group.
Name: SG-Lambda-Function
VPC: Select your Default VPC.
Inbound Rules: Leave completely empty (nothing needs to initiate inbound requests directly to Lambda).

2. Now, go to your RDS Security Group (SG-Database).
Add a new Inbound Rule:
Type: MySQL/Aurora (3306)
Source: Select SG-Lambda-Function (Security Group Referencing).
Save the rule.


# Step 3: Create the Lambda Function
Open the AWS Lambda Console and click Create function (Author from scratch).

Function name: fetch-rds-users
Runtime: Python 3.12 (or matching your deployment build version).
Advanced settings:
Check the box for Enable VPC.
VPC: Select your Default VPC.
Subnets: Choose at least 2 or 3 availability zone subnets (for high availability).
Security groups: Choose SG-Lambda-Function.
Click Create function.

Note: AWS will auto-assign the standard AWSLambdaVPCAccessExecutionRole managed policy to your function's Execution Role, giving it rights to manage the elastic interfaces inside your VPC network.

# Step 4: Upload Code and Set Environment Variables
Inside your new Lambda function dashboard, navigate to the Code tab.
**Click Upload from -> .zip file and select your lambda_deployment.zip.**
Switch over to the Configuration tab -> Environment variables -> Edit.

Map out your explicit strings:
DB_HOST = Your RDS connection endpoint
DB_USER = admin
DB_PASS = Your database password
DB_NAME = exam_practice
Click Save.

# Step 5: Test Execution
Click on the Test tab in the Lambda console.
Create a new dummy test event (the payload properties don't matter since our function ignores inputs and grabs all records). Give it a name like TestClick.
Click Test.

Your execution logs will output a successful 200 OK return block containing your JSON database records directly back inside the AWS console layout!
