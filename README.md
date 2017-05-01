# AWS X-Ray Microservices Calculator

![Alt text](documentation/ServiceMap.png?raw=true "Amazon X-Ray Console Service Map")

## Architecture

![Alt text](documentation/XRayDockerArch.png?raw=true "AWS X-Ray Microservices Calculator")

Dockerised Microservices Calculator - demonstrating AWS X-Ray instrumentation and telemetry workflows.

## Background

This project implements a simple Node.js microservices based calculator for the purpose demonstrating Amazon's newly launched X-Ray service: `https://aws.amazon.com/xray/`

AWS X-Ray helps developers analyze and debug production, distributed applications, such as those built using a microservices architecture. With X-Ray, you can understand how your application and its underlying services are performing to identify and troubleshoot the root cause of performance issues and errors. X-Ray provides an end-to-end view of requests as they travel through your application, and shows a map of your application’s underlying components. You can use X-Ray to analyze both applications in development and in production, from simple three-tier applications to complex microservices applications consisting of thousands of services.

The Node.js microservices based calculator has been instrumented with the Node.js `aws-xray-sdk` - allowing it to propogate telemetry into the Amazon X-Ray cloud hosted service.

The sample project has been designed to run locally on a workstation using Docker containers.

A `docker-compose.yml` file has been provided to orchestrate the provisioning of the entire microservices docker container architecture.

## Prerequisites

You will need to have a Docker runtime installed locally. This project uses both `docker` and `docker-compose` utilities. There are generally 2 approaches to installing a workstation Docker runtime:
* Download and install Docker Toolbox from: `https://www.docker.com/products/docker-toolbox`, or
* Install and configure Vagrant - then download and setup using CoreOS box located at: `https://github.com/coreos/coreos-vagrant`

This project has been successfully tested on:

1. Docker Community Edition 17.03.1-ce-mac5 running on macOS Sierra 10.12.4
* `docker --version` -> `Docker version 17.03.1-ce, build c6d412e`
* `docker-compose --version` -> `docker-compose version 1.11.2, build dfed245`

## Installation

1. Create a new IAM credential for the XRAY and SQS service accesses. Ensure that the credential has API programmatic access - this will provision an ACCESS_KEY and SECRET_ACCESS_KEY - we will add these into the `.env` configuration file (step 4 below).
2. Attach the following 2 IAM policies:
    1. `AWSXrayWriteOnlyAccess`
    2. `AmazonSQSFullAccess`

Notes: 
AWSXrayWriteOnlyAccess
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "xray:PutTraceSegments",
                "xray:PutTelemetryRecords"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

