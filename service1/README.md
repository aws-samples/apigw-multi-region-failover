# Amazon API Gateway Multi-Region Public REST API Failover: Service 1

Service1 consists of a regional rest API with a single root path calling a Lambda function.

This stack deploys service1 on a primary and secondaty region. It will also setup Route53 public failover records and Route 53 ARC routing controls for you.

## Deployment Instructions

1. Change directory to:
    ```
    cd apigw-multi-region-failover/service1
    ```
1. From the command line, use AWS SAM to deploy the AWS resources for the stack as specified in the template.yml file on the primary region:
    ```
    sam deploy --guided --config-env primary
    ```
1. During the prompts:
    * **Stack Name:** Enter a stack name.
    * **AWS Region:** Enter the desired primary AWS Region. This stack has been tested with both us-east-1 and us-east-2.
    * **PublicHostedZoneId:** You must have a public hosted zone in Route 53 with your domain name (i.e. mydomain.com). Enter the Hosted Zone Id for this hosted zone.
    * **DomainName:** Enter your custom domain name (i.e. service1.mydomain.com).
    * **CertificateArn** You must have an ACM certificate that covers your custom domain namespace (i.e. *.mydomain.com) on the region your are deploying this stack. Enter the ARN for this certificate here. **Make sure you are getting the certificate arn for the right region**.
    * **Route53ArcClusterArn:** Before deploy this stack, you should deploy the Route 53 infrastructure. Add here the Route 53 Cluster Arn created during that deployment.
    * **Service1ControlPlaneArn**: Before deploy this stack, you should deploy the Route 53 infrastructure. Add here the  Route 53 ARC control pane Arn for service 1.
    * **Stage:** Enter the name of the stage within your API Gateway that you would like to map to your custom domain name.
    * **FailoverType:** Accept the defauls and use **PRIMARY** here.
    * Allow SAM CLI to create IAM roles with the required permissions.
    * Allow SAM CLI to create the Service1LambdaRegionalApi Lambda function.
    * **SAM configuration environment** Accept the **primary** default value.

    Once you have run `sam deploy --guided --config-env primary` mode once and saved arguments to a configuration file (samconfig.toml), you can use `sam deploy --config-env primary` in future to use these defaults.

1. From the command line, use AWS SAM to deploy the AWS resources for the stack as specified in the template.yml file on the primary region:
    ```
    sam deploy --guided --config-env secondary
    ```
1. During the prompts:
    * **Stack Name:** Enter a stack name.
    * **AWS Region:** Enter the desired secondary AWS Region. This stack has been tested with both us-east-1 and us-east-2. **Make sure to use a different region from the prymary one**.
    * **PublicHostedZoneId:** You must have a public hosted zone in Route 53 with your domain name (i.e. mydomain.com). Enter the Hosted Zone Id for this hosted zone.
    * **DomainName:** Enter your custom domain name (i.e. service1.mydomain.com).
    * **CertificateArn** You must have an ACM certificate that covers your custom domain namespace (i.e. *.mydomain.com) on the region your are deploying this stack. Enter the ARN for this certificate here. **Make sure you are getting the certificate arn for the right region**.
    * **Route53ArcClusterArn**: Before deploy this stack, you should deploy the Route 53 infrastructure. Add here the Route 53 Cluster Arn created during that deployment.
    * **Service1ControlPlaneArn**: Before deploy this stack, you should deploy the Route 53 infrastructure. Add here the  Route 53 ARC control pane Arn for service 1.
    * **Stage:** Enter the name of the stage within your API Gateway that you would like to map to your custom domain name.
    * **FailoverType:** Accept the defauls and use **SECONDARY** here.
    * Allow SAM CLI to create IAM roles with the required permissions.
    * Allow SAM CLI to create the Service1LambdaRegionalApi Lambda function.
    * **SAM configuration environment** Accept the **primary** default value.

    Once you have run `sam deploy --guided --config-env secondary` mode once and saved arguments to a configuration file (samconfig.toml), you can use `sam deploy --config-env secondary` in future to use these defaults.
    
1. Note the outputs from the SAM deployment process. These contain details which are used for testing.

## How it works

This stack will deploy an Amazon API Gateway Rest Regional API with a Lambda integration. The AWS Lambda function is written in Python3.9. The function returns a small message with the service name and the region it is deployed at. The inline code of the lambda is written in the template itself.

## Testing

Once the stack is deployed, get the API endpoint from the EndpointUrl output parameter.
Paste the URL in a browser, or in Postman, or using the curl command.
Eg: 
```bash
curl https://aabbccddee.execute-api.us-east-1.amazonaws.com/prod
```

You should see a response similar to:
```json
{"service": "service1", "region": "your-selected-region"}
```


Now test that one of your regional services is accessible via your custom fomain.
You can get that URL from the **CustomDomainNameEndpoint** output parameter.
Eg: 
```bash
curl https://service1.mydomain.com
```

You should see a response similar to:
```json
{"service": "service1", "region": "your-primary-region"}
```

## Cleanup
 
1. Delete the stack on the primary region.
    ```bash
    sam delete --config-env primary
    ```
1. Delete the stack on the secondary region.
    ```bash
    sam delete --config-env secondary
    ```
----
Copyright 2023 Amazon.com, Inc. or its affiliates. All Rights Reserved.

SPDX-License-Identifier: MIT-0