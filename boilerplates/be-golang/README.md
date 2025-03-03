# Backend - Go (be-golang)

## Purpose of this quickstarter
Use this quickstarter when you want to use [Go](https://golang.org). Go is well
suited for CLI tools, network/operational related things and microservices.

## What files / which architecture is generated?

```
├── Jenkinsfile - Contains Jenkins build configuration
├── README.md
├── docker - Contains Dockerfile for the build
│   └── Dockerfile
├── sonar-project.properties - SonarQube Configuration
├── main.go - Example Go file
```

## Frameworks used
None, except the ODS [Jenkins Shared Library](https://github.com/opendevstack/ods-jenkins-shared-library)

## Usage - how do you start after you provisioned this quickstarter
Simply start to write Go code, e.g. by extending `main.go`. No further adjustments
should be necessary. Typically, you'd want to use Go modules:
```
go mod init example.com/project/component
```

## How this quickstarter is built through Jenkins
There are six steps:

* Check that all files are gofmt'd.
* Run SonarQube analysis.
* Run all package tests.
* Build the binary (placing it into the `docker` directory).
* Build the container image.
* Deploy.

## Builder Slave used

This quickstarter uses
[Golang builder slave](https://github.com/opendevstack/ods-project-quickstarters/tree/master/jenkins-slaves/golang).

## Known limitations
N/A
