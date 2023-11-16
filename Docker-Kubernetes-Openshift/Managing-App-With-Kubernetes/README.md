# Table Of Contents
- [Replica Set](#replica)
- [Autoscaling](#autoscaling)
- [Deployment Strategies](#deployment)
- [Rolling Updates](#rollupdate)
- [ConfigMaps](#configmap)
- [service binding](#serviceBinding)
- [Excercise: Scaling and Updating Applications](#excercise)

<a name="replica"></a>

# ReplicaSet
if an application is deployed on a single pod, the pod will be unable to perform on a single pod, the pod will be unable to perform  certain actions if requests increase manifold or outages occur.
Single Pod Deployments cannot:
- Accomodate growing demands
- Handle Outages
- Minimize downtime
- Auto Restart on Interruptions

the `Replica set` ensures the right number of pods are always up and running. It always tries to match the actual state of the replicas to the desired state.
A Replica Set:
- Adds or deletes pods for scaling and redundancy
- Replaces failing pods or deletes additional pods to maintain the desires state
- Supersedes ReplicaController

Replica set:
- Kubernetes keep object types independent
- A replica set does not own pods. is uses pod labels.

## Create Replicaset from scratch
- ReplicaSet:
- Make sure to create the configuration file in yaml and then specified the `kind` property and number of replicas:
    ![replica](/images/replicaset-1.png)<br>
    - execute the yaml configuration file:
        ```
        kubectl create -f replicaset.yaml
        ```
    - see the pod list:
        ```
        kubectl get pods
        ```
    - see the replicaset list:
        ```
        kubectl get rs
        ```
- Deployment and Pod:
    - before we can scale the deployment, we must ensure we have a deployment and pod. execute the deployment configuration file
        ```
        kubectl create -f deployment.yaml
        ```
    - see the pod list:
        ```
        kubectl get pods
        ```
    - see the deployment list:
        ```
        kubectl get deploy
        ```
- Once the deployment and pod are in place, we can scale our deployment:
    ```
    kubectl scale deploy <pod name> --replicas=3
    ```
    the replicaset created two new pods.


<a name="autoscaling"></a>

# Autoscaling

Autoscaling enables scaling as needed. Autoscaling occurs at two different layers:
1. Cluster level
2. Pod level

There are three type Autoscaler:
1. Horizontal Pod Autoscaler (increase or decrease the number of pods)
2. Vertical Pod Autoscaler (increase or decrease the resource size or speed of the pods)
3. CCluster Autoscaler (increase or decrease in relation to the capacity of existing nodes)

## Create autoscaling with kubectl
to Create autoscaling for a deployment or ReplicaSet:
1. Ensure that you have a pod. checking the running pod
2. Ensure that you have a replicaset from your pod
3. To autoscale use the specified command:
    ```
    kubectl autoscale deploy <name> --min=2 --max=5 --cpu-percent=50
    ```

## Autoscaling with HPA from Scratch
![img](/images/hpa.png)<br>


<a name="deployment"></a>

# Deployment Strategies 
A Kubernetes deployment strategy defines an application’s lifecycle that achieves and maintains the configured state for objects and applications in an automated manner. Effective deployment strategies minimize risk.

Kubernetes deployment strategies are used to: 
- Deploy, update, or rollback ReplicaSets, Pods, Services, and Applications 
- Pause/Resume Deployments 
- Scale Deployments manually or automatically 

## Types of deployment strategies
The following are six types of deployment strategies:
1. Recreate 
2. Rolling 
3. Blue/green 
4. Canary 
5. A/B testing 
6. Shadow 

### Recreate strategy 
![img](/images/S0OHGQciTMiDhxkHIuzIZw_59fd9b955e3749cd8653f50a997075f1_Recreate-Strategy.png)<br>
In the recreate strategy, Pods running the live version of the application are all shut down simultaneously, and a new version of the application is deployed on newly created Pods. 

Recreate is the simplest deployment strategy. There is a short downtime between the shutdown of the existing deployment and the new deployment. 

Recreate strategy steps include:
1. A new version of the application (v2) is ready for deployment. 
2. All Pods running the current version (v1) are shut down or deleted. 
3. New (v2) Pods are created. 

### Rolling (ramped) strategy 
![img](/images/ramped.png)<br>
In a rolling strategy, each Pod is updated one at a time. A single v1 Pod is replaced with a new v2 Pod. Each v1 Pod is updated in this way until all Pods are v2. During a rolling strategy update, there is hardly any downtime since users are directed to either version.

Rolling strategy steps include:
1. A new version of the application (v2) is ready for deployment. 
2. One of the Pods running the current version (v1) is shut down or deleted. 
3. A new (v2) Pod is created to replace the (v1) Pod that was removed. 
4. Steps 2 and 3 are repeated until all (v1) Pods are removed and replaced with (v2) Pods. 

### Blue/green strategy
![img](/images/blue-green.png)<br>
In a blue/green strategy, the blue environment is the live version of the application. The green environment is an exact copy that contains the deployment of the new version of the application. The green environment is thoroughly tested. Once all changes, bugs, and issues are addressed, user traffic is switched from the blue environment to the green environment.

Blue/green strategy steps include:
1. Create a new environment identical to the current production environment. 
2. Design the new version and test it thoroughly until it is ready for production. 
3. Route all user traffic to the new version. 

### Canary strategy
![img](/images/canary.png)<br>
In a canary strategy, the new version of the application is tested using a small set of random users alongside the current live version of the application. Once the new version of the application is successfully tested, it is then rolled out to all users. 

Canary strategy steps include:
1. Design a new version of the application. 
2. Route a small sample of user requests to the new version. 
3. Test for efficiency, performance, bugs, and issues, and rollback as needed. 
4. Repeat steps 1 to 3. Once all issues are resolved, route all traffic to the new version.  

### A/B testing strategy
![img](/images/a-b.png)<br>
The A/B testing strategy, also known as split testing, evaluates two versions of an application (version A and version B). With A/B testing, each version has features that cater to different sets of users. You can select which version is best for global deployment based on user interaction and feedback. 

A/B testing strategy steps include:
1. Design a new version of the application by adding mostly UI features. 
2. Identify a small set of users based on conditions like weight, cookie value, query parameters, geolocalization, browser version, screen size, operating system, and language. 
3. Route requests from the user set to the new version. 
4. Check for bugs, efficiency, performance, and issues. 
5. Once all issues are resolved, route all traffic to the new version. 

### Shadow Strategy
![img](/images/shado.png)<br>
In a shadow strategy, a “shadow version” of the application is deployed alongside the live version. User requests are sent to both versions, and both handle all requests, but the shadow version does not forward responses back to the users. This lets developers see how the shadow version performs using real-world data without interrupting user experience.

<a name="rollupdate"></a>

# Rolling Updates
To prepare your applications to enable rolling updates,
1. Step 1 is to Add `Liveness probes` and `Readiness Probes` to deployments 
    ![probe](/images/probe.png)<br>
2. Add rolling updates strategy to your YAML file
    ![rollingupdate](/images/rollingupdates.png)<br>
3. lets look at working example of rolling out an update on your application
    - the scenario is, we have a new image of our application with a different message
    - to tackle this we need to create our new image version from our current application by upload our new image to docker hub
4. the new image will be apply to our deployment
    - set the iamge flag to updated tag image on Docker hub
        ```
        kubectl set iamge deployments/hello-kubernetes hello-kubernetes=upkar/hello-kubernetes:2.0
        ```
    - Now, our version2 is deployed.
5. see the `rolled out` status our version2 that been deployed from the previous
    ```
    kubectl rollout status deployments/hello-kubernetes
    ```
6. if your application has been error, or client changes their mind we can `rollback` to our previous version by doing:
    ```
    kubectl rollout undo deployments/hello-kubernetes
    ```

## Practical Demos
1. All-At-Once rollout
    - All previous version must be removed before new version can become active.
2. All-At-Once rollback
    - All curent version must be removed before previous version can become active
3. One-At-A-Time rollout
    - the update is staggered so user access is not interrupted
4. One-At-A-Time rollback
    - the update rollback is staggered so user access is not interrupted

<a name='configmap'></a>

# ConfigMaps And Secrets
![env](/images/env.png)<br>

<a name="serviceBinding"></a>

# Service Binding
- Is a process to consume external services or  backing services
- Manages configuration and credentials for back-end SServices whileprotecting sensitive data
- Make services credentials available to you automatically as a secret
- Consume the external services by binding the application to a deployment

<a name="excercise"></a>

# Excercise: Scaling and Updating Applications
In this lab, you will:
- Scale an application with a ReplicaSet
- Apply rolling updates to an application
- Use a ConfigMap to store application configuration
- Autoscale the application using Horizontal Pod Autoscaler

## Build and push application image to IBM Cloud Container Registry
1. Use the Explorer to view the Dockerfile that will be used to build an image.
    ![exec-3](/images/exec-3.png)<br>
2. Build and push the image again, as it may have been deleted automatically since you completed the first lab.
    ```
    docker build -t us.icr.io/$MY_NAMESPACE/hello-world:1 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:1
    ```

## Deploy the application to Kubernetes
1. Use the Explorer to edit deployment.yaml in this directory.
2. Run your image as a Deployment.
    ```
    kubectl apply -f deployment.yaml
    ```
3. List Pods until the status is “Running”.
    ```
    kubectl get pod
    ```
4. In order to access the application, we have to expose it to the internet via a Kubernetes Service.
    ```
    kubectl expose deployment/hello-world
    ```
5. Open a new terminal window using Terminal > New Terminal
6. Cluster IPs are only accesible within the cluster. To make this externally accessible, we will create a proxy.
    ```
    kubectl proxy
    ```
    This command will continue running until it exits. Keep it running so that you can continue to access your app.
7. Go back to your original terminal window, ping the application to get a response.
    ```
    curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
    ```
    Observe the message “Hello world from hello-world-xxxxxxxx-xxxx. Your app is up and running!

## Scaling the application using a ReplicaSet
In real-world situations, load on an application can vary over time. If our application begins experiencing heightened load, we want to scale it up to accommodate that load. There is a simple kubectl command for scaling:
1. Use the scale command to scale up your Deployment. Make sure to run this in the terminal window that is not running the proxy command.
    ```
    kubectl scale deployment hello-world --replicas=3
    ```
2. Get Pods to ensure that there are now three Pods instead of just one. In addition, the status should eventually update to “Running” for all three.
    ```
    kubectl get pods
    ```
3. As you did in the last lab, ping your application multiple times to ensure that Kubernetes is load-balancing across the replicas.
    ```
    for i in `seq 10`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy; done
    ```
4. Similarly, you can use the scale command to scale down your Deployment.
    ```
    kubectl scale deployment hello-world --replicas=1
    ```

## Perform rolling updates
Rolling updates are an easy way to update our application in an automated and controlled fashion. To simulate an update, let’s first build a new version of our application and push it to Container Registry:
1. Use the Explorer to edit app.js. 
    ![image](/images/step_5.1.png)<br>
2. Build and push this new version to Container Registry. Update the tag to indicate that this is a second version of this application. Make sure to use the terminal window that isn’t running the proxy command.
    ```
    docker build -t us.icr.io/$MY_NAMESPACE/hello-world:2 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:2
    ```
3. Update the deployment to use this version instead.
    ```
    kubectl set image deployment/hello-world hello-world=us.icr.io/$MY_NAMESPACE/hello-world:2
    ```
4. You can also get the Deployment with the wide option to see that the new tag is used for the image.
    ```
    kubectl get deployments -o wide
    ```
5. Ping your application to ensure that the new welcome message is displayed.
    ```
    curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
    ```
6. It’s possible that a new version of an application contains a bug. In that case, Kubernetes can roll back the Deployment like this:
    ```
    kubectl rollout undo deployment/hello-world
    ```
7. Get the Deployment with the wide option to see that the old tag is used. Look for the IMAGES column and ensure that the tag is 1.

## Using a ConfigMap to store configuration
ConfigMaps and Secrets are used to store configuration information separate from the code so that nothing is hardcoded. It also lets the application pick up configuration changes without needing to be redeployed. To demonstrate this, we’ll store the application’s message in a ConfigMap so that the message can be updated simply by updating the ConfigMap:
1. Create a ConfigMap that contains a new message.
    ```
    kubectl create configmap app-config --from-literal=MESSAGE="This message came from a ConfigMap!"
    ```
2. Use the Explorer to edit deployment-configmap-env-var.yaml.
    ![img](/images/step_6.2.png)<br>
3. In the same file, notice the section reproduced below. The bottom portion indicates that environment variables should be defined in the container from the data in a ConfigMap named app-config.
    ```
    containers:
    - name: hello-world
      image: us.icr.io/<my_namespace>/hello-world:3
      ports:
      - containerPort: 8080
    envFrom:
    - configMapRef:
      name: app-config
    ```
4. Use the Explorer to open the app.js and edit the file
    ![img](/images/step_6.4.png)<br>
    Make sure to save the file when you’re done. This change indicates that requests to the app will return the environment variable MESSAGE.
5. Build and push a new image that contains your new application code.
    ```
    docker build -t us.icr.io/$MY_NAMESPACE/hello-world:3 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:3
    ```
6. Apply the new Deployment configuration.
    ```
    kubectl apply -f deployment-configmap-env-var.yaml
    ```
    `Note`:The deployment-configmap-env-var.yaml file is already configured to use the tag 3.
7. Ping your application again to see if the message from the environment variable is returned.
    ```
    curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
    ```
    If you see the message, “This message came from a ConfigMap!”, then great job!
8. Because the configuration is separate from the code, the message can be changed without rebuilding the image. Using the following command, delete the old ConfigMap and create a new one with the same name but a different message.
    ```
    kubectl delete configmap app-config && kubectl create configmap app-config --from-literal=MESSAGE="This message is different, and you didn't have to rebuild the image!"
    ```
9. Restart the Deployment so that the containers restart. This is necessary since the environment variables are set at start time.
    ```
    kubectl rollout restart deployment hello-world
    ```
10. Ping your application again to see if the new message from the environment variable is returned.
    ```
    curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
    ```

## Autoscale the hello-world application using Horizontal Pod Autoscaler
1. Please add the following section to the deployment.yaml file under the template.spec.containers section for increasing the CPU resource utilization
    ```
          name: http
        resources:
          limits:
            cpu: 50m
          requests:
            cpu: 20m
    ```
    The updated file will be as below:
    ![examp](/images/cpu_addn--deployment.yaml.png)<br>

2. Apply the deployment:
    ```
    kubectl apply -f deployment.yaml
    ```
3. Autoscale the hello-world deployment using the below command:
    ```
    kubectl autoscale deployment hello-world --cpu-percent=5 --min=1 --max=10
    ```
4. You can check the current status of the newly-made HorizontalPodAutoscaler, by running:
    ```
    kubectl get hpa hello-world
    ```
5. Please ensure that the kubernetes proxy is still running in the 2nd terminal. If it is not, please start it again by running:
    ```
    kubectl proxy
    ```
6. Open another new terminal and enter the below command to spam the app with multiple requests for increasing the load:
    ```
    for i in `seq 100000`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy; done
    ```
7. Run the below command to observe the replicas increase in accordance with the autoscaling:
    ```
    kubectl get hpa hello-world --watch
    ```
8. Run the below command to observe the details of the horizontal pod autoscaler:
    ```
    kubectl get hpa hello-world
    ```
9.  Delete the Deployment
    ```
    kubectl delete deployment hello-world
    ```
10. Delete the Service.
    ```
    kubectl delete service hello-world
    ```