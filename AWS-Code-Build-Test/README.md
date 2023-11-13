# What is DevOps?
DevOps is the combination of cultural philosophies, practices, and tools that increases an organization’s ability to deliver applications and services at high velocity. Put simply, DevOps is a way for development teams and operations teams to work better together and ultimately share the responsibility of the software they build.


# AWS CodeCommit
AWS CodeCommit is a secure, highly scalable, managed source-control service that hosts private Git repositories.

For more information about what CodeCommit is and how it works, see: 
AWS CodeCommit User Guide

After a repository is created in CodeCommit, you can securely interact with it with the proper authentication and access permissions.

# Cloud9
Cloud9 is an integrated development environment (IDE) that allows developers to write, run, and debug code in the cloud. It provides a collaborative environment where multiple developers can work on the same project simultaneously, facilitating teamwork and code sharing. Cloud9 supports various programming languages, and it comes with features such as code highlighting, autocompletion, and version control integration.

One notable aspect of Cloud9 is that it is entirely web-based, meaning you can access your development environment through a web browser without the need to install any software locally. This makes it convenient for developers who may need to switch between different computers or work on projects from various locations.

In 2016, Amazon Web Services (AWS) acquired Cloud9, and it is now part of the AWS ecosystem. AWS Cloud9 is particularly well-integrated with other AWS services, making it easier for developers to deploy and manage applications on the AWS cloud platform.

# Testing

To ensure quality, it’s important for teams to incorporate testing into the software development lifecycle at different stages of the continuous integration and continuous delivery (CI/CD) pipeline. Overall, testing should start as early as possible. 

While there are many kinds of application tests, the three mentioned in this week are:
- Regression testing - Tests to ensure that previously developed applications don’t break with new changes
- Integration testing - Tests to ensure separately developed modules work together as expected
- Unit testing - Tests a specific section of code to ensure the code does what it is expected to do

# Automate testing

AWS CodeBuild will provision a clean and consistent environment to perform various application tests. You can also create reports in CodeBuild that contain details about tests that are run during builds. 

# AWS CodePipeline

You can automate your release process by using AWS CodePipeline to test your code and run your builds with CodeBuild. 

CodePipeline is a continuous delivery (CD) service that you can use to model, visualize, and automate the steps required to release your code. This process includes building your code. A pipeline is a workflow construct that describes how code changes go through a release process.