AmazonSQSFullAccess
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "sqs:*"
            ],
            "Effect": "Allow",
            "Resource": "*"
        }
    ]
}
```

3. Create a new Amazon SQS queue. Record the SQS URL -  we will add this into the `.env` configuration file (step 4 below).

4. Create a `.env` file in the project root directory. Add the following enviroment variables:
```javascript
AWS_ACCESS_KEY_ID=<your access key here>
AWS_SECRET_ACCESS_KEY=<your secret access key here>
AWS_REGION=<aws region>
TIMEZONE=<time zone that the X-Ray daemon runs in>
CALC_SQS_QUEUE=<your SQS URL>
```

example `.env` file:

```javascript
AWS_ACCESS_KEY_ID=ABCD1234ABCD1234ABCD
AWS_SECRET_ACCESS_KEY=abcd1234ABCD1234abcd1234ABCD1234abcd1234
AWS_REGION=ap-southeast-2
TIMEZONE=Pacific/Auckland
CALC_SQS_QUEUE=https://sqs.ap-southeast-2.amazonaws.com/123456789012/calclog-syd
```

5. Run `docker-compose build` from within the project root directory - this step will take approx 5mins to complete as it downloads the base images over the Internet.
6. Run `docker images` - this will list all of the container images that have just been built:

```bash
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
node-subtract       latest              034ff79532fe        57 seconds ago       128 MB
node-calc           latest              1ba68a9c70b9        About a minute ago   166 MB
node-multiply       latest              234f14958e8f        2 minutes ago        128 MB
node-add            latest              02b62d251ffb        2 minutes ago        148 MB
xray-daemon         latest              f73e98971b76        2 minutes ago        107 MB
node-postfix        latest              541aa36c5874        5 minutes ago        166 MB
node-power          latest              8d10cdae9009        6 minutes ago        128 MB
node-divide         latest              34b4bcc5ade0        7 minutes ago        128 MB
debian              stable-slim         40617bfb5493        6 days ago           80.3 MB
node                alpine              7fce0a61c1d6        10 days ago          59 MB
```

7. Run `docker-compose up` from within the project root directory:

```bash
Creating POSTIX
Creating POWER
Creating DIVIDE
Creating SUBTRACT
Creating MULTIPLY
Creating ADD
Creating XRAY
Creating CALC
Attaching to CALC, POSTIX, POWER, SUBTRACT, ADD, XRAY, MULTIPLY, DIVIDE
CALC        | npm info it worked if it ends with ok
CALC        | npm info using npm@4.2.0
POSTIX      | npm info it worked if it ends with ok
SUBTRACT    | npm info it worked if it ends with ok
XRAY        | 2017-05-01T14:03:49+12:00 [Info] Initializing AWS X-Ray daemon 1.0.1
POSTIX      | npm info using npm@4.2.0
XRAY        | 2017-05-01T14:03:49+12:00 [Info] Using memory limit of 99 MB
SUBTRACT    | npm info using npm@4.2.0
POWER       | npm info it worked if it ends with ok
POWER       | npm info using npm@4.2.0
SUBTRACT    | npm info using node@v7.9.0
XRAY        | 2017-05-01T14:03:49+12:00 [Info] 633 segment buffers allocated
POSTIX      | npm info using node@v7.9.0
POWER       | npm info using node@v7.9.0
CALC        | npm info using node@v7.9.0
CALC        | npm info lifecycle node-api@~prestart: node-api@
CALC        | npm info lifecycle node-api@~start: node-api@
CALC        | 
CALC        | > node-api@ start /usr/src/app
CALC        | > node server.js
CALC        | 
CALC        | CALCULATOR service listening on port: 8080
POSTIX      | npm info lifecycle node-api@~prestart: node-api@
POSTIX      | npm info lifecycle node-api@~start: node-api@
POSTIX      | 
POSTIX      | > node-api@ start /usr/src/app
POSTIX      | > node server.js
POSTIX      | 
POWER       | npm info lifecycle node-api@~prestart: node-api@
POWER       | npm info lifecycle node-api@~start: node-api@
POWER       | 
POWER       | > node-api@ start /usr/src/app
POWER       | > node server.js
POWER       | 
SUBTRACT    | npm info lifecycle node-api@~prestart: node-api@
SUBTRACT    | npm info lifecycle node-api@~start: node-api@
SUBTRACT    | 
SUBTRACT    | > node-api@ start /usr/src/app
SUBTRACT    | > node server.js
SUBTRACT    | 
MULTIPLY    | npm info it worked if it ends with ok
MULTIPLY    | npm info using npm@4.2.0
MULTIPLY    | npm info using node@v7.9.0
ADD         | npm info it worked if it ends with ok
ADD         | npm info using npm@4.2.0
ADD         | npm info using node@v7.9.0
DIVIDE      | npm info it worked if it ends with ok
DIVIDE      | npm info using npm@4.2.0
DIVIDE      | npm info using node@v7.9.0
ADD         | npm info lifecycle node-api@~prestart: node-api@
ADD         | npm info lifecycle node-api@~start: node-api@
ADD         | 
ADD         | > node-api@ start /usr/src/app
ADD         | > node server.js
ADD         | 
DIVIDE      | npm info lifecycle node-api@~prestart: node-api@
MULTIPLY    | npm info lifecycle node-api@~prestart: node-api@
DIVIDE      | npm info lifecycle node-api@~start: node-api@
DIVIDE      | 
DIVIDE      | > node-api@ start /usr/src/app
DIVIDE      | > node server.js
DIVIDE      | 
MULTIPLY    | npm info lifecycle node-api@~start: node-api@
MULTIPLY    | 
MULTIPLY    | > node-api@ start /usr/src/app
MULTIPLY    | > node server.js
MULTIPLY    | 
SUBTRACT    | SUBTRACT service listening on port: 8082
POWER       | POWER service listening on port: 8085
ADD         | ADD service listening on port: 8081
POSTIX      | POSTFIX service listening on port: 9090
MULTIPLY    | MULTIPLY service listening on port: 8083
DIVIDE      | DIVIDE service listening on port: 8084
```

8. In another console window, fire a test calculation at it:

`curl --data-urlencode "calcid=testid123" --data-urlencode "expression=(2*(9+22/5)-((9-1)/4)^2)+(3^2+((5*5-1)/2)" http://localhost:8080/api/calc'`

