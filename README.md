# Deploy Spring Boot and Docker Microservices to AWS using ECS and AWS Fargate

## Deploy Spring Boot and Docker Microservices to AWS Fargate. Implement Service Discovery, Load Balancing, Auto Discovery, Centralized Configuration and Distributed Tracing in AWS.

## Container Images

|     Application                 |    Container                                  |
| ------------------------------- | --------------------------------------------- |
| Hello World | hemantseth0210/aws-hello-world-rest-api:1.0.0-RELEASE |
| Simple Task | hemantseth0210/aws-simple-spring-task:1.0.0-RELEASE |
| CurrencyExchangeMicroservice-H2 | hemantseth0210/aws-currency-exchange-service-h2:0.0.1-SNAPSHOT |
| CurrencyExchangeMicroservice-H2 - V2 | hemantseth0210/aws-currency-exchange-service-h2:1.0.1-RELEASE|
| CurrencyExchangeMicroSevice-MySQL| hemantseth0210/aws-currency-exchange-service-mysql:0.0.1-SNAPSHOT|
| CurrencyConversionMicroservice | hemantseth0210/aws-currency-conversion-service:0.0.1-SNAPSHOT |
| Currency Exchange - X Ray | hemantseth0210/aws-currency-exchange-service-h2-xray:0.0.1-SNAPSHOT|
| Currency Conversion - X Ray | hemantseth0210/aws-currency-conversion-service-xray:0.0.1-SNAPSHOT|

|     Utility       |     Container Image        |
| ------------- | ------------------------- |
| aws-xray-daemon| amazon/aws-xray-daemon:1|

## Microservice URLs and Details

### Currency Exchange Service

- PORT - 8000
- URL - `http://localhost:8000/api/currency-exchange-microservice/currency-exchange/from/EUR/to/INR`
- HEALTH URL - `http://localhost:8000/api/currency-exchange-microservice/manage/health`
- Enviroment Variables
  - SSM URN - `arn:aws:ssm:us-east-1:<account-id>:parameter/<name>`
  - /dev/currency-exchange-service/RDS_DB_NAME  - exchange_db
  - /dev/currency-exchange-service/RDS_HOSTNAME 
  - /dev/currency-exchange-service/RDS_PASSWORD 
  - /dev/currency-exchange-service/RDS_PORT     - 3306
  - /dev/currency-exchange-service/RDS_USERNAME - exchange_db_user

### Currency Conversion Service

- PORT - 8100
- URL - `http://localhost:8100/api/currency-conversion-microservice/currency-converter/from/USD/to/INR/quantity/10`
- HEALTH URL - `http://localhost:8100/api/currency-conversion-microservice/manage/health`
- Enviroment Variables
  - SSM URN - `arn:aws:ssm:us-east-1:<account-id>:parameter/<name>`
  - /dev/currency-conversion-service/CURRENCY_EXCHANGE_URI

## Enviroment Variables

SSM URN - `arn:aws:ssm:us-east-1:<account-id>:parameter/<name>`

- /dev/currency-conversion-service/CURRENCY_EXCHANGE_URI
- /dev/currency-exchange-service/RDS_DB_NAME  - exchange_db
- /dev/currency-exchange-service/RDS_HOSTNAME 
- /dev/currency-exchange-service/RDS_PASSWORD 
- /dev/currency-exchange-service/RDS_PORT     - 3306
- /dev/currency-exchange-service/RDS_USERNAME - exchange_db_user

## Setting up App Mesh

#### Virtual nodes 
- currency-exchange-service-vn - currency-exchange-service.hemantseth0210-dev.com
- currency-conversion-service-vn - currency-conversion-service.hemantseth0210-dev.com

#### Virtual services 
- currency-exchange-service.hemantseth0210-dev.com -> currency-exchange-service-vn
- currency-conversion-service.hemantseth0210-dev.com -> currency-conversion-service-vn

#### Backend Registration
- currency-conversion-service-vn -> currency-exchange-service.hemantseth0210-dev.com

#### Task Definition Updates
- aws-currency-conversion-service
- aws-currency-exchange-service-h2
- ```ENVOY_LOG_LEVEL-trace, ENABLE_ENVOY_XRAY_TRACING-1```

#### Service Updates
- aws-currency-conversion-service-appmesh
- aws-currency-exchange-service-appmesh

## Deploying Version 2 of Currency Exchange Service to ECS and App Mesh

#### App Mesh - New Virtual Node
currency-exchange-service-v2-vn - currency-exchange-service-v2.hemantseth0210-dev.com

#### ECS Fargate - Update Task Definition
aws-currency-exchange-service-h2
 - hemantseth0210/aws-currency-exchange-service-h2:1.0.1-RELEASE
 - Use New Virtual Node
 
#### ECS Fargate - Create New Service 
aws-currency-exchange-service-v2-appmesh
- Service Discovery - currency-exchange-service-v2

#### App Mesh - Create Virtual Router
currency-exchange-service-vr distributing traffic to
- currency-exchange-service-vn
- currency-exchange-service-v2-vn

#### App Mesh - Update Service to Use Virtual Router
currency-exchange-service.hemantseth0210-dev.com -> currency-exchange-service-vr


#### jq

```
sudo yum install jq
```

#### AWS CLI Hosted Zones

```
aws --version
aws configure
aws servicediscovery list-services
aws servicediscovery delete-service --id={id}
aws servicediscovery list-services

aws servicediscovery list-namespaces
aws servicediscovery delete-namespace --id={id}
aws servicediscovery list-namespaces

aws servicediscovery delete-service --id=srv-7q3fkztnbo6aa5kc
aws servicediscovery delete-service --id=srv-mdybugm4bh5u4ugx
aws servicediscovery delete-service --id=srv-7upzjx3mhfleyfoz
aws servicediscovery delete-namespace --id=ns-ctvtysasurklojm3
```
