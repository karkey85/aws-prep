I am using laptop which is not allowing me to download the zip file directly to my laptop due to firewall issues

So i am creating a aws lambda layer for pymysql which will be used by my lambda function.

# Build Your Own Layer Using CloudShell (No Local Downloads)

## 1. Generate the Zip Bundle in CloudShell
Click the CloudShell icon (>_) in the top navigation bar of the AWS console. Once the terminal opens, paste these commands:
### 1. AWS Lambda looks strictly for a folder named "python" inside the zip file
mkdir -p python_layer/python && cd python_layer/python

### 2. Target installation into the folder matching Lambda's environment paths
pip3 install pymysql -t .

### 3. Zip the bundle up from the parent directory
cd ..
zip -r pymysql_layer.zip python/

### 4. Publish this directly as a reusable layer in your account
aws lambda publish-layer-version \
    --layer-name pymysql-layer \
    --description "Native PyMySQL database driver" \
    --zip-file fileb://pymysql_layer.zip \
    --compatible-runtimes python3.10 python3.11 python3.12


output from my cloud shell
```
~ $ 
~ $ 
~ $ 
~ $ mkdir -p python_layer/python && cd python_layer/python
python $ 
python $ pip3 install pymysql -t .
Collecting pymysql
  Downloading pymysql-1.2.0-py3-none-any.whl.metadata (4.3 kB)
Downloading pymysql-1.2.0-py3-none-any.whl (45 kB)
Installing collected packages: pymysql
Successfully installed pymysql-1.2.0

[notice] A new release of pip is available: 26.0.1 -> 26.1.1
[notice] To update, run: pip install --upgrade pip
python $ cd ..
python_layer $ zip -r pymysql_layer.zip python/
  adding: python/ (stored 0%)
  adding: python/pymysql-1.2.0.dist-info/ (stored 0%)
  adding: python/pymysql-1.2.0.dist-info/top_level.txt (stored 0%)
  adding: python/pymysql-1.2.0.dist-info/REQUESTED (stored 0%)
  adding: python/pymysql-1.2.0.dist-info/licenses/ (stored 0%)
  adding: python/pymysql-1.2.0.dist-info/licenses/LICENSE (deflated 41%)
  adding: python/pymysql-1.2.0.dist-info/METADATA (deflated 56%)
  adding: python/pymysql-1.2.0.dist-info/INSTALLER (stored 0%)
  adding: python/pymysql-1.2.0.dist-info/WHEEL (stored 0%)
  adding: python/pymysql-1.2.0.dist-info/RECORD (deflated 57%)
  adding: python/pymysql/ (stored 0%)
  adding: python/pymysql/__pycache__/ (stored 0%)
  adding: python/pymysql/__pycache__/protocol.cpython-313.pyc (deflated 59%)
  adding: python/pymysql/__pycache__/_auth.cpython-313.pyc (deflated 54%)
  adding: python/pymysql/__pycache__/connections.cpython-313.pyc (deflated 57%)
  adding: python/pymysql/__pycache__/optionfile.cpython-313.pyc (deflated 40%)
  adding: python/pymysql/__pycache__/cursors.cpython-313.pyc (deflated 56%)
  adding: python/pymysql/__pycache__/err.cpython-313.pyc (deflated 56%)
  adding: python/pymysql/__pycache__/times.cpython-313.pyc (deflated 43%)
  adding: python/pymysql/__pycache__/__init__.cpython-313.pyc (deflated 45%)
  adding: python/pymysql/__pycache__/charset.cpython-313.pyc (deflated 75%)
  adding: python/pymysql/__pycache__/converters.cpython-313.pyc (deflated 57%)
  adding: python/pymysql/times.py (deflated 58%)
  adding: python/pymysql/_auth.py (deflated 69%)
  adding: python/pymysql/err.py (deflated 61%)
  adding: python/pymysql/__init__.py (deflated 60%)
  adding: python/pymysql/charset.py (deflated 82%)
  adding: python/pymysql/protocol.py (deflated 72%)
  adding: python/pymysql/converters.py (deflated 73%)
  adding: python/pymysql/constants/ (stored 0%)
  adding: python/pymysql/constants/__pycache__/ (stored 0%)
  adding: python/pymysql/constants/__pycache__/ER.cpython-313.pyc (deflated 55%)
  adding: python/pymysql/constants/__pycache__/CR.cpython-313.pyc (deflated 50%)
  adding: python/pymysql/constants/__pycache__/CLIENT.cpython-313.pyc (deflated 28%)
  adding: python/pymysql/constants/__pycache__/FIELD_TYPE.cpython-313.pyc (deflated 31%)
  adding: python/pymysql/constants/__pycache__/FLAG.cpython-313.pyc (deflated 21%)
  adding: python/pymysql/constants/__pycache__/SERVER_STATUS.cpython-313.pyc (deflated 32%)
  adding: python/pymysql/constants/__pycache__/__init__.cpython-313.pyc (deflated 19%)
  adding: python/pymysql/constants/__pycache__/COMMAND.cpython-313.pyc (deflated 39%)
  adding: python/pymysql/constants/__init__.py (stored 0%)
  adding: python/pymysql/constants/COMMAND.py (deflated 57%)
  adding: python/pymysql/constants/CR.py (deflated 63%)
  adding: python/pymysql/constants/SERVER_STATUS.py (deflated 50%)
  adding: python/pymysql/constants/CLIENT.py (deflated 51%)
  adding: python/pymysql/constants/FIELD_TYPE.py (deflated 45%)
  adding: python/pymysql/constants/FLAG.py (deflated 31%)
  adding: python/pymysql/constants/ER.py (deflated 63%)
  adding: python/pymysql/connections.py (deflated 75%)
  adding: python/pymysql/cursors.py (deflated 72%)
  adding: python/pymysql/optionfile.py (deflated 58%)
python_layer $ aws lambda publish-layer-version \
>     --layer-name pymysql-layer \
>     --description "Native PyMySQL database driver" \
>     --zip-file fileb://pymysql_layer.zip \
>     --compatible-runtimes python3.10 python3.11 python3.12
    "LayerVersionArn": "arn:aws:lambda:ap-south-1:785716434644:layer:pymysql-layer:1",
    "Description": "Native PyMySQL database driver",
    "CreatedDate": "2026-05-22T14:28:52.485+0000",
    "Version": 1,
    "CompatibleRuntimes": [
        "python3.10",
        "python3.11",
        "python3.12"
    ]
}
python_layer $ 
```


# 2. Attach Your Custom Layer to Your Function
Once that script finishes executing in CloudShell, it is officially saved in your private account catalog:

Go back to your Lambda Function window and scroll down to Layers -> Add a layer.
Select Custom layers as the layer source.
Choose pymysql-layer from the dropdown menu.
Select Version 1 and click Add.
