# AWS Training Reference Application
This repository holds reference Spring Boot project that can be deployed to AWS.

# Run locally
First run ```mvn clean install``` in root directory. Maven will generate Open API auto-generated classes.
Then, you should right-click on the **awstraining-backend** in Project structure on the left and select 
**Maven -> Generate Sources & Update Folders**.

Then, please call ```docker-compose up``` in ```/local/assembly-local``` directory.

This will set up the following components:
* DynamoDB
  * With 'Measurements' table holding measurements for devices 
  * http://localhost:8000
* DynamoDB Admin Panel
  * http://localhost:8001
* Filebeat
  * It will load Spring Boot logs from file and redirect them to Elasticsearch
* Kibana
  * It will allow visual access to application logs 
  * http://localhost:5601
* Prometheus
  * It will allow querying application metrics
  * http://localhost:9090
* Grafana
  * It will allow dashboards creation
  * http://localhost:3003
* Elasticsearch
  * It will allow indexing and saving logs for later visual access via Kibana
  * http://localhost:9200

DynamoDB will be populated with test measurement data.

Then, please configure ```application.yml```:
```yml
aws:
  region: eu-central-1
  dynamodb:
    endpoint: http://localhost:8000
    accessKey: dummyAccess
    secretKey: dummySecret
```

We have to point to our local DynamoDB instance. Access and secret keys must be set to any values, they simply cannot 
stay empty.

Finally, simply run Application in IntelliJ with 'Run' button.

# Deploying AWS infrastructure (GitHub)

## Configuring secrets in AWS
In order for our application to be able to access AWS Secrets Manager containing credentials for basic auth, please 
go to AWS Secret Manager, copy ARN of the created Secret and set it in the task definition for the region that you are deploying.

You need to add new environment variable:
- Key -> SPRING_APPLICATION_JSON
- Value type -> ValueFrom
- Value -> [copied arn of your secret from secer manager]

Then, please go to AWS Secrets Manager, open your secrets and edit JSON string.
You should add the following secrets that will create users for basic auth:
```json
{
  "backend": {
    "security": {
      "users": [
        {
          "username": "userEMEATest",
          "password": "$2a$10$uKw9ORqCF.qA3p6woHCgmeGW0jFuU9AstYhl61Uw8RTQ5AaZCfuru",
          "roles": "USER"
        }
      ]
    }
  }
}
```

Spring will automatically load this JSON to the Spring container at the application start up and user **userEMEATest** 
with password **welt** will be available for basic auth during application execution in EMEA TEST environment.