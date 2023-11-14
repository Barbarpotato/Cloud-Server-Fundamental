# Table Of Contents
- [Kubernetes Architecture](#architecture)
    - [Mastrer Node (Control Plane)](#master)
    - [Worker Node (Worker Plane)](#worker)
- [Kubernetes Object](#object)
- [kubectl](#kubectl)
    - [Imperative Object Configuration](#imperative)
    - [Declarative Object Configuration](#declarative)
- [Excercise: Create the Pod](#excercise)

# Container Orchestration
is a process that automates the container lifecycle of containerized applications. Including:
1. Deployment
2. Management
3. Scaling
4. Networking
5. Availability

## Container Orchestration Features
- Defines container images and  registry
- Improves provisioning and deployment
- Secures network connection
- Ensure availability and performance
- Manages scalability and load balancing

## How Does Container Orchestration work?
- Container Orchestration uses configuration file written in YAML or JSON. 
- These file configure each container so it can find resources, estabilish network and store logs.
- can automatically schedules the deployment of a new container to a cluster and finds the right host absed on predefined settings or restrictions

# Introduction to Kubernetes

<a name='architecture'></a>

## Kubernetes Architecture
![kubeimage](/images/kube-1.png)<br>
- A deployment of kubernetes is called kubernetes cluster.
    - is a cluster of nodes that runs containerized applications.
        - ![kubeimage](/images/kube-2.png)<br>
    - each cluster has `one master node` (kubernetes control plane) and `one or more worker nodes`
    - the control plane maintains the intended cluster state by making decisions about the cluster, detecting and responding to events in cluster.
        - like a termostat: specifying the temprature and the thermostat regulates heating and cooling systems continuously to achieve the specified state.

<a name='master'></a>

### Master Node
The Master Node is responsible for managing the cluster overall. There are several components inside the Master Node, including:
1. API Server: Provides an interface for interacting with Kubernetes.
2. Controller Manager: Maintains the current status of objects in the cluster, such as replication and scaling.
3. Scheduler: Schedules containers to run on Nodes.
4. etcd:
    - Function: Distributed data store used to store cluster configuration and status.
    - Responsibility: Provides consistent storage for cluster configuration data and metadata.

<a name='worker'></a>

### Worker Node
1. Kubelet:
    - Function: Runs containers in Pods on the Node and ensures that the state of the Pod aligns with the desired state.
    - Responsibility: Executes commands from the Master Node to create, deploy, and manage containers.
2. Kube Proxy:
    - Function: Enables network communication between Pods within the cluster.
    - Responsibility: Handles traffic forwarding and network rules between Pods.
3. Container Runtime:
    - Function: Handles container execution, such as Docker or another container runtime.
    - Responsibility: Manages the container lifecycle, including creation, deletion, and communication with the container.

<a name='object'></a>

## Kubernetes Object 
These objects provide the foundation for defining, managing, and automating various aspects of applications and resources within the Kubernetes cluster.
1. Pods:
    - Simplest unit in Kubernetes
    - Represents processes running on your cluster
    - Encapsulates one or more containers
    - Replicating a pod serves to scale application horizontally
    - YAML Files are often used to define the objects taht you want to create
        - ![kube-3](/images/kube-3.png)<br>
            - `Kind`: specified the kind of object to be created.
            - `Spec`: Provide the appropriate fields for the object to be created
2. Replica set:
    - Is a set of horizontally scaled running pods
        - ![kube-4](/images/kube-4.png)<br>
        - ![kube-5](/images/kube-5.png)<br>
            - defines: 
                - `Number of replicas`: the number of replicas that should be running at any given time. Whenever this field updated, the replicaset creates or deletes Pods to meet the desired number of replicas
                - `Pod Template`: defines a pod that should be created by the Replicaset
                - `Deployments`: Run multiple replicas of an application
3. Service:
    - Services provide an abstract way to access a set of Pods. It allows communication between applications within the cluster or with the external world. 
4. Namespace:
    - Namespace allows the division of the cluster into several isolated virtual environments. It helps organize, group, and isolate resources.
5. Deployment:
    - Deployment is used to define and manage applications running in Pods. It allows for the management of rolling updates and rollbacks.

<a name='kubectl'></a>
<a name='imperative'></a>
<a name='declarative'></a>

# Kubectl
![ctl](/images/kube-6.png)
<br>

- The `kubectl` command specifies required operations, optional flags, and atleast one filename:
    - Imperative Object Configuration
        - the configuration file specified must contain a full definition of the objects in YAML or JSON format.
        ```
        kubectl create -f nginx.yaml
        ```
        - Configuration templates help replicate identical results. the advantages is:
            - may be stored in a source control system like Git.
            - can integrate with change processes
            - provide audit trails and template for creating new objects
    - Declarative Object Configuration
        - Declarative object configurations stores configuration data in files
        - Operations are identified by kubectl, not the user.
        - works on dircetories or individual files.
        ```
        kubectl apply -f nginx/
        ```
        - Configuration file define desired state

## Create a Resource (cmd + output)
Let`s create a deployment with 3 replicas of the nginx image:
1. Create the deployment using an `apply` command.
    ```
    kubectl apply -f nginx.yaml
    ```
    the outputs confirm the deployment creation
    ```
    deployment.apps/nginx-deployment created
    ```
2. see the deployment details
    ```
    kubectl get deployment my-dep
    ```

<a name='excercise'></a>

# Excercise : Introduction to Kubernetes
In this lab, you will:
- Use the kubectl CLI
- Create a Kubernetes Pod
- Create a Kubernetes Deployment
- Create a ReplicaSet that maintains a set number of replicas
- Witness Kubernetes load balancing in action

## Use the kubectl CLI
Let’s look at some basic kubectl commands.
1. kubectl requires configuration so that it targets the appropriate cluster. Get cluster information with the following command:
    ```
    kubectl config get-clusters
    ```
    ![tutor 1](/images/kubectlCLI_1.png)<br>
2. A kubectl context is a group of access parameters, including a cluster, a user, and a namespace. View your current context with the following command:
    ```
    kubectl config get-contexts
    ```
    ![tutor 2](/images/kubectlCLI_2.png)<br>
3. List all the Pods in your namespace. If this is a new session for you, you will not see any Pods.
    ```
    kubectl get pods
    ```
    ![tutor 2](/images/kubectlCLI_3.png)<br>

## Create a Kubernetes Pod
Now it’s time to create your first Pod. This Pod will run the hello-world image you built and pushed to IBM Cloud Container Registry in the last lab. As explained in the videos for this module, you can create a Pod imperatively or declaratively. Let’s do it imperatively first.
1. Export your namespace as an environment variable so that it can be used in subsequent commands.
    ```
    export MY_NAMESPACE=sn-labs-$USERNAME
    ```
2. See the Dockerfile in this folder named `2_introKubernetes`
3. Build and push the image again, as it may have been deleted automatically since you completed the first lab.
    ```
    docker build -t us.icr.io/$MY_NAMESPACE/hello-world:1 . && docker push us.icr.io/$MY_NAMESPACE/hello-world:1
    ```
    ![tutor 2](/images/imp_cmd_3.png)<br>
4. Run the `hello-world` image as a container in Kubernetes.
    ```
    kubectl run hello-world --image us.icr.io/$MY_NAMESPACE/hello-world:1 --overrides='{"spec":{"template":{"spec":{"imagePullSecrets":[{"name":"icr"}]}}}}'
    ```
    The --overrides option here enables us to specify the needed credentials to pull this image from IBM Cloud Container Registry. Note that this is an imperative command, as we told Kubernetes explicitly what to do: run hello-world.
5. List the Pods in your namespace.
    ```
    kubectl get pods
    ```
    ![tutor 2](/images/imp_cmd_5.png)<br>
6. Describe the Pod to get more details about it.
    ```
    kubectl describe pod hello-world
    ```
7. Delete the Pod.
    ```
    kubectl delete pod hello-world
    ```

## Create a Pod with imperative object configuration
Imperative object configuration lets you create objects by specifying the action to take (e.g., create, update, delete) while using a configuration file. A configuration file, `hello-world-create.yaml`, is provided to you in this directory.
1. See the the filename `hello-world-create.yaml` in the `2_IntroKubernetes` folder.
2. Use the Explorer to edit hello-world-create.yaml. You need to insert your namespace where it says <my_namespace>. Make sure to save the file when you’re done.
3. Imperatively create a Pod using the provided configuration file.
    ```
    kubectl create -f hello-world-create.yaml
    ```
4. List the pod and delete the pod again like the step before.

## Create a Pod with a declarative command
The previous two ways to create a Pod were imperative – we explicitly told kubectl what to do. While the imperative commands are easy to understand and run, they are not ideal for a production environment. Let’s look at declarative commands.
1. A sample hello-world-apply.yaml file is provided in this directory. Use the Explorer again to open this file. Notice the following:
    - ![tutor 2](/images/create_pod_declr_1.png)<br>
    - We are creating a Deployment `(kind: Deployment)`.
    - There will be three replica Pods for this Deployment `(replicas: 3)`.
    - The Pods should run the hello-world image `(- image: us.icr.io/<my_namespace>/hello-world:1)`.
    - You can ignore the rest for now. We will get to a lot of those concepts in the next lab.

2. Use the Explorer to edit hello-world-apply.yaml. You need to insert your namespace where it says <my_namespace>. Make sure to save the file when you’re done.
3. Use the kubectl apply command to set this configuration as the desired state in Kubernetes.
    ```
    kubectl apply -f hello-world-apply.yaml
    ```
4. Get the Deployments to ensure that a Deployment was created.
    ```
    kubectl get deployments
    ```
5. List the Pods to ensure that three replicas exist.
    ```
    kubectl get pods
    ```
    With declarative management, we did not tell Kubernetes which actions to perform. Instead, kubectl inferred that this Deployment needed to be created. If you delete a Pod now, a new one will be created in its place to maintain three replicas.
6. Note one of the Pod names from the previous step, replace the pod_name in the following command with the pod name that you noted and delete that Pod and list the pods. To see one pod being terminated, there by having just 2 pods, we will follow the delete, immediately with get.
    ```
    kubectl delete pod <pod_name> && kubectl get pods
    ```
    This command takes a while to execute the deletion of the pod. Please wait till the terminal prompt appears again.

## Load balancing the application
Since there are three replicas of this application deployed in the cluster, Kubernetes will load balance requests across these three instances. Let’s expose our application to the internet and see how Kubernetes load balances requests.
1. In order to access the application, we have to expose it to the internet using a Kubernetes Service.
    ```
    kubectl expose deployment/hello-world
    ```
    This command creates what is called a ClusterIP Service. This creates an IP address that accessible within the cluster
2. List Services in order to see that this service was created.
    ```
    kubectl get services
    ```
    ![tutor 2](/images/load_balancing_2.png)<br>
3. Since the cluster IP is not accessible outside of the cluster, we need to create a proxy. Note that this is not how you would make an application externally accessible in a production scenario. Run this command in the new terminal window since your environment variables need to be accessible in the original window for subsequent commands. Add new terminal and type:
    ```
    kubectl proxy
    ```
4. ping the application to get a response.
    ```
    curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy
    ```
    Notice that this output includes the Pod name.
5. Run the command which runs a for loop ten times creating 10 different pods and note names for each new pod.
    ```
    for i in `seq 10`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/hello-world/proxy; done
    ```
    ![tutor 2](/images/load_balancing_6.png)<br>
    You should see more than one Pod name, and quite possibly all three Pod names, in the output. This is because Kubernetes load balances the requests across the three replicas, so each request could hit a different instance of our application.
6. Delete the Deployment and Service. This can be done in a single command by using slashes.
    ```
    kubectl delete deployment/hello-world service/hello-world
    ```
7. Return to the terminal window running the proxy command and kill it using Ctrl+C.