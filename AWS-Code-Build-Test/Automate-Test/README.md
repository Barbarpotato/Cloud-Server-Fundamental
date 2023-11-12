# Exercise: Using AWS CodeBuild
In this exercise, you will be updating the application with a buildspec file, and using CodeBuild to test the application. The buildspec file contains the same tests you performed with the local build: linting with pylint, and unit tests with pytest. You will also use CodeBuild to run unit tests against the application, and then view the log output.

## Task 1: Creating a buildspec file
In this task, you create a buildspec file, which CodeBuild uses to run a build. You will then check these changes into your repository.

1. If needed, return to AWS Cloud9 and open your development environment.
2. In the Environment pane, create a file by right-clicking the trivia-app folder and choosing New File.
3. Rename the new file to buildspecs/unittests.yaml. In the Confirm move to a new folder window, choose Yes.
4. Open the new file and in the new file, copy and paste the following code:
    ```
    version: 0.2

    phases:
    install:
        commands:
        - pip3 install -U -r back-end-python/gameactions/requirements.txt
        - pip3 install -U -r back-end-python/tests/requirements.txt

    build:
        commands:
        - pylint --fail-under=8 back-end-python/gameactions/app.py
        - pytest back-end-python/tests/unit --junit-xml=unittests.xml --cov-report=xml --cov=gameactions --cov-branch

    reports:
    UnitTests:
        files:
        - 'unittests.xml'
    NewCoverage: #
        files:
        - 'coverage.xml'
        file-format: COBERTURAXML
    ```

5. Save the file.
6. Add the file to your AWS CodeCommit repository.
    ```
    git add *
    git commit -m "adding a unittests buildspec"
    git push origin main
    ```
![alt img](/images/auto-1.png)
<br>


## Task 2: Running the unit tests in CodeBuild
In this task, you use CodeBuild to run tests and the console to view the log output.

1. At the top left, choose the AWS Cloud9 icon and choose Go To Your Dashboard.

2. Search for and choose CodeCommit.

3. In the AWS CodeCommit navigation pane, choose Repositories.

4. In the Repositories pane, open the trivia-app repository.

5. View the commit that was pushed from AWS Cloud9 by going to the navigation pane and choosing Commits.

6. In the navigation pane, expand Build and then choose Build projects.

7. Choose Create build project and configure the following settings.
    -   Project name: trivia-unittests
    -   Repository: trivia-app
    -   Branch: main
    -   Operating system: Ubuntu
    -   Runtime(s): Standard
    -   Image: aws/codebuild/standard:5.0
    -   Build specifications: Keep Use a buildspec file selected
    -   Buildspec name - optional: buildspecs/unittests.yaml

8. Choose Create build project.

9. Choose Start build.

10. To see the build output, choose Tail logs.

11. When the build shows as Succeeded, close the log window by choosing Close (at the bottom right of the window).

12. Choose the Reports tab.

13. View the Pass rate by selecting the Test report, or the Line coverage and Branch coverage by selecting the Code coverage report.
![alt img](/images/auto-2.png)
<br>
