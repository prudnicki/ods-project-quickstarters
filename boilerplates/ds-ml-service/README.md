# Data Science Industrialization Boilerplate
This boilerplate enables data scientists to develop, serve, version models within a CI/CD
pipeline hosted on OpenShift with the goal in mind that one does not have to take care/change
much of the needed pipeline and infrastructure.

## Basic Setup ##
The boilerplate provides a two pod setup in OpenShift, one pod for training service and one pod for
prediction service.

### Docker ###
Each service has their own `Dockerfile`, under `docker-prediction` and `docker-training`
respectively,
  which can be changed according to your requirements in regards to operating system dependencies.

- The `docker-training` container provides a pod that is able to reproduce/retrain the model that is
developed in the current commit either locally on OpenShift or execute the training on a remote
linux system using ssh. The training process is wrapped into a flask server to be able to monitor
 and possible restart the training process. Moreover, the training service offers an endpoint
 for downloading the created model afterwards. Additionally, unittests and
 integration tests are executed on the training pod, in order to not depend on operating
 dependencies in the jenkins slave.

 - The `docker-prediction` container provides a simple flask service for getting new predictions out
 of your model by making json posts to the prediction REST endpoint.
 The prediction service downloads the newly trained model from the training pod after startup.

### Jenkins ###
The `Jenkinsfile` organizes the correct succession of spinning up the training, executing it and
starting the new deployment of the prediction service.
Additionally, it triggers unittest ensuring the code is functionally before a new training
process is started.
 Moreover, integration tests are run against the reproduced model wrapped into the prediction
 REST endpoint to
ensure that the reproduced model (performance) behaves as expected also when wrapped in the flask
service.

### External Files ###
External files that are needed either for building your model or docker
images are stored
under
`resources`. For demonstration purposes a training and test csv file is stored in resources. This
 approach has to be reevaluated for each new use case, considering data size and confidentiality.

### src -  the heart of your service ###
The `src` folder contains the infrastructure coded needed for providing the services in OpenShift
 in `src/services`. Custom code for developing your prediction service is organized in the
 `src/model`
 package.
 In the (common) `src/requirements.txt` you can specify python dependencies for training, prediction
 and tests. To keep it simple, there is only one requirements.txt for both pods.


### test ###
The `test` directory mirrors the structure of the `src`, either for unittests or integration
tests using the python unittest framework.


## How to Code Your Own Models ##
To run your own customized models there is usually no need to change either the `Jenkinsfile`,
OpenShift setup or the training and prediction microservices.
Custom model code will go under `src/model` and can be organized in custom packages like
showcased with the `src/model/data_cleaning` and `src/model/feature_prep`. In general, it can be
organized as
the users prefers.
There are no further restrictions for developing the in the style you want, for the exception to
provide the mandatory functions and attributes in `src/model/model_wrapper.py` for the `ModelWrapper
class:
- `prep_and_train`: is called by the train script (which one can customize) and expects a
pandas dataframe (current implementation). The train script is called by the training service
- `prep_and_predict`: is called by the predict endpoint service from the prediction service. It
consumes the json post as a dictionary. The predict
endpoint executes `prep_and_predict`.
- Good practice: `source_features`, specifying the name that are used a input for the model. This
features
include really the source columns from which also more complicated features are derived within
the model boundaries
- Good practice: `target_variable`, name of the variable that should be used as target for a
possible
supervised approach.

As well as the `train` function in the `src/trainer.py`. It specifies how the model should be
trained.

Make sure your specified all dependencies in the `requirements.txt`.

## How to Develop your Model Locally ##
It is recommended to develop your code against the python interpreter & dependencies specified in the docker images.
This can be easily achieved, either by using an IDE that supports that (e.g. PyCharm) or by doing
manually in the docker container.

The whole setup (prediction and training service) can be tested using the attached docker-compose

### Local deployment (docker-compose) - testing the services ###

Any new change: Generate `/dist` folder for both training and prediction containers
```
./build.sh
```

Setup the right values for the environment variables in `docker-compose.yml` file wherever there
is a definition like `THIS_PARAMETER_SHOULD_BE_NOT_COMMITED`
```
vim docker-compose.yml
```

Build the images defined in `docker-compose.yml`
```
docker-compose build
```

Run the images defined in `docker-compose.yml`
```
docker-compose up
```

When done: stop the containers
```
[Ctrl+C]
```

## Data Versioning ##
In order to ensure complete reproducibility, in case train and/or test data can't be committed to
 a git repository due to size or confidentiality/data privacy considerations, data versioning can
  be achieved using the built in [dvc](https://dvc.org) data version capabilities.`
  Moreover, technical user account is needed so that the CI/CD pipeline is able to pull the data
  dependencies from the remote data versioning repository.

