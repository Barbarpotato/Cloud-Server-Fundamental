# Table Of Contents
- [Deploying Your first application on Code Engine](#first)
- [Deploying Your first Docker Image on Code Engine](#docker)
- [Deploy, Update, and Scale Microservices using Serverless]($codeengine)


<a name="first></a>

# Deploying your first application on Code Engine

In this lab you will learn how to deploy a Hello World web application from GitHub on Code Engine. IBM Cloud Code Engine has been made availble to you through this lab environment.

1. Run the following command to see the list of applications that exist.
```
ibmcloud code-engine application list
```
2. You can alternatively use the short for ibmcloud ce app in the place of ibmcloud code-engine application.
```
ibmcloud ce app list
```
3. You will deploy the simple flask web application which serves one REST API endpoint at the root level and returns the string Hello World. The code is provided in https://github.com/ibm-developer-skills-network/danum-pythonflaskserver. Run the following command to deploy the application.
```
ibmcloud ce application create --name helloworld --build-source https://github.com/ibm-developer-skills-network/danum-pythonflaskserver  --image us.icr.io/${SN_ICR_NAMESPACE}/helloworld --registry-secret icr-secret --port 5000
```
4. You will see that the command creates the application and also internally sets up the required infrastructure. It takes a a few seconds and it finally gives a confirmation along with the URL.

5. Press ctrl(Windows)/cmd(Mac) and the link that is created. Alternatively copy the link and paste it in a browser page and press enter. The hello world application page renders as given below.

6. To get the details of the application, run the following command.
```
ibmcloud ce app get --name helloworld
```

<a name="docker"></a>

# Deploying your first docker image on Code Engine
In this lab, you will learn how to deploy your first Docker image on Code Engine. IBM Cloud Code Engine has been made available to you through this lab environment.

1. Run the following command to see the list of applications that exist.
```
ibmcloud ce app list
```
2. You will clone the code from github, dockerize it and deploy the web application which serves one REST API endpoint at the root level and returns the string Hello World. Run the following command to clone the code.
```
git clone https://github.com/ibm-developer-skills-network/danum-pythonflaskserver
```

3. Change to the cloned directory by running the following command.
```
cd danum-pythonflaskserver
```

4. Now run `docker build` in the current directory and tag the image. Note that in the below command we are naming the app helloworld2 as we may have the earlier instance of helloworld still in the project space.
```
docker build . -t us.icr.io/${SN_ICR_NAMESPACE}/helloworld2
```

5. Now push the image to the namespace so that you can run it.
```
docker push us.icr.io/${SN_ICR_NAMESPACE}/helloworld2
```

6. Now that the image is all set to be deployed, run the following command. Please note that since we already built and pushed the image, we can create the application without mentioning the build source. You will see that the command creates the application and also internally sets up the required infrastructure. It takes a a few seconds and it finally gives a confirmation along with the URL.
```
ibmcloud ce application create --name helloworld2 --image us.icr.io/${SN_ICR_NAMESPACE}/helloworld2 --registry-secret icr-secret --port 5000
```

7. Press ctrl(Windows)/cmd(Mac) and the link that is created. Alternatively copy the link and paste it in a browser page and press enter. The hello world application page renders as given below.

<a name="codeengine"></a>

# Deploy, Update, and Scale Microservices using Serverless
In this lab, you will learn how to create a UK University information application composed of two microservices on Code Engine, and how to scale and update each microservice independently of the other. IBM Cloud Code Engine has been made available to you through this lab environment.

## Deploying the microservices

1. In the Code Engine CLI, clone the repository https://github.com/ibm-developer-skills-network/gentu-microservices_practiceProj.git by running the following command.
```
git clone https://github.com/ibm-developer-skills-network/gentu-microservices_practiceProj.git
```

2. Change to the universities directory in the project that you just cloned.
```
cd gentu-microservices_practiceProj/universities
```

3. List to see the directories inside it each of which have the microservices that we will deploy on Code Engine.
```
ls
```

You will see two directories listed.
- listing
    The listing directory contains the microservice that will list all universities or a list of universities based on keyword search.
- websites
    The websites directory contains the microservice that will provide the list of websites given a college name. You will later update the application to return websites for all colleges that match a keyword search.

4. Run the following command to deploy the listing as a microservice.
```
ibmcloud ce app create --name listing --image us.icr.io/${SN_ICR_NAMESPACE}/listing --registry-secret icr-secret --port 5000 --build-context-dir listing --build-source .
```

5. Now run the following command to get the details about the app. This will also list URL that you can access the endpoints through. Copy the URL.
    ```
    ibmcloud ce app get -n listing
    ```
    <br>
    ![listing details](/images/listing_details.png)<br>

6. Now try to access the endpoint you copied previously by replacing it in the following command. This command will fetch all the colleges which have the name Trinity in it.
```
curl <your deplymenturl>/colleges/Trinity
```

7. Run the following command to deploy the websites as a microservice.
```
ibmcloud ce app create --name websites --image us.icr.io/${SN_ICR_NAMESPACE}/websites --registry-secret icr-secret --port 5000 --build-context-dir websites --build-source .
```

8. Now try to access the endpoint you copied previously by replacing it in the following command. This command will fetch all the websites for the college Trinity College of Music.
```
curl <your deplymenturl>/websites/Trinity%20College%20of%20Music
```

9. There is no keyword search for the websites. Only if the college name exactly matches the value that is passed, the websites are fetched. Try to access the endpoint you copied previously by replacing it in the following command. This command will fetch no websites.
```
curl <your deplymenturl>/websites/Trinity
```
It will return nothing.

## Scale and update the microservices in the application


## Scale 
Unlike a monolithic application, having the application composed of multiple microservices is very advantageous. It allows us to scale the microservices and/or update them independently without affecting the whole application, having to redeploy it or restart it. For example, if one service is seeing a lot of hits then we can scale just that one.

In this example, we will imagine two scenarios:
1. The listings API endpoint gets a lot of hits and has to be scaled.

2. After creating the websites API endpoint, based on user experience, you decide to offer a keyword search instead of an exact word search.

3. Run the following command to scale the listing application to three instances.

```
ibmcloud ce app update --name listing --min 3
```

This will scale the minimum number of instances to 3. At any point in time, there will be 3 instances of the application running.

Run the following command to see if the listing application has scaled.
```
ibmcloud ce app get --name listing -q
```

Now when you run the curl command to access the endpoint, it will be much faster as the instance is already running 3 instances.


## Updating 
You will now update the microservice serving websites API Endpoint to take keyword search as a parameter instead of giving the whole college name.
1. Click on the button below to open app.py in the websites directory.
2. Replace the implementation of the API endpoint, with the following code. This will enable the microservice to provide keyword searches for websites.
```python
from flask import Flask
from flask_cors import CORS
import json

app = Flask("University Websites")
CORS(app)

with open("UK_Universities.json", "r") as unifile:
    data = json.load(unifile)


@app.route("/websites/<name>")
def getCollegesWebsites(name):
    webpages = []
    for college in data:
        if college["name"].__contains__(name):
            for website in college["web_pages"]:
                webpages.append(website)
    return json.dumps({"Webpages": webpages},indent=4)

if __name__ == "__main__":
    app.run(debug=True, port=5001)
```

3. Run the following command to update the application.
```
ibmcloud ce app update --name websites --image us.icr.io/${SN_ICR_NAMESPACE}/websites --registry-secret icr-secret --port 5000 --build-context-dir websites --build-source .
```

4. This will update the existing application. To verify the same run the following command. You may recollect from the previous exercise that this did not return any websites as the previous code did not support keyword search.
```
curl <your deplymenturl>/websites/Trinity
```

5. You will see the websites of all colleges which have Trinity in their name listed.

