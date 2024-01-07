# Table Of Contents
- [Building a Tekton Pipeline](#tekton-1)
- [Building a Tektok Triggers](#tekton-triggers)
- [Use Tekton Catalog](#tekton-catalog)
- [Integrating Unit Test](#unit-test)


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

<a name="tekton-triggers"></a>

# Adding Github Triggers
Running a pipeline manually has limited uses. In this lab you will create a Tekton Trigger to cause a pipeline run from external events like changes made to a repo in GitHub.

<img src="https://www.redhat.com/rhdc/managed-files/ohc/Filtering%20Tekton%20trigger%20operations-1.jpeg">

After completing this lab, you will be able to:
1. Create an EventListener, a TriggerBinding and a TriggerTemplate
2. State how to trigger a deployment when changes are made to github

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

<a name="tekton-catalog"></a>

# Using Tekton Catalog

Welcome to hands-on lab for Using the Tekton Catalog. The Tekton community provides a wide selection of tasks and pipelines that you can use in your CI/CD pipelines, so that you do not have to write all of them yourself. Many common tasks can be found at the Tekton Hub. In this lab, you will search for and use one of them.

After completing this lab, you will be able to:
1. Use the Tekton CD Catalog to install the git-clone task
2. Describe the parameters required to use the git-clone task
3. Use the git-clone task in a Tekton pipeline to clone your Git repository

## Step 1: Add the git-clone Task
You start by finding a task to replace the checkout task you initially created. While it was OK as a learning exercise, it needs a lot more capabilities to be more robust, and it makes sense to use the community-supplied task instead.

(Optional) You can browse the Tekton Hub, find the git-clone command, copy the URL to the yaml file, and use kubectl to apply it manually. But it is much easier to use the Tekton CLI once you have found the task that you want.

Use this command to install the git-clone task from Tekton Hub:
```bash
tkn hub install task git-clone --version 0.8
```

This installs the git-clone task into your cluster under your current active namespace.

## Step 2: Create a Workspace
Viewing the git-clone task requirements, you see that while it supports many more parameters than your original checkout task, it only requires two things:
1. The URL of a Git repo to clone, provided with the url param
2. A workspace called output

You start by creating a PersistentVolumeClaim (PVC) to use as the workspace:

A workspace is a disk volume that can be shared across tasks. The way to bind to volumes in Kubernetes is with a PersistentVolumeClaim.

Since creating PVCs is beyond the scope of this lab, you have been provided with the following pvc.yaml file with these contents:
```YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pipelinerun-pvc
spec:
  storageClassName: skills-network-learner
  resources:
    requests:
      storage:  1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
```

Apply the new task definition to the cluster:
```bash
kubectl apply -f pvc.yaml
```

You can now reference this persistent volume by its name pipelinerun-pvc when creating workspaces for your Tekton tasks.

### Step 3: Add a Workspace to the Pipeline

In this step, you will add a workspace to the pipeline using the persistent volume claim you just created. To do this, you will edit the pipeline.yaml file and add a workspaces: definition as the first line under the spec: but before the params: and call it pipeline-workspace. Then you will add the workspace to the pipeline clone task and change the task to reference git-clone instead of your checkout task.

`Your Task`
1. Edit the pipeline.yaml file and add a workspaces: definition as the first line under the spec: but before the params: and call it pipeline-workspace.
2. Next, add the workspace to the clone task after the name: and call it output because this is the workspace name that the git-clone task will be looking for.
3. Change the name of the taskRef in the clone task to reference the git-clone task instead of checkout.
4. Finally, change the name of the repo-url parameter to url because this is the name the git-clone tasks expects, but keep the mapping of $(params.repo-url), which is what the pipeline expects. Also, rename the branch parameter to revision, which is what git-clone expects.

### Step 4: Run the Pipeline

You can now use the Tekton CLI (tkn) to create a PipelineRun to run the pipeline.

Use the following command to run the pipeline, passing in the URL of the repository, the branch to clone, the workspace name, and the persistent volume claim name.
```bash
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

You can always see the pipeline run status by listing the PipelineRuns with:
```bash
tkn pipelinerun ls
```

You can check the logs of the last run with:
```bash
tkn pipelinerun logs --last
```

<a name="unit-test"></a>

# Integrating Unit Test Automation
Welcome to the hands-on lab for Integrating Unit Test Automation. In this lab, you will take the cloned code from the previous pipeline step and run linting and unit tests against it to ensure it is ready to be built and deployed.

After completing this lab, you will be able to:

1. Use the Tekton CD catalog to install the flake8 task
2. Describe the parameters required to use the flake8 task
3. Use the flake8 task in a Tekton pipeline to lint your code
4. Create a test task from scratch and use it in your pipeline

### Step 0: Check for cleanup
Please check as part of Step 0 for the new cleanup task which has been added to tasks.yaml file.

When a task that causes a compilation of the Python code, it leaves behind .pyc files that are owned by the specific user. For consecutive pipeline runs, the git-clone task tries to empty the directory but needs privileges to remove these files and this cleanup task takes care of that.

The init task is added pipeline.yaml file which runs everytime before the clone task.

Check the tasks.yaml file which has the new cleanup task updated.

### Step 1: Add the flake8 Task

Your pipeline has a placeholder for a lint step that uses the echo task. Now it is time to replace it with a real linter.

You are going to use flake8 to lint your code. Luckily, Tekton Hub has a flake8 task that you can install and use:

Use the following Tekton CLI command to install the flake8 task into your namespace.
```bash
tkn hub install task flake8
```

This will install the flake8 task in your Kubernetes namespace.

### Step 2: Modify the Pipeline to Use flake8
Now you will modify the pipeline.yaml file to use the new flake8 task.

In reading the documentation for the flake8 task, you notice that it requires a workspace named source. Add the workspace to the lint task after the name:, but before the taskRef:.

`Your Task`
- Scroll down to the lint task.
- Add the workspaces: keyword to the lint task after the task name: but before the taskRef:.
- Specify the workspace name: as source.
- Specify the workspace: reference as pipeline-workspace, which was created in the previous lab.
- Change the taskRef: from echo to reference the flake8 task.

`The result will look like this:`
```yaml
    - name: lint
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: flake8
```

### Step 3: Modify the Parameters for flake8
Now that you have added the workspace and changed the task reference to flake8, you need to modify the pipeline.yaml file to change the parameters to what flake8 is expecting.

In reading the documentation for the flake8 task, you see that it accepts an optional image parameter that allows you to specify your own container image. Since you are developing in a Python 3.9-slim container, you want to use python:3.9-slim as the image.

The flake8 task also allows you to specify arguments to pass to flake8 using the args parameter. These arguments are specified as a list of strings where each string is a parameter passed to flake8. For example, the arguments --count --statistics would be specified as: ["--count", "--statistics"].

`Your task:`
1. Change the message parameter to the image parameter to specify the value of python:3.9-slim.
2. Add a new parameter called args to specify the arguments as a list [] with the values --count --max-complexity=10 --max-line-length=127 --statistics to pass to flake8.

`The result will look like this:`
```yaml
    - name: lint
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: flake8
      params:
      - name: image
        value: "python:3.9-slim"
      - name: args
        value: ["--count","--max-complexity=10","--max-line-length=127","--statistics"]
      runAfter:
        - clone
```

Apply these changes to your cluster:
```bash
kubectl apply -f pipeline.yaml
```

### Step 4: Run the Pipeline
You are now ready to run the pipeline and see if your new lint task is working properly. You will use the Tekton CLI to do this.

Start the pipeline using the following command:
```bash
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

You should see the pipeline run complete successfully. If you see errors, go back and check your work against the solutions provided.

### Step 5: Create a Test Task
Your pipeline also has a placeholder for a tests task that uses the echo task. Now you will replace it with real unit tests. In this step, you will replace the echo task with a call to a unit test framework called nosetests.

There are no tasks in the Tekton Hub for nosetests, so you will write your own.

Update the tasks.yaml file adding a new task called nose that uses the shared workspace for the pipeline and runs nosetests in a python:3.9-slim image as a shell script as seen in the course video.

`the result will look like this:`
```yaml
---
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: nose
spec:
  workspaces:
    - name: source
  params:
    - name: args
      description: Arguments to pass to nose
      type: string
      default: "-v"
  steps:
    - name: nosetests
      image: python:3.9-slim
      workingDir: $(workspaces.source.path)
      script: |
        #!/bin/bash
        set -e
        python -m pip install --upgrade pip wheel
        pip install -r requirements.txt
        nosetests $(params.args)
```

Apply these changes to your cluster:
```bash
kubectl apply -f tasks.yaml
```

### Step 6: Modify the Pipeline to Use nose
The final step is to use the new nose task in your existing pipeline in place of the echo task placeholder.
```yaml
    - name: tests
      workspaces:
        - name: source
          workspace: pipeline-workspace
      taskRef:
        name: nose
      params:
      - name: args
        value: "-v --with-spec --spec-color"
      runAfter:
        - lint
```

Apply these changes to your cluster:
```bash
kubectl apply -f pipeline.yaml
```

### Step 7: Run the Pipeline Again
Now that you have your tests task complete, run the pipeline again using the Tekton CLI to see your new test tasks run:
```bash
tkn pipeline start cd-pipeline \
    -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
    -p branch="main" \
    -w name=pipeline-workspace,claimName=pipelinerun-pvc \
    --showlog
```

You can see the pipeline run status by listing the PipelineRun with:
```bash
tkn pipelinerun ls
```

You can check the logs of the last run with:
```bash
tkn pipelinerun logs --last
```

