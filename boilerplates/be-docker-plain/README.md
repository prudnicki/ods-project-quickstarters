# Plain docker image (be-docker-plain)

## Purpose of this quickstarter
Use this quickstarter when you want to start from a plain dockerfile only - w/o a framework on top. 
A good usecase here is a dockerfile you found on github that you want to run with OpenDevStack features,
or that you need to "openshiftify", by setting an execution user or alike.

## What files / architecture is generated?

```
├── Jenkinsfile - Contains Jenkins build configuration
├── README.md
├── docker - Contains Dockerfile for the build
│   └── Dockerfile
├── sonar-project.properties  - SonarQube Configuration
```

## Frameworks used
None, except the ODS [jenkins shared library](https://github.com/opendevstack/ods-jenkins-shared-library)

## Usage - how do you start after you provisioned this quickstarter
Amend the generated `Dockerfile` as needed.

## How this quickstarter is built through jenkins
The shared library is used as is - whatever is in the `/docker` folder is passed to `oc start build` as docker context.
In case you want to run testing, plug into `stageUnitTest`.
```
def stageBuild(def context) {
  stage('Build') {
    // copy any other artifacts if needed
    // sh "cp -r build docker/dist"
    // the docker context passed in /docker
  }
}

def stageUnitTest(def context) {
  stage('Unit Test') {
    // if needed add your unit tests here
  }
}
```

## Builder Slave used 
none

## Known limitations
N/A
