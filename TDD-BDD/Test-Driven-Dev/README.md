# Test Driven Development

- In TDD, test cases drive code design. 
- The Red/Green/Refactor workflow includes three steps: 
    - Write a failing unit test case for the code you wish you had 
    - Write just enough code to pass this test case 
    - Refactor the code to improve its quality 
- TDD saves development time and ensures that the code works as expected. 
- To create a DevOps pipeline, you must automate all testing. 
- The xUnit series is one of the most popular testing frameworks for TDD, while others include Jasmine for JavaScript, Mocha for Node.js, and SimpleTest for PHP. 
- The two most popular testing frameworks for Python are PyUnit and Pytest. Two other popular testing frameworks for Python are Doctest and RSpec. 
- Nose is a Python test runner that can add color to test output and call the code coverage tool.


## Running Tests with Nose

Welcome to the Running Tests with Nose lab. In this lab, you will learn how to use fundamental tools for running unit tests in Python.

After completing this lab, you will be able to:

Install Nose, Pinocchio, and Coverage
Run unit tests with unittest and Nose
Produce color-coded test output
Add coverage reports to your test output

### Step 1: Working with unittest
You are ready to run your first tests. The unittest module is built into Python 3. You can invoke it through the Python interpreter by passing it the argument -m to run a module, and then unittest to give it the name of the module that you want to run.

In the terminal, run the following command to run the tests using the Python unittest package:
```bash
python3 -m unittest
```
In the output you will see that all 11 tests passed as indicated by the 11 dots. Had there been any errors, you would have seen the letter E in place of the dot, representing an error in that test.

#### More verbose output
You can make the output more useful by adding the -v flag to turn on verbose mode. Run the command below for more useful output:
```bash
python3 -m unittest -v
```

### Step 2: Working with Nose
There is a test runner called nose that you can use to produce better test output. It is a Python package that you can install using the Python package manager pip utility.

Install Nose using pip:
```bash
python3.8 -m pip install nose
```

To see verbose output from nose, run nosetests -v. The verbose output from Nose will return nicer output than from unittest because it only returns the docstring comments:
```bash
nosetests -v
```

### Step 3: Adding color with Pinocchio
Another way to make your output look better is with a plugin called pinocchio. With this plugin, you can get output as a specification similar to Rspec and also add color to the output. The color really gives you the Red/Green/Refactor workflow that TDD is famous for.

Install pinocchio using pip.
```bash
python3.8 -m pip install pinocchio
```

To get nicer formatting and a colorful output, run nose again and add the --with-spec --spec-color parameters:
```bash
nosetests --with-spec --spec-color
```

### Step 4: Adding test coverage
To know if you鴴en enough tests, you need to know how many lines of code your tests cover. The coverage tool will calculate the number of lines of code executed during your tests, against the total lines of code, and report that as a percentage of coverage.

Install the coverage tool so that you can check your test coverage:
```bash
python3.8 -m pip install coverage
```

Next, call coverage through nose by adding the --with-coverage parameter.
```bash
nosetests --with-spec --spec-color --with-coverage
```

### Step 5: Create missing coverage report
One useful feature of the coverage tool is that it can report which lines of code are missing coverage. With that information, you know the lines for which you need to add more test cases so that your testing executes those missing lines of code.

To get the missing coverage report, run the below command in the terminal:

```bash
coverage report -m
```

### Step 6: Automating these parameters
Up until now you have typed out a lot of command parameters when running tests with nose. Alternatively, you can save all the parameters in a configuration file so that you don’t have to type them in every time.

Create a new file named setup.cfg. Here are the steps:
```
[nosetests]
verbosity=2
with-spec=1
spec-color=1
with-coverage=1
cover-erase=1
cover-package=triangle

[coverage:report]
show_missing = True
```

### Step 7: Run nosetests with config
Now that you have established your setup.cfg file, go to your terminal and run nosetests without any parameters:
```
nosetests
```



