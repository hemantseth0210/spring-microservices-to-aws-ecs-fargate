# Deploy Spring Boot and Docker Microservices to AWS using ECS and AWS Fargate

## Learn Amazon Web Services - AWS - deploying Spring Boot and Docker Microservices to AWS Fargate. Implement Service Discovery, Load Balancing, Auto Discovery, Centralized Configuration and Distributed Tracing in AWS.

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

## Installation Guides



## Diagrams

- Courtesy http://viz-js.com/

```
graph architecture {

node[style=filled,color="#59C8DE"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB;
node[shape=record, width=2]

1 -- 2 -- 3 -- 4

1[label=<Cluster>]
2[label=<Service>]
3[label=<Task>]
4[label=<Container>]

}

digraph architecture {

node[style=filled,color="#59C8DE"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB;
node[shape=record]
CO[shape=record, width=6.5, style=filled,color="#D14D28", fontcolor=white]
CI,CC[shape=record, width=2]
CL[shape=record, width=6]
2,3,4[shape=record, width=1.5]
CI -> CO
CC -> CO
CO -> CL
CL -> 2 
CL -> 3
CL -> 4

CI[label=<Container Image(s)>]
CO[label=<Container Orchestrator>]
CC[label=<Configuration>]
CL[label=<Cluster>]
2[label=<Virtual Server 1>]
3[label=<Virtual Server 2>]
4[label=<Virtual Server 3>]

}

digraph architecture {

node[style=filled,color="#36BF80"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB;
node[shape=record]
CO[shape=record, width=6.5, style=filled,color="#D14D28", fontcolor=white]
CI,CC[shape=record, width=2]
CL[shape=record, width=6]
2,3,4[shape=record, width=1.5]
CI -> CO
CC -> CO
CO -> CL
CL -> 2
CL -> 3
CL -> 4

CI[label=<Container Image(s)>]
CO[label=<ECS>]
CC[label=<Configuration>]
CL[label=<Cluster>]
2[label=<EC2 Instance1>]
3[label=<EC2 Instance2>]
4[label=<EC2 Instance3>]
}

digraph architecture {

node[style=filled,color="#36BF80"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB;
node[shape=record]
CO[shape=record, width=6.5, style=filled,color="#D14D28", fontcolor=white]
CI,CC[shape=record, width=2]
CL[shape=record, width=6,color="#59C8DE"]
CI -> CO
CC -> CO
CO -> CL

CI[label=<Container Image(s)>]
CO[label=<AWS Fargate>]
CC[label=<Configuration>]
CL[label=<Cluster>]
}

digraph architecture {
  rankdir=TB;
{rank=same; CurrencyConversionService, CurrencyExchangeService};
Database[shape=cylinder]
CurrencyConversionService, CurrencyExchangeService[shape=component]

  CurrencyConversionService -> CurrencyExchangeService;
  
  CurrencyExchangeService->Database;
  
  CurrencyConversionService[label=<Currency Conversion Service <BR /><BR /><FONT POINT-SIZE="8">10 USD = 600 INR<BR /><BR /><BR /></FONT>>];
  CurrencyExchangeService[label=<Currency Exchange Service <BR /><BR /><FONT POINT-SIZE="8">1 USD = 60 INR<BR />1 EUR = 70 INR<BR/>1 AUD = 50 INR</FONT>>];

}

graph architecture {

node[style=filled,color="#59C8DE"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB

node[shape=record, width=0.5]
LoadBalancer[width=5,label=<Load Balancer>,color="#D14D28", fontcolor=white]
Listener1[width=3,label=<Listener <BR />>]
Listener2[width=3,label=<Listener ..>]
TG1,TG2[width=2.5,label=<Target Group <BR />>]
Rule1[width=2.75,label=<Rule<BR /><FONT POINT-SIZE="11">/api/currency-conversion-microservice</FONT>>]
Rule2[width=2.75,label=<Rule<BR /><FONT POINT-SIZE="11">/api/currency-exchange-microservice</FONT>>]
CCS1,CCS2,CCS3[label=<Conversion <BR/>Service>]
CES1,CES2[label=<Exchange <BR/>Service>]
LoadBalancer -- Listener1
LoadBalancer -- Listener2
Listener1 -- Rule1
Listener1 -- Rule2
Rule1 -- TG1
Rule2 -- TG2
TG1 -- CCS1
TG1 -- CCS2
TG1 -- CCS3
TG2 -- CES1
TG2 -- CES2

}

graph architecture {

node[style=filled,color="#59C8DE"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB;
node[shape=record]

IAM -- Console
Console -- UserID
Console -- Password
IAM -- Application
Application -- AccessKeyID
Application -- SecretAccessKey

IAM[label=<IAM User>]
AccessKeyID[label=<Access Key Id>]
UserID[label=<User Id>]
SecretAccessKey[label=<Secret Access Key>]
Console[label=<Management Console>]
Application[label=<APIs>]
}

graph architecture {

node[style=filled,color="#59C8DE"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB;
node[shape=record]

Users -- RootUser
Users -- IAM

RootUser[label=<Root User>]
IAM[label=<IAM User>]

}



graph architecture {

node[style=filled,color="#59C8DE"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB
node[shape=record, width=2]
edge [width=0]
graph [pad=".75", ranksep="0.05", nodesep="0.25"];

Applications -- Software [style=invis]
Software -- OS [style=invis]
OS -- Hardware [style=invis]

}


graph architecture {

node[style=filled,color="#59C8DE"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB
node[shape=record, width=3]

Containers, LocalImages [height=1]

DockerClient -- Daemon
Daemon -- Containers 
Daemon -- LocalImages
Daemon -- ImageRegistry

DockerClient[label=<Docker Client>]
ImageRegistry[label=<Image Registry <BR /><FONT POINT-SIZE="10">nginx<BR />mysql<BR />eureka<BR />your-app<BR /><BR /></FONT>>];
Daemon[label=<Docker Daemon>]


}


graph architecture {

node[style=filled,color="#59C8DE"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB
node[shape=record, width=2]
Hypervisor,HostOS, Hardware[shape=record, width=6.5, style=filled,color="#D14D28", fontcolor=white]
edge [width=0]
graph [pad=".75", ranksep="0.05", nodesep="0.25"];

Application1 -- Software1 [style=invis]
Application2 -- Software2 [style=invis]
Application3 -- Software3 [style=invis]

Software1 -- GuestOS1 [style=invis]
Software2 -- GuestOS2 [style=invis]
Software3 -- GuestOS3 [style=invis]
GuestOS1 -- Hypervisor [style=invis]
GuestOS2 -- Hypervisor [style=invis]
GuestOS3 -- Hypervisor [style=invis]
Hypervisor -- HostOS [style=invis]
HostOS -- Hardware [style=invis]

}


graph architecture {

node[style=filled,color="#59C8DE"]
//node [style=filled,color="#D14D28", fontcolor=white];
rankdir = TB
node[shape=record, width=2]
HostOS, CloudInfrastructure, DockerEngine[shape=record, width=6.5, style=filled,color="#D14D28", fontcolor=white]
edge [width=0]
graph [pad=".75", ranksep="0.05", nodesep="0.25"];
Container1,Container2,Container3[height=2]

Container1 -- DockerEngine [style=invis]
Container2 -- DockerEngine [style=invis]
Container3 -- DockerEngine [style=invis]
DockerEngine -- HostOS [style=invis]
HostOS -- CloudInfrastructure [style=invis]

}
```


