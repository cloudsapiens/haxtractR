
[![logo](https://github.com/cloudsapiens/haxtractR/blob/main/imgs/logo.PNG)](https://github.com/cloudsapiens/haxtractR/blob/main/imgs/logo.PNG) 

```sh
AWS Lambda function written in Python to extract data from SAP HANA database storing credentials in AWS Secrets Manager
```

This project is to provide you a python program utilizing pyHDB library to extract data from SAP HANA database, while storing credentials in AWS Secrets Manager.

AWS services used for this solution:
  - AWS Lambda
  - AWS Secrets Manager

# Architecture
[![architecture](https://github.com/cloudsapiens/haxtractR/blob/main/imgs/architecture.PNG)](https://github.com/cloudsapiens/haxtractR/blob/main/imgs/architecture.PNG) 

# Deployment

### Step 1: Setup Secrets Manager

Open AWS Secrets Manager in the AWS Management Console

On the first screen select ```Other type of secrets``` and enter secret key/value combinations like on the following figure:

![secretsmanager](https://github.com/cloudsapiens/HANAssistant/blob/main/imgs/secretsmanager.PNG)

```Note: change the values for DB_USER and DB_PASSWORD for your credentials (SYSTEMDB or Tenant DB)```

On the second screen enter Secret name ```hana-credentials``` and enter a description.

On the third screen leave the default settings just like on the following figure:

### Step 2: Setup Lambda function

Open Lambda service in the AWS Management Console and create a new Function with Runtim ```Python 3.8```:

Clone this repository 
```sh 
git clone https://github.com/cloudsapiens/haxtractR.git
```

Upload haxtractr_lambda.zip as shown on the following figure (in the cloned repository -> ```Lambda``` folder:

![lambda2](https://github.com/cloudsapiens/HANAssistant/blob/main/imgs/lambda2.PNG)

Create two Environment Variables with the following keys/values:

```DB_HOST``` IP of SAP HANA DB (if private than Lambda should be connected to a VPC)

```DB_PORT``` Port of SYSTEMDB or Tenant DB

```SQL_STATEMENT``` SQL statement to be executed on SAP HANA database

```
Note: 

The port number of the system database are fixed: 3<instance>01 (internal), 3<instance>13 (SQL), and 3<instance>14 (HTTP via XS classic server).

Example 1:

You install a new SAP HANA system without creating an initial tenant, by running the SAP HANA database lifecycle manager (HDBLCM) with the create_initial_tenant flag set to off. Then, you create three tenant databases. Each of these tenant databases is automatically assigned port numbers for the following connection types:

Internal communication
SQL
HTTP (This is the port of the XS classic server embedded in the index server.)
The first tenant database is assigned port numbers 3<instance>40—42, the second ports 3<instance>43—45, and the third 3<instance>46—48.

Example 2:

You install a new SAP HANA system without creating an initial tenant, by running the SAP HANA database lifecycle manager (HDBLCM) with the create_initial_tenant flag set to off. Then, you create a tenant database. The same port numbers as above are assigned: 3<instance>40 (internal communication), 3<instance>41 (SQL), and 3<instance>42 (HTTP via XS classic server). Next, you add a separate xsengine service to the first database. This service is automatically assigned the next three available port numbers: 3<instance>43—45. Finally, you create a second tenant database. This tenant database is automatically assigned the next three available port numbers: 3<instance>46—48.

Example 3:

You convert a single-container system to a tenant database system. This results in the automatic creation of one tenant database. This tenant database has the same port numbers as the original single-container system: 3<instance>03 (internal communication), 3<instance>15 (SQL), 3<instance>08 (HTTP via XS classic server). Then, you add a second indexserver to the tenant database. It is automatically assigned port numbers 3<instance>40—42. Finally, you create a second tenant database. It is automatically assigned ports the next three available port numbers: 3<instance>43—45.

You can determine the ports used by a particular tenant database by querying the M_SERVICES system view, either from the tenant database itself or from the system database.

From the tenant database: SELECT SERVICE_NAME, PORT, SQL_PORT, (PORT + 2) HTTP_PORT FROM SYS.M_SERVICES WHERE ((SERVICE_NAME='indexserver' and COORDINATOR_TYPE= 'MASTER') or (SERVICE_NAME='xsengine'))
From the system database: SELECT DATABASE_NAME, SERVICE_NAME, PORT, SQL_PORT, (PORT + 2) HTTP_PORT FROM SYS_DATABASES.M_SERVICES WHERE DATABASE_NAME='<DBNAME>' and ((SERVICE_NAME='indexserver' and COORDINATOR_TYPE= 'MASTER') or (SERVICE_NAME='xsengine'))
```
Source: 
[SAP Help](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/440f6efe693d4b82ade2d8b182eb1efb.html)

Change Timout to 30 seconds as shown on the following screenshot:

![lambda4](https://github.com/cloudsapiens/HANAssistant/blob/main/imgs/lambda4.PNG)

Finally click on ```Deploy``` to save your function.

### Step 2: Setup IAM policies for Lambda

In the Lambda console, select ```Permissions``` and navigate to IAM console by clicking on the Role created automatically during creation of the Lambda function.

In the IAM console, create a new IAM policy as shown on the following figure:

![iam1](https://github.com/cloudsapiens/HANAssistant/blob/main/imgs/iam1.PNG)

Copy and paste the content of the [JSON file](https://github.com/cloudsapiens/HANAssistant/blob/main/lambda-secretsmanager-policy.json) and adjust it with proper ARN (Amazon Resource Name) for the previously created Secret. 

![iam3](https://github.com/cloudsapiens/HANAssistant/blob/main/imgs/iam3.PNG)

```Note: You can find the ARN in the AWS Secrets Manager console```

By clicking on Review, you have to provide a name and description.

Getting back to IAM console and opening the Role assigned to the Lambda function, we can now add the new policy by clicking on ```Attach policies```

Once found it, click on ```Attach policy```.

Now, the Lambda function has access to retrieve DB_USER and DB_PASSWORD for the SAP HANA database.
