# Ipv6Lambda

On the 12th of October 2023, Lambda allows users to access resources in dual-stack VPC (outbound connections)  over IPv6. Previously, Lambda functions configured with an IPv4-only or dual-stack VPC could access VPC resources only over IPv4.

When working with IPv4 and to give internet access to an Amazon VPC-connected Lambda function, you previously had to route the outbound traffic from the private subnet where Lambda was placed to a NAT gateway or NAT instance which was held in a public subnet, where the NAT gateway would route user traffic to a Internet Gateway configured to the VPC to the Global Internet. 

The finished architecture looks like the below:

Public Subnet:
Route Table:
IGW
    Resources:
 NAT Gateway

Private Subnet:
Route Table:
Nat Gateway
Resources:
 Lambda

**IPV6 and Lambda**

When working with IPv6 and lambda and making outbound requests, your function can connect to resources in dual-stack VPC subnets over IPv6. This option is turned off by default.  

We need to set the Ipv6AllowedForDualStack=true, when this parameter is set to true, Lambda will create a new ENI with a IPv6 address to allow communication over IPv6. 

**Important Note** - in order to communicate with an IPv6 address, you typically need to have an IPv6 address yourself. So it is essential to set the Ipv6AllowedForDualStack=true. 

To allow outbound IPv6 traffic in a VPC, all of the subnets that are connected to the function must be dual-stack subnets. Lambda doesn't support outbound IPv6 connections for IPv6-only subnets in a VPC, outbound IPv6 connections for functions that are not connected to a VPC, more of this information is available in Lambda public Documentation Private networking with VPC [1].

**Network Workflow**

Instead of routing traffic from our subnet where Lambda is placed to a NAT Gateway, we now route traffic to a "Egress-Only-Internet-Gateway",  Why?

A NAT Gatewayis primarily used for outbound IPv4 traffic from private subnets in your VPC, NAT Gateways do not support IPv6 traffic and are explicitly designed for IPv4 traffic.

An Egress-Only Internet Gateway, is designed for IPv6 traffic. It allows IPv6-enabled resources in your VPC's subnets to initiate outbound connections to the IPv6 internet. 

**Cloudformation Template**

I have created a template that sets up a VPC, private subnet, and Lambda function with the ability to communicate over IPv6, I have attached a quick template summary below,

Creates a VPC, an egress-only Internet Gateway, a private subnet, a custom route table, a security group, and a Lambda function.
Associates the private subnet with the custom route table, ensuring that IPv6 traffic from the subnet is routed to the egress-only Internet Gateway.
The Lambda function is configured to use IPv6 for dual-stack communication.
The Lambda function code makes an HTTP request to "http://ipv6.google.com/" and logs the response.


**Deploying Template**

*Deploy in US-East-1 Region as the Layer is only suited for this region*, but please feel free to update the template to meet your requirements.

You can either use the AWS CLI to deploy the template or simply download the template and upload the template manually via the cloudformation console.

***Prerequisites***
You need to have the AWS CLI installed and configured with your AWS credentials. 

If you are creating a new stack, use the create-stack command:

```
aws cloudformation create-stack --stack-name YourStackName --template-body file://template.yaml
```
