
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

# 1. Create a clean project folder
mkdir lambda_release && cd lambda_release

# 2. Target installation directly into this directory
pip3 install pymysql -t .

# 3. Create your script file
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
