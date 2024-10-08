# Amazon API Gateway Multi-Region Public REST API Failover

>Note: This repository is a companion demo for the blog post: [Implementing multi-Region failover for Amazon API Gateway](https://aws.amazon.com/blogs/compute/implementing-multi-region-failover-for-amazon-api-gateway/).

Companies often have multiple teams managing different services behind a shared public API. In disaster recovery scenarios, each team needs the ability to fail over their services independently.

This demo demonstrates an Amazon API Gateway multi-region active-passive public API that proxies two independent multi-region active-passive service APIs. The primary and secondary regions can be configured independently for the external API and each service. 

![alt text](images/diagram.jpg)

This allows you to fail over the external API and each service independently as needed for disaster recovery.

![alt text](images/routing-controls.jpg)


Learn more about this pattern at Serverless Land: https://serverlessland.com/repos/apigw-multi-region-failover

Important: this application uses various AWS services and there are costs associated with these services after the Free Tier usage - please see the [AWS Pricing page](https://aws.amazon.com/pricing/) for details. You are responsible for any AWS costs incurred. No warranty is implied in this example.

## Deployment Instructions
Before deploying this application you will need the following:
* A public domain (example.com) registered with Amazon Route 53. More details [here](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/registrar.html)
* An AWS Certificate Manager (ACM) certificate (*.example.com) for your domain name **on both primary and secondary regions** you plan to deploy your APIs on. More details [here](https://docs.aws.amazon.com/acm/latest/userguide/gs.html)


Then follow the following steps, in this exact order:

1. Create a new directory, navigate to that directory in a terminal and clone the GitHub repository:
    ``` 
    git clone https://github.com/aws-samples/apigw-multi-region-failover.git
    ```
1. [Deploy the Amazon Route 53 Application Recovery Controller (ARC) stack](route53/README.md#deployment-instructions )
1. [Deploy the service1 stack](service1/README.md#deployment-instructions)
1. [Deploy the service2 stack](service2/README.md#deployment-instructions)
1. [Deploy the external api stack](external-api/README.md#deployment-instructions)

## How it works

You will deploy 3 applications (external api, service2 and service 2) in two separate regions. The external api (i.e. https://externalapi.example.com) is your entry point to access service 1  (i.e. https://externalapi.example.com/service1) and service 2  (i.e. https://externalapi.example.com/service2). The external api uses public HTTP endpoint integrations (/service1 and /service2) to access service 1 and service 2.

If an issue with the primary region occurs, you can user Amazon Route53 ARC to route traffic to the secondary region. You can failover each application (external api, service1 and service2) independently.

This example demonstrates the failover only and does not encompass authentication and data for the multiple regions.


## Testing

Deploy all 3 applications to both primary and secondary regions. Traffic will initially be routed to the primary region only. Use Amazon Route 53 ARC to independently failover the applications to the primary or secondary region. Amazon Route 53 will then route traffic the the new chosen region for each service.

Edit the test.sh file on lines 3-5 to point to your api endpoint. Then give that file execution permissions and run it:

```bash
chmod +x ./test/sh
./test.sh
```

This script will send an HTTP request to each one of your 3 endpoints every 5 seconds. You can then use Amazon Route 53 ARC to failover your services independently and see the responses being served from different regions.

For example, on the test below, we initially had the external api and service 1 routing traffic to us-east-1. Service 2 was initially routing traffic to us-west-2.


![alt text](images/testing.jpg)

1. We failed over service2 from us-west-2 to us-east-1.
1. We failed over service1 from us-east-1 to us-west-2.
1. We failed over the external api from us-east-1 to us-west-2.

> Notes: Each service (external app, service1 and servce 2) have their own Amazon Route 53 ARC control pannel. To manage [routing controls](https://docs.aws.amazon.com/r53recovery/latest/dg/routing-control.html) for each service, you need to use their specific control panels. You can check the [route53 stack](./route53/README.md) outputs to see the details for each control panel.


## Costs
The estimated monthly cost of this solution is $1900/month. Please note this estimate is based on leveraging the free-tier for many of the services present in the solution and could vary based on consumption of the services.  Increased requests, AWS Lambda invocations and runtime, and number of DNS records and health checks used are all variables that could lead to a different monthly cost. 

> Notes: The [AWS Pricing Calculator](https://calculator.aws/#/) can be used to help you provide a more accurate representation of the costs of deploying this in your environment. Additionally, pricing for Amazon Route 53 Application Recovery Controller can be found here: [ARC Pricing](https://aws.amazon.com/route53/pricing/#Pricing_components). 

## Cleanup

Please follow the following steps, in this exact order.

1. [Delete service1 stacks](service1/README.md#cleanup)
1. [Delete service2 stacks](service2/README.md#cleanup)
1. [Delete the external stacks](external-api/README.md#cleanup)
1. [Delete the Route 53 ARC stack](route53/README.md#cleanup)


## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

