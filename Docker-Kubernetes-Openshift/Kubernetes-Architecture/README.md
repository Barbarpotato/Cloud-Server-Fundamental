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
## Kubernetes Architecture
![kubeimage](/images/kube-1.png)<br>
- A deployment of kubernetes is called kubernetes cluster.
    - is a cluster of nodes that runs containerized applications.
        - ![kubeimage](/images/kube-2.png)<br>
    - each cluster has `one master node` (kubernetes control plane) and `one or more worker nodes`
    - the control plane maintains the intended cluster state by making decisions about the cluster, detecting and responding to events in cluster.
        - like a termostat: specifying the temprature and the thermostat regulates heating and cooling systems continuously to achieve the specified state.

### Master Node
The Master Node is responsible for managing the cluster overall. There are several components inside the Master Node, including:
1. API Server: Provides an interface for interacting with Kubernetes.
2. Controller Manager: Maintains the current status of objects in the cluster, such as replication and scaling.
3. Scheduler: Schedules containers to run on Nodes.
4. etcd:
    - Function: Distributed data store used to store cluster configuration and status.
    - Responsibility: Provides consistent storage for cluster configuration data and metadata.

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
4. Pods:
    - Simplest unit in Kubernetes
    - Represents processes running on your cluster
    - Encapsulates one or more containers
    - Replicating a pod serves to scale application horizontally
    - YAML Files are often used to define the objects taht you want to create
        - ![kube-3](/images/kube-3.png)<br>
            - `Kind`: specified the kind of object to be created.
            - `Spec`: Provide the appropriate fields for the object to be created
5. Replica set:
    - Is a set of horizontally scaled running pods
        - ![kube-4](/images/kube-4.png)<br>
        - ![kube-5](/images/kube-5.png)<br>
            - defines: 
                - `Number of replicas`: the number of replicas that should be running at any given time. Whenever this field updated, the replicaset creates or deletes Pods to meet the desired number of replicas
                - `Pod Template`: defines a pod that should be created by the Replicaset
                - `Deployments`: Run multiple replicas of an application       
                
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
