# Table Of Contents
- [Building a Tekton Pipeline](#tekton-1)



# Understanding Continuous Delivery

- The conceptual building blocks of Tekton are events, triggers, pipelines, tasks, and steps. 

- The physical building blocks of Tekton are Kubernetes custom resource definitions (CRDs). 

- You can create Tekton pipelines by referencing tasks and passing in required parameters. 

- EventListeners listen for external events, TriggerBindings respond to those events and bind parameters from them, and TriggerTemplates create PipelineRuns that pass the parameters to the pipeline. 

- The Tekton Catalog, or Tekton Hub, contains reusable tasks for your CI/CD pipelines. 

- The PipelineRun must map the workspace to a PersistentVolumeClaim. 

- You can use existing shell scripts within tasks and define environment variables to pass configuration information into tasks. 

- You can use Tekton Hub or Tekton CLI to find build tasks for your CI/CD pipelines.

- To run a task after parallel tasks, you must specify the parallel tasks in the runAfter field.

- You can deploy applications to an environment using commands or YAML manifests.


<a name="tekton-1"></a>

# Building a Tekton Pipeline

## Step 1: Create an echo Task
In true computer programming tradition, the first task you create will echo "Hello World!" to the console.

There is starter code in the labs/01_base_pipeline folder for a task and a pipeline. Navigate to this folder in left explorer panel, and open the tasks.yaml file to edit it

`Your Task`
1. The first thing you want to do is give the task a good name. Change <place-name-here> to hello-world.

2. The next thing is to add a step that run a single command to include name, image, command, and args. Make the name echo, use the image alpine:3, have the command be [/bin/echo] and the args be ["Hello World"].

The Result will look like this:
```yml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello-world
spec:
  steps:
    - name: echo
      image: alpine:3
      command: [/bin/echo]
      args: ["Hello World!"]
```

3. Apply it to the cluster
```bash
kubectl apply -f tasks.yaml
```

## Step 2: Create a hello-pipeline Pipeline

Next, you will create a very simple pipeline that only calls the hello-world task that you just created. Navigate to this folder in left explorer panel, and open the pipeline.yaml

`Your Task`
1. The first thing you want to do is give the pipeline a good name. Change <place-name-here> to hello-pipeline.

2. The next thing is to add a reference to the hello-world task you just created which needs a name: for the pipeline task, and a taskRef:, with a name: tag under it set to the name of the task you are referencing. Set the name of the pipeline task to hello and the name of the task you are referencing as hello-world.

The Result will look like this:
```yml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-pipeline
spec:
  tasks:
    - name: hello
      taskRef:
        name: hello-world
```

Apply it to the cluster:
```bash
kubectl apply -f pipeline.yaml
```

## Step 3: Run the hello-pipeline
Run the pipeline using the Tekton CLI:
```bash
tkn pipeline start --showlog hello-pipeline
```

You should see the output:
```yaml
PipelineRun started: hello-pipeline-run-9vkbb
Waiting for logs to be available...
[hello : echo] Hello World!
```

## Step 4: Add a parameter to the task
Hopefully the hello-world task has given you a sense for how pipelines call tasks. Now it is time to make that task a little more useful by making it print any message that you want, not just "Hello World".

To do this, you will add a parameter called message to the task and use that parameter as the message that it echoes. You will also rename the task to echo.

Edit the tasks.yaml file to add the parameter to both the input and the echo command:

`Your Task`
1. Change the name of the task from hello-world to echo to more acurately reflect its new functionality, by changing the name: in the metadata: section.
2. Add a params: section to the task with a parameter that has a name: of "message", a type: of "string", and a description of "The message to echo".
3. Change the name of the step from echo to echo-message to better describe its new functionality.
4. Modify the args: tag to use the message parameter you just created.

The result task.yaml file will look like this:
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: echo
spec:
  params:
    - name: message
      description: The message to echo
      type: string
  steps:
    - name: echo-message
      image: alpine:3
      command: [/bin/echo]
      args: ["$(params.message)"]
```

Apply the new task definition to the cluster:
```bash
kubectl apply -f tasks.yaml
```

## Step 5: Update the hello-pipeline
You now need to update the pipeline to pass the message that you want to send to the echo task so that it can echo the message to the console.
Edit the pipeline.yaml file to add the parameter:

`Your Task`
1. Add a params: section to the pipeline under spec:, with a parameter that has a name: of "message".
2. Change the name: of the taskRef: from hello-world to the new echo task.
3. Add a params: section to the task, with a parameter that has a name: of "message" and a value: that is a reference to the pipeline parameter for params.message.


The result will look like this:
```yml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-pipeline
spec:
  params:
    - name: message
  tasks:
    - name: hello
      taskRef:
        name: echo
      params:
        - name: message
          value: "$(params.message)"
```

Apply it to the cluster:
```bash
kubectl apply -f pipeline.yaml
```

##  Step 6: Run the message-pipeline
Run the pipeline using the Tekton CLI:
```bash
tkn pipeline start hello-pipeline \
    --showlog  \
    -p message="Hello Tekton!"
```

You should see the output:
```
PipelineRun started: hello-pipeline-run-9qf42
Waiting for logs to be available...
[hello : echo-message] Hello Tekton!
```

## Step 7: Create a checkout Task
In this step, you will combine your knowledge of running a command in a container with your knowledge of passing parameters, to create a task that checks out your code from GitHub as the first step in a CD pipeline.

#### Create checkout task
You can have multiple definitions in a single yaml file by separating them with three dashes --- on a single line. In this step, you will add a new task to tasks.yaml that uses the bitnami/git:latest image to run the git command passing in the branch name and URL of the repo you want to clone.

`Your Task`
Your new task will create a Tekton task that accepts a repository URL and a branch name and calls git clone to clone your source code.

1. Create a new task and name it checkout.

2. Add a parameter named repo-url with a type: of string and a description: of "The URL of the git repo to clone".

3. Add a second parameter named branch with a type: of string and a description: of "The branch to clone".

4. Add a step with the name: "checkout" that uses the bitnami/git:latest image to run the git command by specifying clone and --branch parameters and passing both the params created in spec as the arguments.

the result will look like this:
```yml
---
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: checkout
spec:
  params:
    - name: repo-url
      description: The URL of the git repo to clone
      type: string
    - name: branch
      description: The branch to clone
      type: string
  steps:
    - name: checkout
      image: bitnami/git:latest
      command: [git]
      args: ["clone", "--branch", "$(params.branch)", "$(params.repo-url)"]
```

Apply it to the cluster:

```bash
kubectl apply -f tasks.yaml
```

`The echo task was unchanged and the checkout task has been created.`

## Step 8: Create the cd-pipeline Pipeline
Finally, you will create a pipeline called cd-pipeline to be the starting point of your Continuous Delivery pipeline.

Open the pipeline.yaml file to create a new pipeline called cd-pipeline:

You can use --- on a separate line to separate your new pipeline, or you can modify the existing pipeline to look like the new one.

`Your Task`
1. Create a new pipeline and name it cd-pipeline.
2. Add two parameters named repo-url and branch.
3. Set the default: for branch to "master".
4. Add a task with the name: "clone" that has a taskRef: to the checkout task that you just created.
5. Add the two parameters repo-url and branch to the task, mapping them back to the pipeline parameters of the same name.

the result will look like this:
```yml
---
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: cd-pipeline
spec:
  params:
    - name: repo-url
    - name: branch
      default: "master"
  tasks:
    - name: clone
      taskRef:
        name: checkout
      params:
      - name: repo-url
        value: "$(params.repo-url)"
      - name: branch
        value: "$(params.branch)"
```

Apply it to the cluster:

```bash
kubectl apply -f pipeline.yaml
```

## Step 9: Run the cd-pipeline
Run the pipeline using the Tekton CLI:

```bash
tkn pipeline start cd-pipeline \
    --showlog \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch="main"
```
The output should look like this:
```
PipelineRun started: cd-pipeline-run-rf6zp
Waiting for logs to be available...
[clone : checkout] Cloning into 'wtecc-CICD_PracticeCode'...
```

<a name="github-triggers"></a>

# Adding Github Triggers
Running a pipeline manually has limited uses. In this lab you will create a Tekton Trigger to cause a pipeline run from external events like changes made to a repo in GitHub.

<img src="https://www.redhat.com/rhdc/managed-files/ohc/Filtering%20Tekton%20trigger%20operations-1.jpeg">

After completing this lab, you will be able to:
1. Create an EventListener, a TriggerBinding and a TriggerTemplate
2. State how to trigger a deployment when changes are made to github

## Set Up the Lab Environment
You have a little preparation to do before you can start the lab.

1. Open a Terminal
2. Clone the repo: git clone https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git
3. Change directory : cd wtecc-CICD_PracticeCode/labs/02_add_git_trigger/

## Prerequisites
This lab starts with the `cd-pipeline` pipeline and `checkout` and `echo` tasks from the previous lab.
1. Check the task you will created: `tkn task ls`
2. Check that the pipeline was created: `tkn pipeline ls`

## Step1: Create an EventListener
The first thing you need is an event listener that is listening for incoming events from GitHub.

You will update the eventlistener.yaml file to define an EventListener named cd-listener that references a TriggerBinding named cd-binding and a TriggerTemplate named cd-template.

`Your Task`

1. The first thing you want to do is give the EventListener a good name. Change <place-name-Here> to cd-listener.
2. The next thing is to add a service account. Add a serviceAccountName: with a value of pipeline to the spec section.
3. Now you need to define the triggers. Add a triggers: section under spec:. This is where you will define the bindings and template.
4. Add a bindings: section under the triggers: section with a ref: to cd-binding. Since there can be mutiple triggers, make sure you define - bindings as a list using the dash - prefix. Also since there can be multiple bindings, make sure you defne the - ref: with a dash - prefix as well.
5. Add a template: section at the same level as bindings with a ref: to cd-template.

`The result will look like this:`
```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: cd-listener
spec:
  serviceAccountName: pipeline
  triggers:
    - bindings:
      - ref: cd-binding
      template:
        ref: cd-template
```

6. Apply the event listener resource to the cluster
```bash
kubectl apply -f eventlistener.yaml
```

7. Check that it was created correctly.
```
tkn eventlistener ls
```

You will create the TriggerBinding named cd-binding and a TriggerTemplate named cd-template in the next steps.

## Step 2: Create a TriggerBinding
The next thing you need is a way to bind the incoming data from the event to pass on to the pipeline. To accomplish this, you use a TriggerBinding.

Update the triggerbinding.yaml file to create a TriggerBinding named cd-binding that takes the body.repository.url and body.ref and binds them to the parameters repository and branch, respectively.

`Your Task`
1. The first thing you want to do is give the TriggerBinding the same name that is referenced in the EventListener, which is cd-binding.
2. Next, you need to add a parameter named repository to the spec: section with a value that references $(body.repository.url).
3. Finally, you need to add a parameter named branch to the spec: section with a value that references $(body.ref).

`The result will look like this`
```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: cd-binding
spec:
  params:
    - name: repository
      value: $(body.repository.url)
    - name: branch
      value: $(body.ref)
```

4. Apply the new TriggerBinding definition to the cluster:
```bash
kubectl apply -f triggerbinding.yaml
```

## Step 3: Create a TriggerTemplate
The TriggerTemplate takes the parameters passed in from the TriggerBinding and creates a PipelineRun to start the pipeline.

Update the triggertemplate.yaml file to create a TriggerTemplate named cd-template that defines the parameters required, and create a PipelineRun that will run the cd-pipeline you created in the previous lab.

`Your Task`
1. You must update the parameter section of the TriggerTemplate and fill out the resourcetemplates section:
  Update Name and Add Parameters
  - The first thing you want to do is give the TriggerTemplate the same name that is referenced in the EventListener, which is cd-template.
  - Next, you need to add a parameter named repository to the spec: section with a description: of "The git repo" and a default: of " ".
  - Then, you need to add a parameter named branch to the spec: section with a description: of "the branch for the git repo" and a default: of master.
  Complete the Resource Template
  - Add a serviceAccountName: with a value of pipeline.
  - Add a pipelineRef: that refers to the cd-pipeline created in the last lab.
  - Add a parameter named repo-url with a value referencing the TriggerTemplate repository parameter above.
  - Add a second parameter named branch with a value referencing the TriggerTemplatebranch parameter above.

`The result will look like this`
```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: cd-template
spec:
  params:
    - name: repository
      description: The git repository
      default: ""
    - name: branch
      description: the branch for the git repo
      default: "master"
  resourcetemplates:
    - apiVersion: tekton.dev/v1beta1
      kind: PipelineRun
      metadata:
        generateName: cd-pipeline-run-
      spec:
        serviceAccountName: pipeline
        pipelineRef: cd-pipeline
        params:
          - name: repo-url
            value: $(tt.params.repository)
          - name: branch
            value: $(tt.params.branch)
```

2. Apply the new TriggerTemplate definition to the cluster:
```bash
kubectl apply -f triggertemplate.yaml
```

## Optional: Adding serviceAccount
The EventListener requires a service account to run. To create the service account for this example create a file named rbac.yaml and add the following:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-robot
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: triggers-example-eventlistener-binding
subjects:
- kind: ServiceAccount
  name: tekton-robot
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-roles
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: triggers-example-eventlistener-clusterbinding
subjects:
- kind: ServiceAccount
  name: tekton-robot
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-clusterroles
```

This service account allows the EventListener to create PipelineRuns.
Apply the file to your cluster:
```bash
kubectl apply -f rbac.yaml
```

## Step 4: Start a Pipeline Run
Now it is time to call the event listener and start a PipelineRun. You can do this locally using the curl command to test that it works.

For this last step, you will need two terminal sessions.

### Terminal 1
In one of the sessions, you need to run the kubectl port-forward command to forward the port for the event listener so that you can call it on localhost.

Use the kubectl port-forward command to forward port 8090 to 8080.
```bash
kubectl port-forward service/el-cd-listener  8090:8080
```

### Terminal 2
Now you are ready to trigger the event listener by posting to the endpoint that it is listening on. You will now need to open a second terminal shell to issue commands.

1. Open a new Terminal shell wtih the menu item Terminal > New Terminal.

2. Use the curl command to send a payload to the event listener service.
  ```bash
  curl -X POST http://localhost:8090 \
  -H 'Content-Type: application/json' \
  -d '{"ref":"main","repository":{"url":"https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode"}}'
  ```

3. This should start a PipelineRun. You can check on the status with this command:
  ```bash
  tkn pipelinerun ls
  ``` 
  You should see something like this come back:

4. You can also examine the PipelineRun logs using this command (the -L means "latest" so that you do not have to look up the name for the last run):
  ```bash
  tkn pipelinerun logs --last
  ```

## Conclusion
Congratulations, you have successfully set up Tekton Triggers.

In this lab, you learned how to create a Tekton Trigger to cause a pipeline run from external events like changes made to a repo in GitHub. You learned how to create EventListerners, TriggerTemplates, TriggerBindings and how to start a Pipeline Run on a port.