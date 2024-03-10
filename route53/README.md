# Amazon API Gateway Multi-Region Public REST API Failover: Route 53 ARC Infrastructure

This stack creates a Route 53 ARC cluster and one control panel per api to be used during the failover process.

Learn more about this pattern at Serverless Land Patterns: << Add the live URL here >>


## Deployment Instructions

1. Change directory to the route53 stack:
    ```
    cd apigw-multi-region-failover/route53
    ```
1. From the command line, use AWS SAM to deploy the AWS resources for the pattern as specified in the template.yml file:
    ```
    sam deploy --guided
    ```
1. During the prompts:
    * Enter a stack name
    * Enter the desired AWS Region. This stack will only create global AWS resources, but the region you select is where the Cloud Formation stack will be created. This stack has been tested with both us-east-1 and us-east-2.
    * Allow SAM CLI to create IAM roles with the required permissions.

    Once you have run `sam deploy --guided` mode once and saved arguments to a configuration file (samconfig.toml), you can use `sam deploy` in future to use these defaults.

1. Note the outputs from the SAM deployment process, since you will need it as inputs to deploy the other stacks.

## How it works

This stack will create a [Route53 ARC cluster](https://docs.aws.amazon.com/r53recovery/latest/dg/introduction-components.html) and 3 control panels, one for each service (extarnal api, service 1 and service 2) so you can independently manage [routing controls](https://docs.aws.amazon.com/r53recovery/latest/dg/routing-control.html) and fail them over.


## Cleanup
 
1. Delete the stack
    ```bash
    sam delete
    ```