Do the following steps in order to make use of the data versioning capabilities
1. Initialize the quickstarter repository as a `dvc` repository:
```dvc init```
2. Setup the a remote repository on a remote ssh machine, e.g. Data Lake
```dvc remote add <remote_name>  ssh://<remote_server>:/<path_to_storage>```
3. Configure authentification. For local development you can set your own user account, assuming
it has access to `<path_to_storage>` or use a technical user account.
```dvc remote modify <remote_repository_name> user <technical_user_account>```
and set the prompt for password, so that you don't commit your password to the repository
```dvc remote modify <remote_name> ask_password True```
4. Start adding files that should be tracked by data versioning
``` dvc add <some_file>```
this will create a new file with meta information about `<some_file>` called `<some_file>.dvc`.
This meta file needs to be tracked with git, so that it is ensured that each git commit is linked
 with a specific data version
 ```git add .gitignore <some_file>.dvc```
5. Modify your train() and potentially the integration tests to pull the data dependencies from
the remote repository. A helper class is provided in `src/services/remote/dvc/data_sync.py` that can
 be used as follows:
 ```from services.infrastructure.remote.dvc.data_sync import DataSync ```
 ```syncer = DataSync(dvc_data_repo, dvc_ssh_user, dvc_ssh_password)```
```syncer.pull_data_dependency(file_name)```
6. Commit your code and push the data versioned files to the remote repository
````git commit````
```dvc push -r <remote_name>```
```git push```

7. In order for a successful Jenkins build, the following environment variables need to be set in
 the `training` pod deployment: `DSI_DVC_REMOTE`, `DSI_SSH_USERNAME`, `DSI_SSH_PASSWORD





## Example & Example Dataset ##
An example implementation of a custom model is given in `src/model`, to demonstrate how to organize
custom code.
A Logistic Regression using scikit-learn with some (unnecessary) feature cleaning and engineering
 is trained on the iris data flower set.

**Iris flower data set**. (n.d.). In Wikipedia. Retrieved January 7, 2019, from [Iris_flower_data_set](https://en.wikipedia.org/wiki/Iris_flower_data_set)

## Structure of the quick starter ##

* Training
    * Build Config
		* name: `<componentId>-training-service`
		* variables: None
	* Deployment Config
		* name: `<componentId>-training-service`
		* variables:
			* `DSI_EXECUTE_ON`: LOCAL
			* `DSI_TRAINING_SERVICE_USERNAME`: auto generated username
			* `DSI_TRAINING_SERVICE_PASSWORD`: auto generated password
	* Route: None by default - no routes exposed to internet
* Prediction
	* Build Config
		* `name`: `<componentId>-prediction-service`
		* `variables`: None
	* Deployment Config
		* `name`: `<componentId>-prediction-service`
		* `variables`:
			* `DSI_TRAINING_BASE_URL`: `http://<componentId>-training-service.<env>.svc:8080`
			* `DSI_TRAINING_SERVICE_USERNAME`: username of the training service
			* `DSI_TRAINING_SERVICE_PASSWORD`: password of the training service
			* `DSI_PREDICTION_SERVICE_USERNAME`: auto generated username
			* `DSI_PREDICTION_SERVICE_PASSWORD`: auto generated password
	* Route: None by default - no routes exposed to internet

### Remote Training ###

Remote training allows you to run your training outside of the OpenShift training pod on a linux
node using a ssh connection.
A conda environment is installed in the remote node and the requirements specified in
`src/requirements.txt` are installed.
Once this step is finished the training is executed on that node and the trained model is
transferred back to the training pod.

To enable remote training set the `DSI_EXECUTE_ON` variable in OpenShift to *SSH* and specify the
 connection information in the environment variables: `DSI_SSH_HOST`, `DSI_SSH_PORT`,
 `DSI_SSH_USERNAME` and `DSI_SSH_PASSWORD`.

