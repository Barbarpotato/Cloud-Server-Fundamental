# AWS Global Infrastructure

- Table Of Contents:
    - [AWS Identity and Access Management (IAM)](#IAM)

## REGIONS
Regions are geographic locations worldwide where AWS hosts its data centers. AWS Regions are named after the location where they reside. For example, in the United States, there is a Region in Northern Virginia called the Northern Virginia Region and a Region in Oregon called the Oregon Region. There are Regions in Asia Pacific, Canada, Europe, the Middle East, and South America, and AWS continues to expand to meet the needs of its customers.Each AWS Region is associated with a geographical name and a Region code.

### CHOOSE THE RIGHT AWS REGION
Consider four main aspects when deciding which AWS Region to host your applications and workloads: latency, price, service availability, and compliance.
1. Latency. If your application is sensitive to latency, choose a Region that is close to your user base. This helps prevent long wait times for your customers. Synchronous applications such as gaming, telephony, WebSockets, and IoT are significantly affected by higher latency, but even asynchronous workloads, such as ecommerce applications, can suffer from an impact on user connectivity.
2. Price. Due to the local economy and the physical nature of operating data centers, prices may vary from one Region to another. The pricing in a Region can be impacted by internet connectivity, prices of imported pieces of equipment, customs, real estate, and more. Instead of charging a flat rate worldwide, AWS charges based on the financial factors specific to the location.
3. Service availability. Some services may not be available in some Regions. The AWS documentation provides a table containing the Regions and the available services within each one.
4. Data compliance. Enterprise companies often need to comply with regulations that require customer data to be stored in a specific geographic territory. If applicable, you should choose a Region that meets your compliance requirements.

## AVAILABILITY ZONES
Inside every Region is a cluster of Availability Zones (AZ). An AZ consists of one or more data centers with redundant power, networking, and connectivity. These data centers operate in discrete facilities with undisclosed locations. They are connected using redundant high-speed and low-latency links.AZs also have a code name. Since they’re located inside Regions, they can be addressed by appending a letter to the end of the Region code name. For example:
- us-east-1a: an AZ in us-east-1 (Northern Virginia Region)
- sa-east-1b: an AZ in sa-east-1 (São Paulo Region in South America)

If you see that a resource exists in us-east-1c, you know this resource is located in AZ c of the us-east-1 Region.

## Interacting with AWS

### THE AWS MANAGEMENT CONSOLE
One way to manage cloud resources is through the web-based console, where you log in and click on the desired service. This can be the easiest way to create and manage resources when you’re first begin working with the cloud. Below is a screenshot that shows the landing page when you first log into the AWS Management Console.

### THE AWS COMMAND LINE INTERFACE (CLI)
Consider the scenario where you run tens of servers on AWS for your application’s frontend. You want to run a report to collect data from all of these servers. You need to do this programmatically every day because the server details may change. Instead of manually logging into the AWS Management Console and copying/pasting information, you can schedule an AWS Command Line Interface (CLI) script with an API call to pull this data for you.The AWS CLI is a unified tool to manage AWS services. With just one tool to download and configure, you control multiple AWS services from the command line and automate them with scripts. The AWS CLI is open-source, and there are installers available for Windows, Linux, and Mac OS.

### AWS SOFTWARE DEVELOPMENT KITS (SDKS)
API calls to AWS can also be performed by executing code with programming languages. You can do this by using AWS Software Development Kits (SDKs). SDKs are open-source and maintained by AWS for the most popular programming languages, such as C++, Go, Java, JavaScript, .NET, Node.js, PHP, Python, and Ruby.Developers commonly use AWS SDKs to integrate their application source code with AWS services. Let’s say the frontend of the application runs in Python and every time it receives a cat photo, it uploads that photo to a storage service. This action can be achieved from within the source code by using the AWS SDK for Python.

# Security in the AWS Cloud

## Security and the AWS Shared Responsibility Model
When you begin working with the AWS Cloud, managing security and compliance is a shared responsibility between AWS and you. To depict this shared responsibility, AWS created the shared responsibility model. This distinction of responsibility is commonly referred to as security of the cloud, versus security in the cloud.
![security](/images/security-1.png)<br>

<a name="IAM></a>

## Introduction to AWS Identity and Access Management

### WHAT IS IAM?
IAM is a web service that enables you to manage access to your AWS account and resources. It also provides a centralized view of who and what are allowed inside your AWS account (authentication), and who and what have permissions to use and work with your AWS resources (authorization).With IAM, you can share access to an AWS account and resources without having to share your set of access keys or password. You can also provide granular access to those working in your account, so that people and services only have permissions to the resources they need. For example, to provide a user of your AWS account with read-only access to a particular AWS service, you can granularly select which actions and which resources in that service they can access.

### WHAT IS AN IAM USER?
An IAM user represents a person or service that interacts with AWS. You define the user within your AWS account. And any activity done by that user is billed to your account. Once you create a user, that user can sign in to gain access to the AWS resources inside your account.You can also add more users to your account as needed. For example, for your cat photo application, you could create individual users in your AWS account that correspond to the people who are working on your application. Each person should have their own login credentials. Providing users with their own login credentials prevents sharing of credentials.

### WHAT IS AN IAM GROUP?
An IAM group is a collection of users. All users in the group inherit the permissions assigned to the group. This makes it easy to give permissions to multiple users at once. It’s a more convenient and scalable way of managing permissions for users in your AWS account. This is why using IAM groups is a best practice.If you have a an application that you’re trying to build and have multiple users in one account working on the application, you might decide to organize these users by job function. You might want IAM groups organized by developers, security, and admins. You would then place all of your IAM users in the respective group for their job function.This provides a better view to see who has what permissions within your organization and an easier way to scale as new people join, leave, and change roles in your organization.Consider the following examples.

A new developer joins your AWS account to help with your application. You simply create a new user and add them to the developer group, without having to think about which permissions they need.

A developer changes jobs and becomes a security engineer. Instead of editing the user’s permissions directly, you can instead remove them from the old group and add them to the new group that already has the correct level of access.

### WHAT IS AN IAM POLICY?
To manage access and provide permissions to AWS services and resources, you create IAM policies and attach them to IAM users, groups, and roles. Whenever a user or role makes a request, AWS evaluates the policies associated with them. For example, if you have a developer inside the developers group who makes a request to an AWS service, AWS evaluates any policies attached to the developers group and any policies attached to the developer user to determine if the request should be allowed or denied.

Most policies are stored in AWS as JSON documents with several policy elements. Take a look at the following example of what providing admin access through an IAM identity-based policy looks like.
```
{

"Version": "2012-10-17",    

     "Statement": [{        
          "Effect": "Allow",        

          "Action": "*",        

          "Resource": "*"     

     }]

}

In this polic
```

### Creating an IAM user
- In this task, you will create an IAM user with administrator access.
    1. In the Services search box, enter IAM, and open the IAM console.
    2. In the navigation pane, choose Users.
    3. Choose Add users and in the Specify user details page, configure the following settings.
    4. For User name use Admin
    5. Select Provide user access to the AWS Management Console - optional.
    6. Under Are you providing console access to a person? choose I want to create an IAM user.
    7. Under Console password select Custom password and enter in a complex password.
    8. De-select Users must create a new password at next sign-in - Recommended. Choose Next.
    9. In the Set permissions page, choose Attach existing policies directly.
    10. In the Permissions policies box, search for administrator.
    11. Under Policy name, select AdministratorAccess.
    12. Choose Next, and then choose Create user.
    13. You can sign in with the new IAM admin user by choosing the Console sign-in URL.
    Note: The sign-in URL should look like the following: https://123456789012.signin.aws.amazon.com/console.
    Log in to the console with the Admin user and password that you created.

###  Setting up an IAM role for an EC2 instance
In this task, you will log in as the Admin user and create an IAM role. The role allows Amazon Elastic Compute Cloud (Amazon EC2) to access both Amazon Simple Storage Service (Amazon S3) and Amazon DynamoDB. You will later assign this role to an EC2 instance that hosts the employee directory application.

1. Now that you are logged in as the Admin user, use the Services search bar to search for IAM again, and open the service by choosing IAM.
2. In the navigation pane, choose Roles.
3. Choose Create role.
4. In the Select trusted entity page, configure the following settings.
    - Trusted entity type: AWS service
    - Use case: EC2
5. Choose Next.
6. In the permissions filter box, search for amazons3full, and select AmazonS3FullAccess.
7. In the filter box, search for amazondynamodb, and select AmazonDynamoDBFullAccess.
8. Choose Next.
9. For Role name, paste S3DynamoDBFullAccessRole and choose Create role.