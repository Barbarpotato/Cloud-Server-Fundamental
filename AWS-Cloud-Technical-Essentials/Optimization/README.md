#  Optimizing Solutions on AWS

- Table Of Contents
    - [Elastic Load Balancer](#balancer)
    - [EC2 Auto Scaling](#auto)
    - [Excercise](#excercise)

To increase availability, you need redundancy. This typically means more infrastructure: more data centers, more servers, more databases, and more replication of data. You can imagine that adding more of this infrastructure means a higher cost. Customers want the application to always be available, but you need to draw a line where adding redundancy is no longer viable in terms of revenue. 

## Improve Application Availability
In the current application, there is only one EC2 instance used to host the application, the photos are served from Amazon Simple Storage Service (S3) and the structured data is stored in Amazon DynamoDB. That single EC2 instance is a single point of failure for the application. Even if the database and S3 are highly available, customers have no way to connect if the single instance becomes unavailable. One way to solve this single point of failure issue is by adding one more server.

## Use a Second Availability Zone
The physical location of that server is important. On top of having software issues at the operating system or application level, there can be a hardware issue. It could be in the physical server, the rack, the data center or even the Availability Zone hosting the virtual machine. An easy way to fix the physical location issue is by deploying a second EC2 instance in a different Availability Zone. That would also solve issues with the operating system and the application. However, having more than one instance brings new challenges. 


## Manage Replication, Redirection, and High Availability
Create a Process for ReplicationThe first challenge is that you need to create a process to replicate the configuration files, software patches, and application itself across instances. The best method is to automate where you can.

Address Customer RedirectionThe second challenge is how to let the clients, the computers sending requests to your server, know about the different servers. There are different tools that can be used here. The most common is using a Domain Name System (DNS) where the client uses one record which points to the IP address of all available servers. However, the time it takes to update that list of IP addresses and for the clients to become aware of such change, sometimes called propagation, is typically the reason why this method isn’t always used. 

Another option is to use a load balancer which takes care of health checks and distributing the load across each server. Being between the client and the server, the load balancer avoids propagation time issues. We discuss load balancers later.
- Active-Passive: With an active-passive system, only one of the two instances is available at a time. One advantage of this method is that for stateful applications where data about the client’s session is stored on the server, there won’t be any issues as the customers are always sent to the same server where their session is stored.
- Active-Active: A disadvantage of active-passive and where an active-active system shines is scalability. By having both servers available, the second server can take some load for the application, thus allowing the entire system to take more load. However, if the application is stateful, there would be an issue if the customer’s session isn’t available on both servers. Stateless applications work better for active-active systems.


<a name='balancer'></a>

# Route Traffic with Amazon Elastic Load Balancing
## WHAT’S A LOAD BALANCER?
Load balancing refers to the process of distributing tasks across a set of resources. In the case of the corporate directory application, the resources are EC2 instances that host the application, and the tasks are the different requests being sent. It’s time to distribute the requests across all the servers hosting the application using a load balancer.

To do this, you first need to enable the load balancer to take all of the traffic and redirect it to the backend servers based on an algorithm. The most popular algorithm is round-robin, which sends the traffic to each server one after the other.

A typical request for the application would start from the browser of the client. It’s sent to a load balancer. Then, it’s sent to one of the EC2 instances that hosts the application. The return traffic would go back through the load balancer and back to the client browser. Thus, the load balancer is directly in the path of the traffic.

Although it is possible to install your own software load balancing solution on EC2 instances, AWS provides a service for that called Elastic Load Balancing (ELB).

## FEATURES OF ELB
The ELB service provides a major advantage over using your own solution to do load balancing, in that you don’t need to manage or operate it. It can distribute incoming application traffic across EC2 instances as well as containers, IP addresses, and AWS Lambda functions.

The fact that ELB can load balance to IP addresses means that it can work in a hybrid mode as well, where it also load balances to on-premises servers.

ELB is highly available. The only option you have to ensure is that the load balancer is deployed across multiple Availability Zones.

In terms of scalability, ELB automatically scales to meet the demand of the incoming traffic. It handles the incoming traffic and sends it to your backend application.

## HEALTH CHECKS
Taking the time to define an appropriate health check is critical. Only verifying that the port of an application is open doesn’t mean that the application is working. It also doesn’t mean that simply making a call to the home page of an application is the right way either.

For example, the employee directory application depends on a database, and S3. The health check should validate all of those elements. One way to do that would be to create a monitoring webpage like “/monitor” that will make a call to the database to ensure it can connect and get data, and make a call to S3. Then, you point the health check on the load balancer to the “/monitor” page.
![ELB](/images/ELB.png)
<br>

 After determining the availability of a new EC2 instance, the load balancer starts sending traffic to it. If ELB determines that an EC2 instance is no longer working, it stops sending traffic to it and lets EC2 Auto Scaling know. EC2 Auto Scaling’s responsibility is to remove it from the group and replace it with a new EC2 instance. Traffic only sends to the new instance if it passes the health check.

In the case of a scale down action that EC2 Auto Scaling needs to take due to a scaling policy, it lets ELB know that EC2 instances will be terminated. ELB can prevent EC2 Auto Scaling from terminating the EC2 instance until all connections to that instance end, while preventing any new connections. That feature is called connection draining.

## ELB COMPONENTS
The ELB service is made up of three main components.

![ELB-2](/images/ELB-2.png)
<br>

- `Listeners`: The client connects to the listener. This is often referred to as client-side. To define a listener, a port must be provided as well as the protocol, depending on the load balancer type. There can be many listeners for a single load balancer.
- `Target groups`: The backend servers, or server-side, is defined in one or more target groups. This is where you define the type of backend you want to direct traffic to, such as EC2 Instances, AWS Lambda functions, or IP addresses. Also, a health check needs to be defined for each target group.
- `Rules`: To associate a target group to a listener, a rule must be used. Rules are made up of a condition that can be the source IP address of the client and a condition to decide which target group to send the traffic to.

# Amazon EC2 Auto Scaling
Availability and reachability is improved by adding one more server. However, the entire system can again become unavailable if there is a capacity issue. Let’s look at that load issue with both types of systems we discussed, active-passive and active-active.

## Vertical Scaling
If there are too many requests sent to a single active-passive system, the active server will become unavailable and hopefully failover to the passive server. But this doesn’t solve anything. With active-passive, you need vertical scaling. This means increasing the size of the server. With EC2 instances, you select either a larger type or a different instance type. This can only be done while the instance is in a stopped state. In this scenario, the following steps occur: 

1. Stop the passive instance. This doesn’t impact the application since it’s not taking any traffic.
2. Change the instance size or type, then start the instance again.
3. Shift the traffic to the passive instance, turning it active.
4. The last step is to stop, change the size, and start the previous active instance as both instances should match.

When the amount of requests reduces, the same operation needs to be done. Even though there aren’t that many steps involved, it’s actually a lot of manual work to do. Another disadvantage is that a server can only scale vertically up to a certain limit.

Once that limit is reached, the only option is to create another active-passive system and split the requests and functionalities across them. This could require massive application rewriting.This is where the active-active system can help. When there are too many requests, this system can be scaled horizontally by adding more servers.

## Horizontal Scaling
As mentioned above, for the application to work in an active-active system, it’s already created as stateless, not storing any client session on the server. This means that having two servers or having four wouldn’t require any application changes. It would only be a matter of creating more instances when required and shutting them down when the traffic decreases.

The Amazon EC2 Auto Scaling service can take care of that task by automatically creating and removing EC2 instances based on metrics from Amazon CloudWatch. 

You can see that there are many more advantages to using an active-active system in comparison with an active-passive. Modifying your application to become stateless enables scalability.

## Integrate ELB with EC2 Auto Scaling
The ELB service integrates seamlessly with EC2 Auto Scaling. As soon as a new EC2 instance is added to or removed from the EC2 Auto Scaling group, ELB is notified. However, before it can send traffic to a new EC2 instance, it needs to validate that the application running on that EC2 instance is available.

This validation is done via the health checks feature of ELB. Monitoring is an important part of load balancers, as it should route traffic to only healthy EC2 instances. That’s why ELB supports two types of health checks. 
- Establishing a connection to a backend EC2 instance using TCP, and marking the instance as available if that connection is successful.
- Making an HTTP or HTTPS request to a webpage that you specify, and validating that an HTTP response code is returned.

## Differentiate Between Traditional Scaling and Auto Scaling
With a traditional approach to scaling, you buy and provision enough servers to handle traffic at its peak. However, this means that at night time, there is more capacity than traffic. This also means you’re wasting money. Turning off those servers at night or at times where the traffic is lower only saves on electricity. 

The cloud works differently, with a pay-as-you-go model. It’s important to turn off the unused services, especially EC2 instances that you pay for On-Demand. One could manually add and remove servers at a predicted time. But with unusual spikes in traffic, this solution leads to a waste of resources with over-provisioning or with a loss of customers due to under-provisioning.

The need here is for a tool that automatically adds and removes EC2 instances according to conditions you define—that’s exactly what the EC2 Auto Scaling service does.

<a name='auto'></a>

# Use Amazon EC2 Auto Scaling
The EC2 Auto Scaling service works to add or remove capacity to keep a steady and predictable performance at the lowest possible cost. By adjusting the capacity to exactly what your application uses, you only pay for what your application needs. And even with applications that have steady usage, EC2 Auto Scaling can help with fleet management. If there is an issue with an EC2 instance, EC2 Auto Scaling can automatically replace that instance. This means that EC2 Auto Scaling helps both to scale your infrastructure and ensure high availability. 

## Configure EC2 Auto Scaling Components
There are three main components to EC2 Auto Scaling.
- Launch template or configuration: What resource should be automatically scaled?
- EC2 Auto Scaling Group: Where should the resources be deployed?
- Scaling policies: When should the resources be added or removed?

### Learn About Launch Templates
There are multiple parameters required to create EC2 instances: Amazon Machine Image (AMI) ID, instance type, security group, additional Amazon Elastic Block Store (EBS) volumes, and more. All this information is also required by EC2 Auto Scaling to create the EC2 instance on your behalf when there is a need to scale. This information is stored in a launch template.

You can use a launch template to manually launch an EC2 instance. You can also use it with EC2 Auto Scaling. It also supports versioning, which allows for quickly rolling back if there was an issue or to specify a default version of your launch template. This way, while iterating on a new version, other users can continue launching EC2 instances using the default version until you make the necessary changes. 

### Get to Know EC2 Auto Scaling Groups
The next component that EC2 Auto Scaling needs is an EC2 Auto Scaling Group (ASG). An ASG enables you to define where EC2 Auto Scaling deploys your resources. This is where you specify the Amazon Virtual Private Cloud (VPC) and subnets the EC2 instance should be launched in. 

EC2 Auto Scaling takes care of creating the EC2 instances across the subnets, so it’s important to select at least two subnets that are across different Availability Zones.

ASGs also allow you to specify the type of purchase for the EC2 instances. You can use On-Demand only, Spot only, or a combination of the two, which allows you to take advantage of Spot instances with minimal administrative overhead.To specify how many instances EC2 Auto Scaling should launch, there are three capacity settings to configure for the group size. 
- Minimum: The minimum number of instances running in your ASG even if the threshold for lowering the amount of instances is reached.
- Maximum: The maximum number of instances running in your ASG even if the threshold for adding new instances is reached.
- Desired capacity: The amount of instances that should be in your ASG. This number can only be within or equal to the minimum or maximum. EC2 Auto Scaling automatically adds or removes instances to match the desired capacity number. 

When EC2 Auto Scaling removes EC2 instances because the traffic is minimal, it keeps removing EC2 instances until it reaches a minimum capacity. Depending on your application, using a minimum of two is a good idea to ensure high availability, but you know how many EC2 instances at a bare minimum your application requires at all times. When reaching that limit, even if EC2 Auto Scaling is instructed to remove an instance, it does not, to ensure the minimum is kept.

On the other hand, when the traffic keeps growing, EC2 Auto Scaling keeps adding EC2 instances. This means the cost for your application will also keep growing. That’s why it’s important to set a maximum amount to make sure it doesn’t go above your budget.

The desired capacity is the amount of EC2 instances that EC2 Auto Scaling creates at the time the group is created. If that number decreases, then EC2 Auto Scaling removes the oldest instance by default. If that number increases, then EC2 Auto Scaling creates new instances using the launch template.

### Enable Automation with Scaling Policies
By default, an ASG will be kept to its initial desired capacity. Although it’s possible to manually change the desired capacity, you can also use scaling policies.

In the AWS Monitoring module, you learned about Amazon CloudWatch metrics and alarms. You use metrics to keep information about different attributes of your EC2 instance like the CPU percentage. You use alarms to specify an action when a threshold is reached. Metrics and alarms are what scaling policies use to know when to act. For example, you set up an alarm that says when the CPU utilization is above 70% across the entire fleet of EC2 instances, trigger a scaling policy to add an EC2 instance.

There are three types of scaling policies: simple, step, and target tracking scaling.

# Excercise Load Balancing and Auto Scaling

For this scenario, you are tasked with setting up an ELB load balancer and an Auto Scaling group so that your application can scale horizontally.

In this exercise, you first launch another EC2 instance. You then create an Application Load Balancer and a launch template. Next, you set up an Auto Scaling group that uses the load balancer and launch template that you created. Finally, you test and stress the application, and watch your application scale in real time.

## Task 1: Launching an EC2 instance
In this task, you will launch an EC2 instance that hosts the application.

1. If needed, log in to the AWS Management Console as your Admin user.
2. Search for and open EC2.
3. In the navigation pane, choose Instances.
4. Select the check box for the employee-directory-app-exercise6 instance, which should be in the Stopped state.
5. Choose Actions and then choose Image and templates, Launch more like this.
6. For Name and at the end of the Value, append -exercise7.
    - Example: employee-directory-app-exercise7
7.  For Key pair name, select app-key-pair.
8. Under Network settings and Auto-assign Public IP, choose Enable.
9. Choose Launch instance.
10. Choose View all instances.
    - The instance should now be in the Instances list.
11. Wait for the Instance state to change to Running and the Status check to change to 2/2 checks passed.
12. Select the check box for employee-directory-app-exercise7.
13. On the Details tab, copy the Public IPv4 address and paste it into a new browser window.
14. In a new browser window, paste the IP address that you copied. Make sure to remove the ‘S’ after HTTP so you are using only HTTP instead.

## Task 2: Creating the Application Load Balancer
In this task, you will create the Application Load Balancer.

1. Return to the Amazon EC2 console.
2. In the navigation pane, under Load Balancing, choose Load Balancers.
3. Choose Create Load Balancer.
4. On the Application Load Balancer card, choose Create.
5. Configure the following load balancer settings.
    - Load balancer name: app-alb
    - VPC: app-vpc
    - Mappings: Select both Availability Zones
        - Example: If you are in US West (Oregon), you would select both us-west-2a and us-west-2b
    - First Availability Zone Subnet: Public Subnet 1
    - Second Availability Zone Subnet: Public Subnet 2
6. In the Security groups section, remove the default security group (by choosing the X) and choose Create new security group.
    A new window opens for creating a security group.
7. Configure the following `security group` settings:
    - Security group name: load-balancer-sg
    - Description: HTTP access
    - VPC: If needed, paste the VPC ID for app-vpc and choose it when it appears under the box
        - Note: You can find the app-vpc ID by opening the VPC console in a new window
    - Inbound rules: Add Rule
    - Type: HTTP
    - Source: Anywhere-IPv4
8. Choose Create security group.
9. Close the security group browser window or return to the Load balancers window.
10. For Security groups, add the new load-balancer-sg group. Note: To see the new security group, you might need to refresh the Security groups list.
11. In Listeners and routing, choose Create target group.
    A new window opens for creating a target group.
12. For Specify group details, configure the following settings.
    - Choose a target type: Keep Instances selected
    - Target group name: app-target-group
    - Health checks: Expand Advanced health check settings and configure the following:
        - Healthy threshold: 2
        - Unhealthy threshold: 5
        -  Timeout:30
        - Interval: 40
13. Choose Next.
14. For Register targets, select employee-directory-app-exercise7 and choose Include as pending below.
15. Choose Create `target group`.
16. Close the target groups window or return to the Load balancers window.
17. Under `Listeners and routing`, refresh the available listener and choose app-target-group.
18. Finally, choose Create load balancer.
19. Choose View load balancer.
20. Make sure that app-alb is selected and wait for the load balancer State to become Active.
21. On the Description tab, copy DNS name and paste it into a text editor of your choice.
22. In the text editor, at the beginning of the URL, add http://.
    - Example: http://app-elb-123456789012.us-west-2.elb.amazonaws.com
23. Copy the DNS name (with http:// added) and paste it into a new browser window.
You should see the employee directory application.

## Task 3: Creating the launch template
Now that you can access your application from a singular DNS name, you can scale the application horizontally. To scale horizontally, you need a launch template. In this task, you will create a launch template.

1.  Back in the console, if needed, search for and open EC2.
2. In the navigation pane, under Instances, choose Launch Templates.
3. Choose Create launch template and configure the following settings.
- Launch template name: app-launch-template
- Template version description: A web server for the employee directory application
- Auto Scaling guidance: Provide guidance to help me set up a template that I can use with EC2 Auto Scaling
- Application and OS Images (Amazon Machine Image) - required: Currently in use
- Instance type: t2.micro
- Key pair name: app-key-pair
- Security groups: web-security-group
4. Expand the Advanced details section.
5. For IAM instance profile, choose S3DynamoDBFullAccessRole.
6. Scroll to User data and paste the following code:
```
#!/bin/bash -ex
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip
unzip FlaskApp.zip
cd FlaskApp/
yum -y install python3-pip
pip install -r requirements.txt
yum -y install stress
export PHOTOS_BUCKET=${SUB_PHOTOS_BUCKET}
export AWS_DEFAULT_REGION=<INSERT REGION HERE>
export DYNAMO_MODE=on
FLASK_APP=application.py /usr/local/bin/flask run --host=0.0.0.0 --port=80
```
7. In the user data code, replace the PHOTOS_BUCKET placeholder value with the name of your bucket.
    Example: export PHOTOS_BUCKET=employee-photo-bucket-al-907
8. Replace the AWS_DEFAULT_REGION placeholder value with your Region (the Region is listed at the top right, next to your user name).
    Example: This example uses US West (Oregon) (us-west-2) as the Region.
    export AWS_DEFAULT_REGION=us-west-2
9. Choose Create launch template.
10. Choose View Launch templates.

## Task 4: Creating the Auto Scaling group
In this task, you will create the Auto Scaling group.
1. In the navigation pane, under Auto Scaling, choose Auto Scaling Groups.
2. Choose Create Auto Scaling group.
3. For Choose launch template or configuration, configure these settings:
    - Auto Scaling group name: app-asg
    - Launch template: app-launch-template
4. Choose Next.
5. For Choose instance launch options, configure these settings:
    - VPC: app-vpc
    - Availability Zones and subnets: Choose the Availability Zones with Public Subnet 1 and Public Subnet 2
6. Choose Next.
7. For Configure advanced options, use these settings:
    - Load balancing: Attach to an existing load balancer
    - Attach to an existing load balancer: Choose from your load balancer target groups
    - Existing load balancer target groups: app-target-group
    - Health checks: ELB
8. Choose Next.
9. For Configure group size and scaling policies, use these settings:
    - Desired capacity: 2
    - Minimum capacity: 2
    - Maximum capacity: 4
    - Scaling policies: Target tracking scaling policy
    - Target value: 60
    - Instances need: 300
10. Choose Next.
11. For Add notifications, choose Add notification and configure these settings:
    - SNS Topic: Create a topic
    - Send a notification to: app-sns-topic
    - With these recipients: Enter your email address
12. Choose Next and then choose Next again.
13. Choose Create Auto Scaling group.
    You should receive an AWS Notification - Subscription Confirmation email.
14. Open this email message and choose Confirm subscription.
A web browser window should open with a Subscription confirmed! message.

