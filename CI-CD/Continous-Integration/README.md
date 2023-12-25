# Understanding Continuous Integration (CI)

- CI is an automation process that helps developers continuously integrate code by using short-lived branches. 

- CI involves frequent pull requests that are easy to review and encourage collaboration. 

- CI automatically runs code through predefined tests, streamlining development. 

- With CI/CD, you get the following: 
    - Faster reaction times to changes 
    - Reduced code integration risk 
    - Higher code quality 

- Confirmation that the code in version control works. 

- Social coding leads to high-quality code and increased collaboration, saving time, effort, and money. 

- Git enables DevOps and has many commands that provide essential functionality. 

- Developers use Git’s Feature Branch Workflow to develop clean, concise, high-quality code. 

- The Git Feature Branch Workflow involves five steps: 
    - Clone a repository to your local system and create a branch to work on your issue. 
    - Commit changes to that branch. 
    - Push your changes to the remote branch. 
    - Issue a pull request to have your work reviewed. 
    - Merge your code to the main branch and close the issue. 

- Standard CI tools include Jenkins, CircleCI, TravisCI, and GitHub Actions, and each has advantages and disadvantages. 

- You can write your CI pipelines as code for your CI tool to run. 

- Many CI tools are offered as services that you can easily run and scale on the cloud.

# Implementing Continuous Integration (CI)

## Using GitHub Actions - Part 1
In this part, you will build a workflow in a GitHub repository using GitHub Actions. You will create an empty workflow file in Step 1 and add events and a job runner in the following steps. You will subsequently finish the workflow in the next lab called Using GitHub Actions - Part 2. Ensure you finish this lab completely before starting part 2.

After completing this lab, you will be able to:
- Create a GitHub workflow to run your CI pipeline
- Add events to trigger the workflow
- Add a job to the workflow
- Add a job runner to the job
- Add a container to the job runner

### Generate GitHub Personal Access Token
You have a little preparation to do before you can start the lab.

#### Generate a Personal Access Token
You will fork and clone a repo in this lab using the gh CLI tool. You will also push changes to your cloned repo at the end of this lab. This requires you to authenticate with GitHub using a personal access token. Follow the steps here to generate this token and save it for later use:
1. Navigate to GitHub Settings of your account.
2. Click Generate new token to create a personal access token.
3. Give your token a descriptive name and optionally change the expiration date.
4. Select the minimum required scopes needed for this lab: repo, read:org, and workflow.
5. Click Generate token.
6. Make sure you copy the token and paste it somewhere safe as you will need it in the next step. WARNING: You will not be able to see it again.

If you lose this token at any time, repeat the above steps to regenerate the token.

### Fork and Clone the Repository

#### Authenticate with GitHub
Run the following command to authenticate with GitHub in the terminal. You will need the GitHub Personal Token you created in the previous step.
```
gh auth login
```
After you have authenticated successfully, you will need to fork and clone this GitHub repo in the terminal. You will then create a workflow to trigger GitHub Actions in your forked version of the repository.

#### Fork and Clone the Reference Repo
```
gh repo fork ibm-developer-skills-network/wtecc-CICD_PracticeCode --clone=true
```

### Step 1: Create a Workflow
To get started, you need to create a workflow yaml file. The first line in this file will define the name of the workflow that shows up in GitHub Actions page of your repository.

1. Create the directory structure `.github/workflows` and create a file called `workflow.yml`.
```
mkdir -p .github/workflows
touch .github/workflows/workflow.yml
```

2. Every workflow starts with a name. The name will be displayed on the Actions page and on any badges. Give your workflow the name CI workflow by adding a name: tag as the first line in the file.
```
name: {insert name here}
```


### Step 2: Add Event Triggers
Event triggers define which events can cause the workflow to run. You will use the on: tag to add the following events:
- Run the workflow on every push to the main branch
- Run the workflow whenever a pull request is created to the main branch.

1. Add the on: keyword to the workflow at the same level of indentation as the name:.
```yml
name: CI workflow

on:
```