9. Examine the console output of the response - the answer should be `43.8`

10. Examine the console output of the `docker-compose up` console:

```bash
CALC        | =====================================
CALC        | Calculator entry point...
CALC        | generating new calcid: rkd-QfNJZ
CALC        | calcid: rkd-QfNJZ, infix: (2*(9+22/5)-((9-1)/4)^2)+(3^2+((5*5)-1)/2)
POSTIX      | POSTFIX->calcid: rkd-QfNJZ, infix: (2*(9+22/5)-((9-1)/4)^2)+(3^2+((5*5)-1)/2)
POSTIX      | POSTFIX->calcid: rkd-QfNJZ, postfix: 2 9 22 5 / + * 9 1 - 4 / 2 ^ - 3 2 ^ 5 5 * 1 - 2 / + +
CALC        | STATUS: 200
CALC        | HEADERS: {"x-powered-by":"Express","date":"Mon, 01 May 2017 01:44:32 GMT","connection":"close","transfer-encoding":"chunked"}
CALC        | BODY: 2 9 22 5 / + * 9 1 - 4 / 2 ^ - 3 2 ^ 5 5 * 1 - 2 / + +
CALC        | postfix:2 9 22 5 / + * 9 1 - 4 / 2 ^ - 3 2 ^ 5 5 * 1 - 2 / + +
CALC        | http request host:port -> 172.19.10.4:8084
DIVIDE      | dividing...
DIVIDE      | 22/5=4.4
CALC        | STATUS: 200
CALC        | result=4.4
CALC        | http request host:port -> 172.19.10.1:8081
ADD         | adding...
ADD         | 4.4+9=13.4
CALC        | STATUS: 200
CALC        | result=13.4
CALC        | http request host:port -> 172.19.10.3:8083
MULTIPLY    | multiplying...
MULTIPLY    | 13.4*2=26.8
CALC        | STATUS: 200
CALC        | result=26.8
CALC        | http request host:port -> 172.19.10.2:8082
SUBTRACT    | subtracting...
SUBTRACT    | 9-1=8
CALC        | STATUS: 200
CALC        | result=8
CALC        | http request host:port -> 172.19.10.4:8084
DIVIDE      | dividing...
DIVIDE      | 8/4=2
CALC        | STATUS: 200
CALC        | result=2
CALC        | http request host:port -> 172.19.10.5:8085
POWER       | powering...
POWER       | 2^2=4
CALC        | STATUS: 200
CALC        | result=4
CALC        | http request host:port -> 172.19.10.2:8082
SUBTRACT    | subtracting...
SUBTRACT    | 26.8-4=22.8
CALC        | STATUS: 200
CALC        | result=22.8
CALC        | http request host:port -> 172.19.10.5:8085
POWER       | powering...
POWER       | 3^2=9
CALC        | STATUS: 200
CALC        | result=9
CALC        | http request host:port -> 172.19.10.3:8083
CALC        | STATUS: 200
CALC        | result=25
CALC        | http request host:port -> 172.19.10.2:8082
MULTIPLY    | multiplying...
MULTIPLY    | 5*5=25
SUBTRACT    | subtracting...
SUBTRACT    | 25-1=24
CALC        | STATUS: 200
CALC        | result=24
CALC        | http request host:port -> 172.19.10.4:8084
DIVIDE      | dividing...
DIVIDE      | 24/2=12
CALC        | STATUS: 200
CALC        | result=12
CALC        | http request host:port -> 172.19.10.1:8081
ADD         | adding...
ADD         | 12+9=21
CALC        | STATUS: 200
CALC        | result=21
CALC        | http request host:port -> 172.19.10.1:8081
ADD         | adding...
ADD         | 21+22.8=43.8
CALC        | STATUS: 200
CALC        | result=43.8
CALC        | result=43.8
DIVIDE      | sqs success for DIVIDE service 50b0243f-0673-4e37-b80f-d6b1081cfd1c
ADD         | sqs success for ADD service 53d45f5c-df5b-45f9-833e-d1d4cc0e4f53
POSTIX      | sqs success for POSTFIX service 9a4c3c75-1449-4947-961b-ac799bd1c515
SUBTRACT    | sqs success for SUBTRACT service 018df1d4-a25e-403b-9db8-d1fec2958d62
MULTIPLY    | sqs success for MULTIPLY service 2e57edfd-09f0-431a-a917-7e92b68d1e0e
DIVIDE      | sqs success for DIVIDE service acad14c1-5f8b-4e4e-9851-b2bff504908a
SUBTRACT    | sqs success for SUBTRACT service 8f269077-8e9f-48ec-bc10-9e10a5a497f9
POWER       | sqs success for POWER service f64f2d3d-7002-4c1c-baa4-22ab47ad2c5c
MULTIPLY    | sqs success for MULTIPLY service cc6b6eb9-3978-45a0-978b-601572b3bbd1
XRAY        | 2017-05-01T13:44:32+12:00 [Info] Successfully sent batch of 1 segments (0.198 seconds)
POWER       | sqs success for POWER service 32792ba3-c0b2-42d3-9c1c-b0bb04d2424d
ADD         | sqs success for ADD service b5262b65-c8b3-4c90-b320-186903dc92c7
SUBTRACT    | sqs success for SUBTRACT service f7800716-7d07-4bdd-9ae1-c692177b67d0
ADD         | sqs success for ADD service 362adfcc-e750-4049-acc2-a6ecde0230d4
DIVIDE      | sqs success for DIVIDE service f61ea040-55f6-44de-8885-c5c466907f74
XRAY        | 2017-05-01T13:44:33+12:00 [Info] Successfully sent batch of 14 segments (0.214 seconds)
```

11. Login into the AWS X-Ray console

    1. Examine the Service Map:

        ![Alt text](documentation/ServiceMap.png?raw=true "Amazon X-Ray Console Service Map")

    2. Perform a filtered trace and examine response codes:

        ![Alt text](documentation/Trace1.png?raw=true "Amazon X-Ray Console Trace - filtered search")

        ![Alt text](documentation/Trace2.png?raw=true "Amazon X-Ray Console Trace - examine response codes")
    3. Examine the configured SQS queue

## Notes

1. The X-Ray daemon may fail on its 1st attempt to publish batch results to the AWS X-Ray service - all subsequent batch sends will work.

2. The docker containers POSTFIX, ADD, SUBTRACT, MULTIPLY, DIVIDE, and POWER each publish a message to the configured SQS queue - this is done only to demostrate how AWS services can be intrumented against, the messages on the SQS queue are not consumed by any external service. The SQS queue should be purged when the sample project is torn down.