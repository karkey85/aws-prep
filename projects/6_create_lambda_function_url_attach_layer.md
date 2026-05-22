The updated code modified specifically for a Lambda Function URL architecture.

```
"""
UI Specifications: Designed for pasting directly into the AWS Lambda Console GUI.
Issue & Fix Log:
  - 2026-05-22: Removed API Gateway formatting assumptions; adapted response keys for AWS Lambda Function URL.
  - 2026-05-22: Added custom JSON encoder subclass to natively handle MySQL DateTime/Decimal fields without manual row iterations.
"""

from datetime import date, datetime
from decimal import Decimal
import json
import os
import pymysql

# Environment variable fallback configuration
DB_HOST = os.environ.get("DB_HOST", "your-rds-mysql-endpoint.amazonaws.com")
DB_USER = os.environ.get("DB_USER", "admin")
DB_PASS = os.environ.get("DB_PASS", "your_password")
DB_NAME = os.environ.get("DB_NAME", "exam_practice")


class MySQLJSONEncoder(json.JSONEncoder):
    """Custom JSON encoder to safely handle native MySQL types during serialization."""

    def default(self, obj):
        if isinstance(obj, (datetime, date)):
            return obj.strftime("%Y-%m-%d %H:%M:%S")
        if isinstance(obj, Decimal):
            return float(obj)
        return super(MySQLJSONEncoder, self).default(obj)


def lambda_handler(event, context):
    """Main entry point for AWS Lambda Function URL execution."""
    connection = None
    try:
        # Establish a persistent socket connection to the raw MySQL database
        connection = pymysql.connect(
            host=DB_HOST,
            user=DB_USER,
            password=DB_PASS,
            database=DB_NAME,
            port=3306,
            cursorclass=pymysql.cursors.DictCursor,
            connect_timeout=5,  # Prevent hanging during cold-starts or network blocks
        )

        with connection.cursor() as cursor:
            sql = "SELECT id, first_name, last_name, email, created_at FROM users"
            cursor.execute(sql)
            results = cursor.fetchall()

        # Format returning payload cleanly for a Function URL endpoint response
        return {
            "statusCode": 200,
            "headers": {
                "Content-Type": "application/json",
                "Access-Control-Allow-Origin": "*",  # Helpful for direct Function URL consumption via frontend apps
            },
            "body": json.dumps(
                {"total_records": len(results), "data": results},
                cls=MySQLJSONEncoder,
            ),
        }

    except Exception as e:
        # Return structured execution failure details
        return {
            "statusCode": 500,
            "headers": {"Content-Type": "application/json"},
            "body": json.dumps(
                {
                    "error": "Database connection or execution failed.",
                    "details": str(e),
                }
            ),
        }

    finally:
        if connection and connection.open:
            connection.close()
```


# Lambda Function URL (Fastest & Simplest)
If you just need a dedicated HTTPS endpoint for a single function without any extra bells and whistles (like rate limiting, custom authorizers, or complex routing),
Function URLs are ideal. They are free and support up to 15-minute timeouts.  
Console Setup
Open the AWS Lambda Console and click on your function.
Go to the Configuration tab,
and select Function URL from the left sidebar.
Click Create function URL.
Set Auth type to NONE (this allows public unauthenticated browser access).
Crucial for Browsers: Expand the Additional settings section and check Configure cross-origin resource sharing (CORS)
if your frontend is hosted on a different domain. You can default the allowed origins to * for testing, or lock it down to your frontend domain. 
Click Save. 
Copy the generated URL from the top of the section.
