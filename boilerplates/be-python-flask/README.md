# Python Flask Quickstarter (be-python-flask) 
The project supports generation of Python Flask project boilerplate and quick 
installation and integration of it with OpenShift CD pipelines.

## Purpose of this quickstarter
The quickstarter is simmple WEB-server written on Python using Flask framework.
The package allows easily build a Python project, using different Python modules
and frameworks.
It contains the basic setup for Docker, Jenkins, SonarQube and OpenShift.

## What files / architecture is generated?
    
    ├── Jenkinsfile - This file contains Jenkins build configuration settings
    ├── README.md
    ├── files
    │   ├── docker - This folder contains Docker configuration settings
    │   │   ├── Dockerfile
    │   │   └── run.sh - This bash script solves issue with permissions for a container user
    │   └── src
    │       ├── app.py - This file is the main entry point in the project. 
    │       ├── requirements.txt - This file contains a list of required Python modules to run application
    │       ├── static
    │       │   ├── css
    │       │   │   └── main.css
    │       │   └── img
    │       │       └── bix.jpg
    │       ├── templates - Flask view teplates
    │       │   └── base.html
    │       ├── test_requirements.txt - This file contains a list of required Python modules to runt tests
    │       └── tests
    │           ├── __init__.py
    │           └── tests.py
    ├── init.sh 
    └── sonar-project.properties - This file contains SonarQube configuration settings
    
## Frameworks used
- [Flask](http://flask.pocoo.org/)
- [Nose](https://nose.readthedocs.io/en/latest/)

## Usage - how do you start after you provisioned this quickstarter
The project should be started automatically by OpenShift. Server should be started
on the port 8080 in the debug mode.
```python
app.run('0.0.0.0', 8080, debug=True)
```
To disable a debug mode set debug to **False**.

To run application locally - specify the next command in a console:
```bash
python app.py
```
If you run application the first time, please install dependencies with the next
command:
```bash
pip install -r requirements.txt
```
It is recommended when you work with a Python project use separated environment 
for every of your project. For this purpose usually iis used 
[virtualenv](https://virtualenv.pypa.io/en/latest/) package.

```bash
# Command install virtualenv package (run only once)
pip install virtualenv
# Creates virtual environment 'venv' (will be located in the folder venv) (run only once)
virtualenv venv

# Initiate virtual environment for the project (every time)
source venv/bin/activate

# Runs installation of required modules in the virtual environment (run only once)
pip install -r requirements.txt

# Start your application
python app.py
```

## How this quickstarter is built through Jenkins
The Jenkinsfile is provisioned with this quick starter to ease CI/CD process. In Jenkinsfile, there are various stages:


- **Test** - Runs unit test cases by executing command:
    ```bash
    nosetests -v
    ```
- **PEP** 8 - Runs lint profiler by running command:
    ```bash
    pycodestyle --show-source --show-pep8 . &&
    pycodestyle --statistics -qq .
    ```
- **Build** - Builds the application, copies output folder dist into docker/dist folder.

## Builder Slave used
This quickstarter uses [Python](https://github.com/opendevstack/ods-project-quickstarters/tree/master/jenkins-slaves/python) builder slave Jenkins builder slave.

## Known limitations
NA