The easiest approach to use the docker-compose and ssh remote training is to create a yml with environment variables for training. E.g.:

`docker-compose.ssh.yml`

```yml
version: '3'
services:
  training:
    environment:
      DSI_EXECUTE_ON: SSH
      DSI_SSH_HOST: my.remote.ssh.server.com
      DSI_SSH_USERNAME: ssh_username
      DSI_SSH_PASSWORD: ssh_password
      DSI_SSH_HTTP_PROXY: http://proxy
      DSI_SSH_HTTPS_PROXY: https://proxy
```

`docker-compose -f docker-compose.yml -f docker-compose.ssh.yml up`

The training pod starts an asynchronous training task. Only one
training task can run at a time.

## Endoints ##

### Training Endpoint ###
* `/` : Return all information about the training service
* `/start` : Starts the training.
* `/finished` : Checks if the current traning task is finished
* `/getmodel` : Download the latest trained model

### Prediction Endpoint ###
* `/predict` : Return all information about the training service
    * payload: Should be a json containing the data necessary for prediciton. The payload is not pre defined, but it is defined by the trainined model



There is not need for any kind of payload in all endpoints.

### Environment Variables for training ###

| Environment Variable | Description | Allowed Values |
|------|-------|---|
| DSI_DEBUG_MODE | Enables debug mode | true, 1 our yes for debug mode, otherwise debug is disasbled|
| DSI_EXECUTE_ON | Where the train should be executed | LOCAL, SSH |
| DSI_TRAINING_SERVICE_USERNAME | Username to be set as default username for accessing the services | string, required |
| DSI_TRAINING_SERVICE_PASSWORD | Password to be set as default password for accessing the services | string, required |
| |Following variables are applicable if `DSI_EXECUTE_ON=SSH`| |
| DSI_SSH_HOST | SSH host name where train should be executed (Only applicable if DSI_EXECUTE_ON=SSH) | host names or ip addresses |
| DSI_SSH_PORT | SSH host port where train should be executed (Only applicable if DSI_EXECUTE_ON=SSH) | port numbers (Default: 22) |
| DSI_SSH_USERNAME | SSH username for remote execution  | string\ |
| DSI_SSH_PASSWORD | SSH password for remote execution  | string |
| DSI_SSH_HTTP_PROXY | HTTP proxy url for remote execution. This is needed if the remote machine needs the proxy for download packages and resources  | string |
| DSI_SSH_HTTPS_PROXY | HTTPS proxy url for remote execution. This is needed if the remote machine needs the proxy for download packages and resources  | string |
| DSI_DVC_REMOTE | Name of the dvc remote repository that has been initialized with dvc  | string |

### Frameworks used

- python 3.6

### Environment Variables for prediction ###

| Environment Variable | Description | Allowed Values |
|------|-------|---|
| DSI_DEBUG_MODE | Enables debug mode | true, 1 our yes for debug mode, otherwise debug is disasbled|
| DSI_TRAINING_BASE_URL | The base url where the prediction service should get the model from | url (e.g. https://training.OpenShift.svc |
| DSI_TRAINING_SERVICE_USERNAME | Username of the training service | string, required |
| DSI_TRAINING_SERVICE_PASSWORD | Password of the training service | string, required |
| DSI_PREDICTION_SERVICE_USERNAME | Username to be set as default username for accessing the service | string, required |
| DSI_PREDICTION_SERVICE_PASSWORD | Password to be set as default password for accessing the service | string, required |

## How this quickstarter is built through jenkins

The build pipeline is defined in the `Jenkinsfile` in the project root. The main stages of the pipeline are:
1. Prepare build
2. Sonarcube checks
3. Build training image
4. Deploy training pod
5. Unittests
6. Execute/reproduce training either on openshift pod or in ssh remote machine
7. Integration test against the newly trained model wrapped in the flask `/prediction` endpoint
8. Build prediction image
9. Deploy prediction service


## Builder slave used

[jenkins-slave-python](https://github.com/opendevstack/ods-project-quickstarters/tree/master/jenkins-slaves/python)

## Known limitions

- Not ready for R models yet