2. Add `push:` event as the first event that can trigger the workflow. This is added as the child element of on: so it must be indented under it.
```yml
on:
    {insert first event name here}:
```

3. Add the `"main"` branch to the push event. You want the workflow to start every time somebody pushes to the main branch. This also includes merge events. You do this by using the branches: keyword followed by a list of branches either as [] or -
```yml
on:
    push:
        branches: [ {insert branch name here} ]
```

4. Add a pull_request: event similar to the push event you just finished. It should be triggered whenever the user makes a pull request on the main branch.
```yml
on:
    push:
        branches: [ "main" ]
    {insert second event name here}:
        branches: [ {insert branch name here} ]
```

Now the yml file will look like this:

```yml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
```

### Step3: Add a Job
You will now add a job called build to the workflow file. This job will run on the ubuntu-latest runner. Remember, a job is a collection of steps that are run on the events you added in the previous step.

1. First you need a job. Add the `jobs:` section to the workflow at the same level of indentation as the `name` (i.e., no indent).
```yml
jobs:
```

2. Next, you need to name the job. Name your job `build:` by adding a new line under the `jobs:` section.
```yml
jobs:
    {insert job name here}:
```

3. Finally, you need a runner. Tell GitHub Actions to use the `ubuntu-latest` runner for this job. You can do this by using the `runs-on:` keyword.
```yml
jobs:
  build:
    runs-on: {insert runner name here}
```

Now the yml file will look like this:
```yml
name: CI workflow
on:
  push:
    branches: [ "main" ] 
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
```

### Target Python 3.9
It is important to consistently use the same version of dependencies and operating system for all phases of development including the CI pipeline. This project was developed on Python 3.9, so you need to ensure that the CI pipeline also runs on the same version of Python. You will accomplish this by running your workflow in a container inside the GitHub action.

1. Add a `container:` section under the `runs-on:` section of the build job, and tell GitHub Actions to use `python:3.9-slim` as the image.
```yml
jobs:
  build:
    runs-on: ubuntu-latest
    container: {insert container name here}
```

Finally the script is like this:
```yml
name: CI workflow
on:
  push:
    branches: [ "main" ] 
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: python:3.9-slim

```

### Save Your Work
It is now time to save your work back to your forked GitHub repository.

1. Configure the Git account with your email and name using the git config --global user.email and git config --global user.name commands.
```git
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```

2. The next step is to stage all the changes you made in the previous exercises and push them to your forked repo on GitHub.
```
git add -A
git commit -m "COMMIT MESSAGE"
git push
```

You are done with part 1 of the lab, however if you were to look at the Actions tab in your forked repository, you will notice the GitHub action was triggered and has failed. The action was triggered because you pushed code to the main branch of the repository. It failed as you have not finished the workflow yet. You will add the remaining steps in part 2 of the lab so the workflow runs successfully. You can ignore this error at this time.


## Using GitHub Actions - Part 2

In this lab, you will build the Continuous Integration pipeline by setting up the steps under job in your workflow. Please ensure you complete setting up the workflow, before starting this lab.
You will add the following steps to the job in this lab:
- Check out the code
- Install dependencies
- Lint code with Flake8
- Run unit tests with nose

### Step 1: Add the checkout Step
You will now add your first step to the job. The first task is to check out the code from the repository. You will use the predefined actions/checkout@v3 action to do this. You can learn more about this action on the GitHub page.

1. Add the steps: section at the end of the build job in the workflow file.
```yml
steps:
```
2. Add the checkout step as the first step. Give it the `name:` Check out and call the action `actions/checkout@v3` by using the `uses:` keyword.
```yml
steps:
  - name: {insert step name here}
    uses: {insert action name here}
```

The script will like like this:
```yml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: python:3.9-slim
    steps:
      - name: Checkout
        uses: actions/checkout@v3
```

### Step 2: Install Dependencies
Now that you have checked out the code, the next step is to install the dependencies. This application uses pip, the Python package manager, to install all the dependencies from the PyPI package. The pip command refers to the requirements.txt file for the list of dependencies. The python:3.9-slim container you are using already has the pip package manager installed.

