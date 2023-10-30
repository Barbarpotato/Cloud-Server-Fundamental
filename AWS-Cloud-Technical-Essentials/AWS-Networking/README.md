# Networking on AWS

- Table Of Contents
    - [Networking](#Networking)
    - [Virtual Private Cloud (VPC)](#VPC)
    - [Route Tables](#route_table)
    - [ACL & Security Groups](#acl_sec)
    - [Excercise](#excercise)

<a name='Networking'></a>

## WHAT IS NETWORKING?

Networking is how you connect computers around the world and allow them to communicate with one another. In this trail, you’ve already seen a few examples of networking. One is the AWS global infrastructure. AWS has created a network of resources using data centers, Availability Zones, and Regions.

## KNOW THE NETWORKING BASICS

Think about sending a letter. When sending a letter, there are three pieces of information you need.
- The payload or letter inside the envelope.
- The address of the sender in the From section.
- The address of the recipient in the To section.

Let’s go further. Each address must contain information such as:

- Name of sender and recipient
- Street
- City
- State or province
- Zip, area, or postal code
- Country

You need all parts of an address to ensure that your letter gets to its destination. Without the correct address, postal workers are not able to properly deliver the message. In the digital world, computers handle the delivery of messages in a similar way. This is called routing.

## WHAT ARE IP ADDRESSES?
In order to properly route your messages to a location, you need an address. Just like each home has a mail address, each computer has an IP address. However, instead of using the combination of street, city, state, zip code, and country, the IP address uses a combination of bits, 0s and 1s. 
![binary](/images/binary.png)
<br>

## WHAT IS IPV4 NOTATION?
Typically, you don’t see an IP address in this binary format. Instead, it’s converted into decimal format and noted as an Ipv4 address. 

In the diagram below, the 32 bits are grouped into groups of 8 bits, also called octets. Each of these groups is converted into decimal format separated by a period. 

![ipv4](/images/ipv4.png)
<br>
In the end, this is what is called an Ipv4 address. This is important to know when trying to communicate to a single computer. But remember, you’re working with a network. This is where CIDR Notation comes in.

## USE CIDR NOTATION

192.168.1.30 is a single IP address. If you wanted to express IP addresses between the range of 192.168.1.0 and 192.168.1.255, how can you do that? 

One way is by using Classless Inter-Domain Routing (CIDR) notation. CIDR notation is a compressed way of specifying a range of IP addresses. Specifying a range determines how many IP addresses are available to you. 

CIDR notation looks like this: 
![cidr](/images/cidr.png)
<br>
It begins with a starting IP address and is separated by a forward slash (the “/” character) followed by a number. The number at the end specifies how many of the bits of the IP address are fixed. In this example, the first 24 bits of the IP address are fixed. The rest are flexible. 
![cidr-2](/images/cidr-2.png)
<br>

32 total bits subtracted by 24 fixed bits leaves 8 flexible bits. Each of these flexible bits can be either 0 or 1, because they are binary. That means you have two choices for each of the 8 bits, providing 256 IP addresses in that IP range. 

The higher the number after the /, the smaller the number of IP addresses in your network. For example, a range of 192.168.1.0/24 is smaller than 192.168.1.0/16. 

When working with networks in the AWS Cloud, you choose your network size by using CIDR notation. In AWS, the smallest IP range you can have is /28, which provides you 16 IP addresses. The largest IP range you can have is a /16, which provides you with 65,536 IP addresses.

<a name='VPC'></a>

## Introduction to Amazon VPC (Virtual Private Cloud)
A VPC is an isolated network you create in the AWS cloud, similar to a traditional network in a data center. When you create a VPC, you need to choose three main things. 
1. The name of your VPC.
2. A Region for your VPC to live in. Each VPC spans multiple Availability Zones within the Region you choose.
3. A IP range for your VPC in CIDR notation. This determines the size of your network. Each VPC can have up to four /16 IP ranges.

Using this information, AWS will provision a network and IP addresses for that network.   
![vpc-1](/images/vpc-1.png)
<br>

`Create a Subnet` After you create your VPC, you need to create subnets inside of this network. Think of subnets as smaller networks inside your base network—or virtual area networks (VLANs) in a traditional, on-premises network. In an on-premises network, the typical use case for subnets is to isolate or optimize network traffic. In AWS, subnets are used for high availability and providing different connectivity options for your resources. When you create a subnet, you need to choose three settings.
1. The VPC you want your subnet to live in, in this case VPC (10.0.0.0/16).
2. The Availability Zone you want your subnet to live in, in this case AZ1.
3. A CIDR block for your subnet, which must be a subset of the VPC CIDR block, in this case 10.0.0.0/24.

When you launch an EC2 instance, you launch it inside a subnet, which will be located inside the Availability Zone you choose.     
![vpc-2](/images/vpc-2.png)
<br>

`High Availability` with A VPC When you create your subnets, keep high availability in mind. In order to maintain redundancy and fault tolerance, create at least two subnets configured in two different Availability Zones.   

As you learned earlier in the trail, it’s important to consider that “everything fails all the time.” In this case, if one of these AZs fail, you still have your resources in another AZ available as backup.   
![vpc-3](/images/vpc-3.png)
<br>

`Reserved IPs` For AWS to configure your VPC appropriately, AWS reserves five IP addresses in each subnet. These IP addresses are used for routing, Domain Name System (DNS), and network management.  

For example, consider a VPC with the IP range 10.0.0.0/22. The VPC includes 1,024 total IP addresses. This is divided into four equal-sized subnets, each with a /24 IP range with 256 IP addresses. Out of each of those IP ranges, there are only 251 IP addresses that can be used because AWS reserves five.  
![vpc-4](/images/vpc-4.png)

Since AWS reserves these five IP addresses, it can impact how you design your network. A common starting place for those who are new to the cloud is to create a VPC with a IP range of /16 and create subnets with a IP range of /24. This provides a large amount of IP addresses to work with at both the VPC and subnet level. 

### Gateways
#### Internet Gateway 
To enable internet connectivity for your VPC, you need to create an internet gateway. Think of this gateway as similar to a modem. Just as a modem connects your computer to the internet, the internet gateway connects your VPC to the internet. Unlike your modem at home, which sometimes goes down or offline, an internet gateway is highly available and scalable. After you create an internet gateway, you then need to attach it to your VPC.  

#### Virtual Private Gateway  

A virtual private gateway allows you to connect your AWS VPC to another private network. Once you create and attach a VGW to a VPC, the gateway acts as anchor on the AWS side of the connection. On the other side of the connection, you’ll need to connect a customer gateway to the other private network. A customer gateway device is a physical device or software application on your side of the connection. Once you have both gateways, you can then establish an encrypted VPN connection between the two sides. 

## Amazon VPC Routing and Security

<a name='route_table'>

### The Main Route Table
When you create a VPC, AWS creates a route table called the main route table. A route table contains a set of rules, called routes, that are used to determine where network traffic is directed. AWS assumes that when you create a new VPC with subnets, you want traffic to flow between them. Therefore, the default configuration of the main route table is to allow traffic between all subnets in the local network. Below is an example of a main route table:   
![route-table](/images/route-table.png)
<br>

### Custom Route Tables
While the main route table controls the routing for your VPC, you may want to be more granular about how you route your traffic for specific subnets. For example, your application may consist of a frontend and a database. You can create separate subnets for these resources and provide different routes for each of them.

If you associate a custom route table with a subnet, the subnet will use it instead of the main route table. By default, each custom route table you create will have the local route already inside it, allowing communication to flow between all resources and subnets inside the VPC. 
![custom-route-table](/images/custom-route-table.png)
<br>

<a name='acl_sec'></a>

### Secure Your Subnets with Network ACLs
Think of a network ACL as a firewall at the subnet level. A network ACL enables you to control what kind of traffic is allowed to enter or leave your subnet. You can configure this by setting up rules that define what you want to filter. Here’s an example.
<table><tbody><tr><td><p><span><strong><span>Inbound</span></strong></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td></tr><tr><td><p><span><strong><span>Rule #</span></strong></span></p></td><td><p><span><strong><span>Type</span></strong></span></p></td><td><p><span><strong><span>Protocol</span></strong></span></p></td><td><p><span><strong><span>Port Range</span></strong></span></p></td><td><p><span><strong><span>Source</span></strong></span></p></td><td><p><span><strong><span>Allow/Deny</span></strong></span></p></td></tr><tr><td><p><span><span>100</span></span></p></td><td><p><span><span>All IPv4 traffic</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>0.0.0.0/0</span></span></p></td><td><p><span><span>ALLOW</span></span></p></td></tr><tr><td><p><span><span>*</span></span></p></td><td><p><span><span>All IPv4 traffic</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>0.0.0.0/0</span></span></p></td><td><p><span><span>DENY</span></span></p></td></tr><tr><td><p><span><strong><span>Outbound</span></strong></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td></tr><tr><td><p><span><strong><span>Rule #</span></strong></span></p></td><td><p><span><strong><span>Type</span></strong></span></p></td><td><p><span><strong><span>Protocol</span></strong></span></p></td><td><p><span><strong><span>Port Range</span></strong></span></p></td><td><p><span><strong><span>Source</span></strong></span></p></td><td><p><span><strong><span>Allow/Deny</span></strong></span></p></td></tr><tr><td><p><span><span>100</span></span></p></td><td><p><span><span>All IPv4 traffic</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>0.0.0.0/0</span></span></p></td><td><p><span><span>ALLOW</span></span></p></td></tr><tr><td><p><span><span>*</span></span></p></td><td><p><span><span>All IPv4 traffic</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>0.0.0.0/0</span></span></p></td><td><p><span><span>DENY</span></span></p></td></tr></tbody></table>

The default network ACL, shown in the table above, allows all traffic in and out of your subnet. To allow data to flow freely to your subnet, this is a good starting place.   However, you may want to restrict data at the subnet level. For example, if you have a web application, you might restrict your network to allow HTTPS traffic and remote desktop protocol (RDP) traffic to your web servers.

<table><tbody><tr><td><p><span><strong><span>Inbound</span></strong></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td></tr><tr><td><p><span><strong><span>Rule #</span></strong></span></p></td><td><p><span><strong><span>Source IP</span></strong></span></p></td><td><p><span><strong><span>Protocol</span></strong></span></p></td><td><p><span><strong><span>Port</span></strong></span></p></td><td><p><span><strong><span>Allow/Deny</span></strong></span></p></td><td><p><span><strong><span>Comments</span></strong></span></p></td></tr><tr><td><p><span><span>100</span></span></p></td><td><p><span><span>All IPv4 traffic</span></span></p></td><td><p><span><span>TCP</span></span></p></td><td><p><span><span>443</span></span></p></td><td><p><span><span>ALLOW</span></span></p></td><td><p><span><span>Allows inbound HTTPS traffic from anywhere</span></span></p></td></tr><tr><td><p><span><span>130</span></span></p></td><td><p><span><span>192.0.2.0/24</span></span></p></td><td><p><span><span>TCP</span></span></p></td><td><p><span><span>3389</span></span></p></td><td><p><span><span>ALLOW</span></span></p></td><td><p><span><span>Allows inbound RDP traffic to the web servers from your home network’s public IP address range (over the internet gateway)</span></span></p></td></tr><tr><td><p><span><span>*</span></span></p></td><td><p><span><span>All IPv4 traffic</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>DENY</span></span></p></td><td><p><span><span>Denies all inbound traffic not already handled by a preceding rule (not modifiable)</span></span></p></td></tr><tr><td><p><span><strong><span>Outbound</span></strong></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td></tr><tr><td><p><span><strong><span>Rule #</span></strong></span></p></td><td><p><span><strong><span>Destination IP</span></strong></span></p></td><td><p><span><strong><span>Protocol</span></strong></span></p></td><td><p><span><strong><span>Port</span></strong></span></p></td><td><p><span><strong><span>Allow/Deny</span></strong></span></p></td><td><p><span><strong><span>Comments</span></strong></span></p></td></tr><tr><td><p><span><span>120</span></span></p></td><td><p><span><span>0.0.0.0/0</span></span></p></td><td><p><span><span>TCP</span></span></p></td><td><p><span><span>1025-65535</span></span></p></td><td><p><span><span>ALLOW</span></span></p></td><td><p><span><span>Allows outbound responses to clients on the internet (serving people visiting the web servers in the subnet)</span></span></p></td></tr><tr><td><p><span><span>*</span></span></p></td><td><p><span><span>0.0.0.0/0</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>All</span></span></p></td><td><p><span><span>DENY</span></span></p></td><td><p><span><span>Denies all outbound traffic not already handled by a preceding rule (not modifiable)</span></span></p></td></tr></tbody></table>

Notice that in the network ACL example above, you allow inbound 443 and outbound range 1025-65535. That’s because HTTP uses port 443 to initiate a connection and will respond to an ephemeral port. Network ACL’s are considered stateless, so you need to include both the inbound and outbound ports used for the protocol. If you don’t include the outbound range, your server would respond but the traffic would never leave the subnet.   

Since network ACLs are configured by default to allow incoming and outgoing traffic, you don’t need to change their initial settings unless you need additional security layers.

### Secure Your EC2 Instances with Security Groups
The next layer of security is for your EC2 Instances. Here, you can create a firewall called a security group. The default configuration of a security group blocks all inbound traffic and allows all outbound traffic.  
![security-groups](/images/sec-group.png)
<br>

You might be wondering: “Wouldn’t this block all EC2 instances from receiving the response of any customer requests?” Well, security groups are stateful, meaning they will remember if a connection is originally initiated by the EC2 instance or from the outside and temporarily allow traffic to respond without having to modify the inbound rules.   

If you want your EC2 instance to accept traffic from the internet, you’ll need to open up inbound ports. If you have a web server, you may need to accept HTTP and HTTPS requests to allow that type of traffic in through your security group. You can create an inbound rule that will allow port 80 (HTTP) and port 443 (HTTPS) as shown below. 

<table><tbody><tr><td><p><span><strong><span>Inbound rules</span></strong></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td><td><p><span><span></span></span></p></td></tr><tr><td><p><span><strong><span>Type</span></strong></span></p></td><td><p><span><strong><span>Protocol</span></strong></span></p></td><td><p><span><strong><span>Port Range</span></strong></span></p></td><td><p><span><strong><span>Source</span></strong></span></p></td></tr><tr><td><p><span><span>HTTP (80)</span></span></p></td><td><p><span><span>TCP (6)</span></span></p></td><td><p><span><span>80</span></span></p></td><td><p><span><span>0.0.0.0/0</span></span></p></td></tr><tr><td><p><span><span>HTTP (80)</span></span></p></td><td><p><span><span>TCP (6)</span></span></p></td><td><p><span><span>80</span></span></p></td><td><p><span><span>::/0</span></span></p></td></tr><tr><td><p><span><span>HTTPS (443)</span></span></p></td><td><p><span><span>TCP (6)</span></span></p></td><td><p><span><span>443</span></span></p></td><td><p><span><span>0.0.0.0/0</span></span></p></td></tr><tr><td><p><span><span>HTTPS (443)</span></span></p></td><td><p><span><span>TCP (6)</span></span></p></td><td><p><span><span>443</span></span></p></td><td><p><span><span>::/0</span></span></p></td></tr></tbody></table>

You learned in a previous unit that subnets can be used to segregate traffic between computers in your network. Security groups can be used to do the same thing. A common design pattern is organizing your resources into different groups and creating security groups for each to control network communication between them.   
![sec-group-2](/images/sec-group-2.png)
<br>

This example allows you to define three tiers and isolate each tier with the security group rules you define. In this case, you only allow internet traffic to the web tier over HTTPS, Web Tier to Application Tier over HTTP, and Application tier to Database tier over MySQL. This is different from traditional on-premises environments, in which you isolate groups of resources via VLAN configuration. In AWS, security groups allow you to achieve the same isolation without tying it to your network. 

<a name='excercise'></a>

## Exercise 4: Setting up a VPC

In this scenario, you create the underlying network infrastructure where the EC2 instance that hosts the employee directory will live.

In this exercise, you set up a new virtual private cloud (VPC). This new VPC will have four subnets (two public subnets and two private subnets) and two route tables (one public route table and one private route table). Then, you launch an EC2 instance inside the new VPC. Finally, at the end of the exercise, you stop the instance to prevent future costs from incurring.

### Task 1: Creating the VPC
In this task, you will create a new VPC.
1. If needed, log in to the AWS Management Console as your Admin user.
2. In the Services search box, enter VPC and open the VPC console by choosing VPC from the list.
3. In the navigation pane, under Virtual private cloud, choose Your VPCs.
4. Choose Create VPC.
5. Configure these settings:
    - Name tag: app-vpc
    - IPv4 CIDR block: 10.1.0.0/16
6. Choose Create VPC.
7. In the navigation pane, under Virtual private cloud, choose Internet gateways
8. Choose Create internet gateway.
9. For Name tag, paste app-igw and choose Create internet gateway.
10. In the details page for the internet gateway, choose Actions and then choose Attach to VPC.
11. For Available VPCs, choose app-vpc and then choose Attach internet gateway.

### Task 2: Creating subnets
In this task, you will create the four subnets for your VPC. You will configure the two public subnets first, and then configure the two private subnets.

1. From the navigation pane, choose Subnets.
2. Choose Create subnet.
3. For the first public subnet, configure these settings:
    - VPC ID: app-vpc
    - Subnet name:Public Subnet 1
    - Availability Zone: Choose the first Availability Zone
    - Example: If you are in US West (Oregon), you would choose us-west-2a
    - IPv4 CIDR block: 10.1.1.0/24
4. Choose Add new subnet.

5. For the second public subnet, configure these settings:
    - Subnet name: Public Subnet 2
    - Availability Zone: Choose the second Availability Zone
        - Example: If you are in US West (Oregon), you would choose us-west-2b
    - IPv4 CIDR block: 10.1.2.0/24
6. Choose Add new subnet and for the first private subnet, configure these settings:
    - Subnet name: Private Subnet 1
    - Availability Zone: Choose the first Availability Zone
        - Example: If you are in US West (Oregon), you would choose us-west-2a
    - IPv4 CIDR block: 10.1.3.0/24.
7. Choose Add new subnet and for the second private subnet, configure the following:
    - Subnet name: Private Subnet 2
    - Availability Zone: Choose the second Availability Zone
        - Example: If you are in US West (Oregon), you would choose us-west-2b
    - Pv4 CIDR block: 10.1.4.0/24
8. Finally, choose Create subnet.
9. After the subnets are created, select the check box for Public Subnet 1.
10. Choose Actions and then choose Edit subnet settings.
11. For Auto-assign IP settings, select Enable auto-assign public IPv4 address and then choose Save.
12. Clear the check box for Public Subnet 1 and select the check box for Public Subnet 2.
13. Again, choose Actions and then Edit subnet settings.
14. For Auto-assign IP settings, select Enable auto-assign public IPv4 address and save the settings.

### Task 3: Creating route tables
In this task, you will create the route tables for your VPC.

First, you will create the public route table.

1. In the navigation pane, choose Route Tables.

2. Choose Create route table.

3. For the route table, configure these settings:
    - Name: app-routetable-public
    - VPC: app-vpc
4. Choose Create route table.

5. If needed, open the route table details pane by choosing app-routetable-public from the list.

6. Choose the Routes tab and choose Edit routes.

7. Choose Add route and configure these settings:
    - Destination: 0.0.0.0/0
    - Target: Internet Gateway, then choose app-igw (which you set up in the VPC task)
8. Choose Save changes.
9. Choose the Subnet associations tab.
10. Scroll to Subnets without explicit associations and choose Edit subnet associations.
11. Select the two public subnets that you created (Public Subnet 1 and Public Subnet 2) and choose Save associations. Next, you will create the private route table.
12. In the navigation pane, choose Route Tables.
13. Choose Create route table and configure these settings:
    - Name: app-routetable-private
    - VPC: app-vpc
14. Choose Create route table.

15. If needed, open the details pane for app-routetable-private by choosing it from the list.

16. Choose the Subnet associations tab.

17. Scroll to Subnets without explicit associations and choose Edit subnet associations.

18. Select the two private subnets (Private Subnet 1 and Private Subnet 2) and choose Save associations.

### Task 4: Launching an EC2 instance that uses a role
Now that you have created a network, you will launch your EC2 instance by using the VPC that you created!

1. In the search box, enter EC2 and open the Amazon EC2 console by choosing EC2 from the list.

2. In the navigation pane, choose Instances and choose Launch instances.

3. For Name use employee-directory-app.

4. Under Application and OS Images (Amazon Machine Image), choose the default Amazon Linux 2023.

5. Under Instance type, select t2.micro.

6. Under Key pair (login) choose the app-key-pair created in exercise-3.

7. Configure the following settings under Network settings and Edit.
    - VPC: app-vpc
    - Subnet: Public Subnet 1
    - Auto-assign Public IP: Enable
8. Under Firewall (security groups) choose Create security group. Use web-security-group as the Security group name and change Description to Enable HTTP access.

9. Under Inbound security groups rules choose Remove above the ssh rule.

10. Choose Add security group rule. For Type choose HTTP. Under Source type choose Anywhere.

11. Choose Add security group rule. For Type choose HTTPS. Under Source type choose Anywhere.

12. Expand Advanced details and under IAM instance profile choose S3DynamoDBFullAccessRole.

13. In the User data box, paste the following code:
```
#!/bin/bash -ex
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/DEV-AWS-MO-GCNv2/FlaskApp.zip
unzip FlaskApp.zip
cd FlaskApp/
yum -y install python3-pip
pip install -r requirements.txt
yum -y install stress
export PHOTOS_BUCKET=${SUB_PHOTOS_BUCKET}
export AWS_DEFAULT_REGION=ap-southeast-1a
export DYNAMO_MODE=on
FLASK_APP=application.py /usr/local/bin/flask run --host=0.0.0.0 --port=80
```
14. Change the following line to match your Region (the Region is listed at the top right, next to your user name):
15. Choose Launch instance.
16. Choose View all instances.
    -  The instance should now be listed under Instances.
17. Wait for the Instance state to change to Running and the Status check to change to 2/2 checks passed.
    - Note: Often, the status checks update, but the console user interface (UI) might not update to reflect the most recent information. You can minimize waiting by refreshing the page after a few minutes.
18. Select the running employee-directory-app instance by selecting its check box.
19. On the Details tab, copy the Public IPv4 address.
    - Note: Make sure that you only copy the address instead of choosing the open address link.
20. In a new browser window, paste the IP address that you copied. Make sure to remove the ‘S’ after HTTP so you are using only HTTP instead.
21. In a new browser window, paste the IP address that you copied.

### Task 5: Stopping the instance
Congratulations! You have launched an EC2 instance that hosts your employee directory application into a customized VPC.
To prevent future costs, you will now stop the instance.
- Note: Do not terminate this instance because you will use it in a later exercise.
    - Return to the console, choose Instance state, and choose Stop instance.
    -In the dialog box, choose Stop.

The Instance state will eventually go into the Stopped state.