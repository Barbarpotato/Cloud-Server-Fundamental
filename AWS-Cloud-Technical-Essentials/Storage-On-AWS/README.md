# Storage Types on AWS

- Table Of Contents
    - [Block & Object Storage](#storage)
        - [Instance Store](#instant)
        - [Elastic Block Store](#ebs)
        - [Objext Storage S3](#s3)
    - [Choose the Right Storage Service](#choose)
    - [Excercise](#excercise)

## Block Storage

While file storage treats files as a singular unit, block storage splits files into fixed-size chunks of data called blocks that have their own addresses. Since each block is addressable, blocks can be retrieved efficiently.

When data is requested, these addresses are used by the storage system to organize the blocks in the correct order to form a complete file to present back to the requestor. Outside of the address, there is no additional metadata associated with each block. So, when you want to change a character in a file, you just change the block, or the piece of the file, that contains the character. This ease of access is why block storage solutions are fast and use less bandwidth.
![blcok storage](/images/block.png)
<br>
Since block storage is optimized for low-latency operations, it is a typical storage choice for high-performance enterprise workloads, such as databases or enterprise resource planning (ERP) systems, that require low-latency storage.

## Object Storage

Objects, much like files, are also treated as a single unit of data when stored. However, unlike file storage, these objects are stored in a flat structure instead of a hierarchy. Each object is a file with a unique identifier. This identifier, along with any additional metadata, is bundled with the data and stored.

Changing just one character in an object is more difficult than with block storage. When you want to change one character in a file, the entire file must be updated.
![object storage](/images/object.png)
<br>
With object storage, you can store almost any type of data, and there is no limit to the number of objects stored, making it easy to scale. Object storage is generally useful when storing large data sets, unstructured files like media assets, and static assets, such as photos.

## Amazon EC2 Instance Storage and Amazon Elastic Block Store

<a name='instant'></a>

### Amazon EC2 Instance Store 
Amazon EC2 Instance Store provides temporary block-level storage for your instance. This storage is located on disks that are physically attached to the host computer. This ties the lifecycle of your data to the lifecycle of your EC2 instance. If you delete your instance, the instance store is deleted as well. Due to this, instance store is considered ephemeral storage.

Instance store is ideal if you are hosting applications that replicate data to other EC2 instances, such as Hadoop clusters. For these cluster-based workloads, having the speed of locally attached volumes and the resiliency of replicated data helps you achieve data distribution at high performance. It’s also ideal for temporary storage of information that changes frequently, such as buffers, caches, scratch data, and other temporary content. 

<a name='ebs'></a>

### Amazon Elastic Block Storage (Amazon EBS)  
As the name implies, Amazon EBS is a block-level storage device that you can attach to an Amazon EC2 instance. These storage devices are called Amazon EBS volumes. EBS volumes are essentially drives of a user-configured size attached to an EC2 instance, similar to how you might attach an external drive to your laptop. 

EBS volumes act similarly to external drives in more than one way.
- Most Amazon EBS volumes can only be connected with one computer at a time. Most EBS volumes have a one-to-one relationship with EC2 instances, so they cannot be shared by or attached to multiple instances at one time. Note: Recently, AWS announced the Amazon EBS multi-attach feature that enables volumes to be attached to multiple EC2 instances at one time. This feature is not available for all instance types and all instances must be in the same Availability Zone.
- You can detach an EBS volume from one EC2 instance and attach it to another EC2 instance in the same Availability Zone, to access the data on it.
- The external drive is separate from the computer. That means, if an accident happens and the computer goes down, you still have your data on your external drive. The same is true for EBS volumes.
- You’re limited to the size of the external drive, since it has a fixed limit to how scalable it can be. For example, you may have a 2 TB external drive and that means you can only have 2 TB of content on there. This relates to EBS as well, since volumes also have a max limitation of how much content you can store on the volume.

### Scale Amazon EBS Volumes
You can scale Amazon EBS volumes in two ways. 
1. Increase the volume size, as long as it doesn’t increase above the maximum size limit. For EBS volumes, the maximum amount of storage you can have is 16 TB. That means if you provision a 5 TB EBS volume, you can choose to increase the size of your volume until you get to 16 TB.
2. Attach multiple volumes to a single Amazon EC2 instance. EC2 has a one-to-many relationship with EBS volumes. You can add these additional volumes during or after EC2 instance creation to provide more storage capacity for your hosts.

### Amazon EBS Use Cases
Amazon EBS is useful when you need to retrieve data quickly and have data persist long-term. Volumes are commonly used in the following scenarios. 

- Operating systems: Boot/root volumes to store an operating system. The root device for an instance launched from an Amazon Machine Image (AMI) is typically an Amazon EBS volume. These are commonly referred to as EBS-backed AMIs. 
- Databases: A storage layer for databases running on Amazon EC2 that rely on transactional reads and writes.
- Enterprise applications: Amazon EBS provides reliable block storage to run business-critical applications.
- Throughput-intensive applications: Applications that perform long, continuous reads and writes.

### Amazon EBS Volume Types
There are two main categories of Amazon EBS volumes: solid-state drives (SSDs) and hard-disk drives (HDDs). SSDs provide strong performance for random input/output (I/O), while HDDs provide strong performance for sequential I/O. AWS offers two types of each. The following chart can help you decide which EBS volume is the right option for your workload.

### Benefits of Using Amazon EBS
Here are the following benefits of using Amazon EBS (in case you need a quick cheat sheet).
- High availability: When you create an EBS volume, it is automatically replicated within its Availability Zone to prevent data loss from single points of failure.
- Data persistence: The storage persists even when your instance doesn’t.
- Data encryption: All EBS volumes support encryption.
- Flexibility: EBS volumes support on-the-fly changes. You can modify volume type, volume size, and input/output operations per second (IOPS) capacity without stopping your instance.
- Backups: Amazon EBS provides you the ability to create backups of any EBS volume

### EBS Snapshots
Errors happen. One of those errors is not backing up data, and then, inevitably losing that data. To prevent this from happening to you, you should back up your data—even in AWS. Since your EBS volumes consist of the data from your Amazon EC2 instance, you’ll want to take backups of these volumes, called snapshots. 

EBS snapshots are incremental backups that only save the blocks on the volume that have changed after your most recent snapshot. For example, if you have 10 GB of data on a volume, and only 2 GB of data have been modified since your last snapshot, only the 2 GB that have been changed are written to Amazon Simple Storage Service (Amazon S3). 

When you take a snapshot of any of your EBS volumes, these backups are stored redundantly in multiple Availability Zones using Amazon S3. This aspect of storing the backup in Amazon S3 will be handled by AWS, so you won’t need to interact with Amazon S3 to work with your EBS snapshots. You simply manage them in the EBS console (which is part of the EC2 console). 

EBS snapshots can be used to create multiple new volumes, whether they’re in the same Availability Zone or a different one. When you create a new volume from a snapshot, it’s an exact copy of the original volume at the time the snapshot was taken. 

<a name='s3'></a>

## Object Storage with Amazon S3

### WHAT IS AMAZON S3?

Unlike Amazon EBS, Amazon S3 is a standalone storage solution that isn’t tied to compute. It enables you to retrieve your data from anywhere on the web. If you’ve ever used an online storage service to back up the data from your local machine, then you most likely have used a service similar to Amazon S3. The big difference between those online storage services and Amazon S3 is the storage type.

Amazon S3 is an object storage service. Object storage stores data in a flat structure, using unique identifiers to look up objects when requested. An object is simply a file combined with metadata and that you can store as many of these objects as you’d like. All of these characteristics of object storage are also characteristics of Amazon S3. 

### UNDERSTAND AMAZON S3 CONCEPTS

In Amazon S3, you have to store your objects in containers called buckets. You can’t upload any object, not even a single photo, to S3 without creating a bucket first. When you create a bucket, you choose, at the very minimum, two things: the bucket name and the AWS Region you want the bucket to reside in. 

 The first part is choosing the Region you want the bucket to reside in. Typically, this will be a Region that you’ve used for other resources, such as your compute. When you choose a Region for your bucket, all objects you put inside that bucket are redundantly stored across multiple devices, across multiple Availability Zones. This level of redundancy is designed to provide Amazon S3 customers with 99.999999999% durability and 99.99% availability for objects over a given year.

The second part is choosing a bucket name which must be unique across all AWS accounts. AWS stops you from choosing a bucket name that has already been chosen by someone else in another AWS account. Once you choose a name, that name is yours and cannot be claimed by anyone else unless you delete that bucket, which then releases the name for others to use. 

AWS uses this name as part of the object identifier. In S3, each object is identified using a URL, which looks like this:
![s3 URL](/images/s3-url.jpeg)
<br>
After the http://, you see the bucket name. In this example, the bucket is named doc. Then, the identifier uses the service name, s3 and specifies the service provider amazonaws. After that, you have an implied folder inside the bucket called 2006-03-01 and the object inside the folder that is named AmazonS3.html. The object name is often referred to as the key name.

`Note`, you can have folders inside of buckets to help you organize objects. However, remember that there’s no actual file hierarchy that supports this on the back end. It is instead a flat structure where all files and folders live at the same level. Using buckets and folders implies a hierarchy, which makes it easy to understand for the human eye.

### S3 USE CASES
Amazon S3 is one of the most widely used storage services, with far more use cases than could fit on one screen. The following list summarizes some of the most common ways you can use Amazon S3.
- Backup and storage: S3 is a natural place to back up files because it is highly redundant. As mentioned in the last unit, AWS stores your EBS snapshots in S3 to take advantage of its high availability.
- Media hosting: Because you can store unlimited objects, and each individual object can be up to 5 TBs, S3 is an ideal location to host video, photo, or music uploads.
- Software delivery: You can use S3 to host your software applications that customers can download.
- Data lakes: S3 is an optimal foundation for a data lake because of its virtually unlimited scalability. You can increase storage from gigabytes to petabytes of content, paying only for what you use.
- Static websites: You can configure your bucket to host a static website of HTML, CSS, and client-side scripts.
- Static content: Because of the limitless scaling, the support for large files, and the fact that you access any object over the web at any time, S3 is the perfect place to store static content.

<a name='choose'></a>

## Choose the Right Storage Service

### Amazon EC2 Instance Store
Instance store is ephemeral block storage. This is preconfigured storage that exists on the same physical server that hosts the EC2 instance and cannot be detached from Amazon EC2. You can think of it as a built-in drive for your EC2 instance. Instance store is generally well-suited for temporary storage of information that is constantly changing, such as buffers, caches, and scratch data. It is not meant for data that is persistent or long-lasting. If you need persistent long-term block storage that can be detached from Amazon EC2 and provide you more management flexibility, such as increasing volume size or creating snapshots, then you should use Amazon EBS. 

### Amazon EBS
Amazon EBS is meant for data that changes frequently and needs to persist through instance stops, terminations, or hardware failures. Amazon EBS has two different types of volumes: 
SSD-backed volumes and HDD-backed volumes.SSD-backed volumes have the following characteristics. 
- Performance depends on IOPS (input/output operations per second).
- Ideal for transactional workloads such as databases and boot volumes.
HDD-backed volumes have the following characteristics: 
- Performance depends on MB/s.
- Ideal for throughput-intensive workloads, such as big data, data warehouses, log processing, and sequential data I/O.

### Amazon S3
If your data doesn’t change that often, Amazon S3 might be a more cost-effective and scalable storage solution. S3 is ideal for storing static web content and media, backups and archiving, data for analytics, and can even be used to host entire static websites with custom domain names.Here are a few important features of Amazon S3 to know about when comparing it to other services. 
- It is object storage.
- You pay for what you use (you don’t have to provision storage in advance).
- Amazon S3 replicates your objects across multiple Availability Zones in a Region.
- Amazon S3 is not storage attached to compute.

<a name='excercise'></a>

## Creating an S3 Bucket and Modifying the EC2 Instance
For this scenario, you create the S3 bucket where the employee photos will be stored.

In this exercise, you create the S3 bucket, upload an object to it, and modify the bucket policy. You also launch an EC2 instance with updated user data so that the application uses the S3 bucket. Finally, you stop the EC2 instance to prevent future costs.

### Task 1: Creating an S3 bucket
In this task, you will create an S3 bucket.

1. If needed, log in to the AWS Management Console with your Admin user.
2. In the search box, enter S3 and open the Amazon S3 console by choosing S3.
3. Choose Create bucket.
4. For Bucket name, enter employee-photo-bucket-<your initials>-<unique number>.
    - Example
    ```
    employee-photo-bucket-al-907
    ```
5. Choose Create bucket.

### Task 2: Uploading a photo
In this task, you will upload an object (a photo) to the S3 bucket.
1. Open the details of your newly created bucket by choosing the bucket name.
2. Choose Upload.
3. Choose Add files.
4. Choose a photo of your choice from your computer and choose Open.
5. Choose Upload.
    - At the top, you should see Upload succeeded in green.
6. Choose Close.

### Task 3: Modifying the S3 bucket policy
In this task, you will update the bucket policy. The updated configuration allows the IAM role that you created previously to access the bucket.
1. Choose the Permissions tab.
2. Scroll down to Bucket policy and choose Edit.
3. In the box, paste the following policy:
    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "AllowS3ReadAccess",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::<INSERT-ACCOUNT-NUMBER>:role/S3DynamoDBFullAccessRole"
                },
                "Action": "s3:*",
                "Resource": [
                    "arn:aws:s3:::<INSERT-BUCKET-NAME>",
                    "arn:aws:s3:::<INSERT-BUCKET-NAME>/*"
                ]
            }
        ]
    }
    ```
4. Replace the <INSERT-ACCOUNT-NUMBER> placeholder with your account number.
5. Replace the <INSERT-BUCKET-NAME> placeholder with your bucket name.
    - Example:
    ```
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "AllowS3ReadAccess",
                "Effect": "Allow",
                "Principal": {
                    "AWS": "arn:aws:iam::123456789012:role/S3DynamoDBFullAccessRole"
                },
                "Action": "s3:*",
                "Resource": [
                    "arn:aws:s3:::employee-photo-bucket-al-907",
                    "arn:aws:s3:::employee-photo-bucket-al-907/*"
                ]
            }
        ]
    }
    ```
6. Choose Save changes.

### Task 4: Modifying the application to use the S3 bucket
In this task, you will launch another EC2 instance. This time, you will modify the user data script so that the application uses the S3 bucket.
1. In the Services search box, enter EC2 and open the service by choosing EC2.
2. In the navigation pane, under Instances, choose Instances.
3. Select the employee-directory-app instance, which should be in the Stopped state.
4. Choose Actions and then choose Image and templates, Launch more like this.
5. For Name and at the end of the Value, append -s3.
    - Example:
    ```
    employee-directory-app-s3
    ```
6. For Key pair name, select app-key-pair.
7. Under Network settings and Auto-assign Public IP, choose Enable.
8. Scroll down and expand Advanced Details.
9. In the User data box, update the values for the PHOTOS_BUCKET variable and (if needed) the AWS_DEFAULT_REGION variable.
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
    - Example:
    This example uses a sample bucket name.
    ```
    export PHOTOS_BUCKET=employee-photo-bucket-al-907
    ```
10. Choose Launch instance.
11. Choose View all instances.
    - The new instance should now be in the Instances list.
12. Wait for the Instance state to change to Running and the Status check to change to 2/2 checks passed.
    Note: You can refresh the page to update the instance status.
13. If needed, clear the check box for the stopped instance that you created previously.
14. Select the check box for the employee-directory-app-s3 instance.
15. Copy the Public IPv4 address.
    - Note: Make sure that you only copy the address instead of choosing the open address link.
16. In a new browser window, paste the IP address that you copied. Make sure to remove the ‘S’ after HTTP so you are using only HTTP instead.

You should see an Employee Directory placeholder. You won’t be able to interact with the application yet because it’s not connected to a database.
Congratulations! You launched an EC2 instance that uses the S3 bucket you created.

### Task 5: Deleting the object from the S3 bucket
In this task, you will delete the object that you uploaded to the S3 bucket.

1. Open the Amazon S3 console by searching for and choosing S3.
2. Open the bucket details by choosing the employee-photo-bucket- link.
3. Select the check box for your object.
4. Choose Delete and confirm the deletion by entering permanently delete.
5. Choose Delete objects and then choose Close.

### Task 6: Stopping your EC2 instance
In this task, you will now stop the instance to prevent future costs.

`Note`: Don’t terminate the instance because you will use this instance in a later exercise.

1. Return to the Amazon EC2 console by searching for and choosing EC2.
2. If needed, in the navigation pane, choose Instances.
3. Select the check box for the employee-directory-app-s3 instance.
4. Choose Instance state and then choose Stop instance.
5. In the dialog box, choose Stop.

The Instance state will eventually go into the Stopped state.