The commands that you will use in this step are:
```
python -m pip install --upgrade pip
```

This command installs pip if it is not found on the GitHub runner or updates it to the latest version if it is already installed. It is always a good practice to make sure that you are using the latest version of pip to minimize installation problems.

```
pip install -r requirements.txt
```

This command instructs pip to install any dependencies listed in the requirements.txt file. It is a best practice to place the names of dependent packages here.

`Note: If the above command gives any error, please execute the below command to install the dependencies`
```
pip3 install -r requirements.txt
```

1. Add a new named step after the checkout step. Name this step `Install dependencies`
```yml
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install dependencies 
```

2. Next, you want to run the commands to update the package manager pip and then install the dependencies. Since there are two commands, you can run these inline using the run: keyword with the pipe | operator.
```yml
         run: |
            python -m pip install --upgrade pip
            pip install -r requirements.txt
```

the script will finally look like this:
```yml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: python:3.9-slim
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
```

### Step 3: Lint with flake8
It is always a good idea to add quality checks to your CI pipeline. This is especially true if you are working on an open source project with many different contributors. This makes sure that everyone who contributes is following the same style guidelines.

The next step is to use flake8 to lint the source code. Linting is checking your code for syntactical and stylistic issues. Some examples are line spacing, using spaces or tabs for indentation, locating uninitialized or undefined variables, and missing parentheses. The flake8 library was installed as a dependency in the requirements.txt file.

The flake8 commands take a few parameters. Now, take a look at the command and the parameters:
```
flake8 service --count --select=E9,F63,F7,F82 --show-source --statistics
```

1. Add a new named step after the Install dependencies step. Call this step Lint with flake8.
```yml
    - name: Lint with flake8
```

2. Next, you want to run the flake8 commands to lint your code. The commands are explained in detail above.
```yml
      run: |
        flake8 service --count --select=E9,F63,F7,F82 --show-source --statistics
        flake8 service --count --max-complexity=10 --max-line-length=127 --statistics
```

The script will look like this:
```yml
name: CI workflow

on:
  push:
    branches: main
  pull_request:
    branches: main

jobs:
  build:
    runs-on: ubuntu-latest
    container: python:3.9-slim
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Lint with flake8
        run: |
          flake8 service --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 service --count --max-complexity=10 --max-line-length=127 --statistics
```

### Step 4: Test Code Coverage with nosetests
You will use nose in this step to unit test the source code. Nose is configured via the included setup.cfg file to automatically include the flags –with-spec and –spec-color so that red-green-refactor is meaningful. If you are in a command shell that supports colors, passing tests will be green and failing tests will be red.

Nose is also configured to automatically run the coverage tool, and you should see a percentage of coverage report at the end of your tests.
1. Add a new named step after the Lint with flake8 step. Call this step Run unit tests with nose.
```yml
  - name: Run Unit tests with nose
```

2. Next, you want to run the nosetests command to test your code and report on code coverage.
```yml
    run: nosetests -v --with-spec --spec-color --with-coverage --cover-package=app
```


the script will look like this:
```yml
name: CI workflow

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: python:3.9-slim
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Lint with flake8
        run: |
          flake8 service --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 service --count --max-complexity=10 --max-line-length=127 --statistics

      - name: Run unit tests with nose
        run: nosetests -v --with-spec --spec-color --with-coverage --cover-package=app
```

### Step 5: Push Code to GitHub
To test the workflow and the CI pipeline, you need to commit the changes and push your branch back to the GitHub repository. As described earlier, each new push to the main branch should trigger the workflow.

1. Configure the Git account with your email and name using the git config --global user.email and git config --global user.name commands.
```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
2. The next step is to stage all the changes you made in the previous exercises and push them to your forked repo on GitHub.
```bash
git add -A
git commit -m "COMMIT MESSAGE"
git push
```

### Step 6: Run the Workflow
You can run the workflow now. Open the Actions tab in the forked repository in your GitHub account. Click the CI workflow action. Pushing changes in the previous step should have triggered a build action and it should be visible in the Actions tab.
