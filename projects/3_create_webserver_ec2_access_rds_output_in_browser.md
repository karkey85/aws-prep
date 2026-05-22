
To achieve this, we will build a lightweight Python web service using FastAPI (an incredibly fast, modern web framework)
and PyMySQL to connect to your RDS instance.

When running this inside an EC2 instance, you want your web app to listen on an open port (like 8000) so you can hit it from your browser or via curl.

# Step 1: Install Dependencies on Your EC2 Instance
SSH back into your EC2 instance. We need to install fastapi for the web layer, uvicorn to act as the web server, and pymysql to speak SQL to your RDS database.

sudo dnf install python3-pip -y
pip3 install fastapi uvicorn pymysql

# Step 2: Write the Python Application
Create a new file named app.py inside your EC2 workspace using nano app.py or your preferred text editor.
Paste the following complete, production-ready code structure:

```
import pymysql
from fastapi import FastAPI, HTTPException

app = FastAPI(title="Exam Prep DB Reader API")

# --- DATABASE CONFIGURATION ---
# Replace these strings with your actual architecture specifics
DB_HOST = "your-rds-endpoint-here.amazonaws.com"
DB_USER = "your_master_user"
DB_PASS = "your_master_password"
DB_NAME = "exam_practice"


def get_db_connection():
    """Helper function to establish a fresh connection to the RDS database."""
    try:
        return pymysql.connect(
            host=DB_HOST,
            user=DB_USER,
            password=DB_PASS,
            database=DB_NAME,
            port=3306,
            cursorclass=pymysql.cursors.DictCursor,  # Returns rows as neat Python dicts
        )
    except Exception as e:
        print(f"Database connection failed: {e}")
        raise HTTPException(
            status_code=500, detail="Could not connect to the backend database"
        )


@app.get("/")
def read_root():
    return {
        "status": "online",
        "message": "Welcome to the EC2 Data API. Head to /users to fetch rows.",
    }


@app.get("/users")
def get_all_users():
    """Fetches all entries from our users table and prints them as JSON."""
    connection = get_db_connection()
    try:
        with connection.cursor() as cursor:
            # Execute standard SQL statement
            sql = "SELECT id, first_name, last_name, email, created_at FROM users"
            cursor.execute(sql)

            # Pull all query matches
            results = cursor.fetchall()

            # Format timestamp objects cleanly to strings for standard JSON output
            for row in results:
                if "created_at" in row and row["created_at"]:
                    row["created_at"] = row["created_at"].strftime(
                        "%Y-%m-%d %H:%M:%S"
                    )

            return {"total_records": len(results), "data": results}
    finally:
        # Crucial step: Always close your network sockets!
        connection.close()
```

# Step 3: Open the Port in Your Security Group

Amazon EC2 Console -> Security Groups->Edit inbound rule -> add 8000 port custom tcp , Source: Anywhere-IPv4 (0.0.0.0/0) or My IP for better security.

# Step 4: Fire Up the Web Server
Back inside your EC2 terminal session, run the Uvicorn deployment engine. We bind it to 0.0.0.0 so it listens on all network interfaces rather than just local loops.

uvicorn app:app --host 0.0.0.0 --port 8000

# Step 5: Test Your New API Endpoint
Open a web browser on your personal computer or open a second terminal layout and execute requests.

Via Browser: Navigate to http://YOUR-EC2-PUBLIC-IP:8000/users


# Issue 1: couldnt connect to backend database error.

```
[ec2-user@ip-172-31-44-158 ~]$ curl http://0.0.0.0:8000/users
{"detail":"Could not connect to the backend database"}[ec2-user@ip-172-31-44-158 ~]$
```

**reason:**

[ec2-user@ip-172-31-44-158 ~]$ uvicorn app:app --host 0.0.0.0 --port 8000
INFO:     Started server process [27122]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
Database connection failed: (2003, "Can't connect to MySQL server on '**your-rds-endpoint-here.amazonaws.com**' ([Errno -2] Name or service not known)")
INFO:     127.0.0.1:48722 - "GET /users HTTP/1.1" 500 Internal Server Error
^CINFO:     Shutting down
INFO:     Waiting for application shutdown.
INFO:     Application shutdown complete.
INFO:     Finished server process [27122]

**FIxed:**

```
[ec2-user@ip-172-31-44-158 ~]$ curl http://0.0.0.0:8000/users
{"total_records":3,"data":[{"id":1,"first_name":"Karthik","last_name":"Dev","email":"karthik@example.com","created_at":"2026-05-22 12:21:21"},{"id":2,"first_name":"Alice","last_name":"Smith","email":"alice@example.com","created_at":"2026-05-22 12:21:57"},{"id":3,"first_name":"Bob","last_name":"Jones","email":"bob@example.com","created_at":"2026-05-22 12:21:57"}]}[ec2-user@ip-172-31-44-158 ~]$
```

[ec2-user@ip-172-31-44-158 ~]$ uvicorn app:app --host 0.0.0.0 --port 8000
INFO:     Started server process [27594]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
INFO:     127.0.0.1:60096 - "GET /users HTTP/1.1" 200 OK
