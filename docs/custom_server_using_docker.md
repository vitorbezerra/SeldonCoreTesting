# Create Custom Server Image with Docker

Seldon Core offers to use custom images as servers, in this file we are going to use a custom server made using the Seldon Core Python API.

## Prepare Image

### Server
First we need to create the server that should have this structure:

```python
class MyModel(object):
    """
    Model template
    """

    def __init__(self):
        """
        Add any initialization parameters. These will be passed at runtime from the graph definition parameters defined in your seldondeployment kubernetes resource manifest.
        """
        print("Initializing")

    # this function is called automaticly after the __init__ when using seldon
    def load(self):
        print("load")
        # load your model here
        self.ready = True

    def predict(self,X,features_names):
        """
        Return a prediction.

        Parameters
        ----------
        X : array-like
        feature_names : array of feature names (optional)
        """
        print("Predict called - will run identity function")
        return [X]

```
### Requirements
Populate a requirements.txt with any software dependencies your code requires. At a minimum the file should contain:
```
seldon-core 
```
### Define Dockerfile
The Dockerfile should follow the structure bellow:


```Dockerfile
# image to run
FROM python:3.7-slim

# copy source code
COPY . /app/
WORKDIR /app

# install normal requirements and add seldon-core
RUN pip3 install -r requirements.txt

# Port for GRPC
EXPOSE 5000
# Port for REST
EXPOSE 9000

# Define environment variables
ENV MODEL_NAME MyModel
ENV SERVICE_TYPE MODEL

# Changing folder to default user
RUN chown -R 8888 /app

CMD exec seldon-core-microservice $MODEL_NAME --service-type $SERVICE_TYPE
```

## Publish Image on Minikube

By default kubernets download files from DockerHub, to use local images we need to load the image to minikube localy

If you already build the image using Docker you can load the image to minikube with the command:

```
minikube image load <IMAGE_NAME>
```

Or you can build the image directly to minikube by using:

```
minikube image build -t <IMAGE_NAME> .
```

## Deployment on Seldon Core

After our custom image is published to minikube we can start the deployment on Seldon Core, First create your namespace:

```
kubectl create namespace seldon
```

Then we are going to create a file with the specifics of our deployment:

```
touch deployment_seldon.yaml
```

```yaml
apiVersion: machinelearning.seldon.io/v1
kind: SeldonDeployment
metadata:
  name: <model-name>
  namespace: <model-namespace>
spec:
  name: <name>
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - name: classifier
          image: <image-name:version>
          imagePullPolicy: Never
    graph:
      name: classifier
    name: default
    replicas: 1
```

Where:
* `<model-name>` is the name of our deployment, we can use for example the name `iris-model`;
* `<model-namespace>` is the namespace where the model will be deployed (in this example the value should be `seldon`);
* `<name>` is the name of the component of the deployment;
* `<image-name:version>` is the name of our Docker image;
* `imagePullPolicy` is set to `Never` because we are using a local image, if you want to use a image published on DockerHub remove this flag.

Then run the following comand to deploy your model:

```
kubectl apply -f deployment_seldon.yaml
```

## Send API Requests

You can test using curl, remember to be running the Istio port fowarding:

```
kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80
```

```
curl -X POST http://localhost:8080/seldon/seldon/iris-model/api/v1.0/predictions \
    -H 'Content-Type: application/json' \
    -d '{ "data": { "ndarray": [[1,2,3,4]] } }'
```


You can also test by accessing the web browser version on the following URL on your browser: http://localhost:8080/seldon/seldon/iris-model/api/v1.0/doc/

![](https://raw.githubusercontent.com/SeldonIO/seldon-core/master/doc/source/images/rest-openapi.jpg)

## Removing Deployment

You can remove the deployment by running.

```
kubectl delete -f deployment_seldon.yaml
```

# References
https://levelup.gitconnected.com/two-easy-ways-to-use-local-docker-images-in-minikube-cd4dcb1a5379

https://docs.seldon.io/projects/seldon-core/en/latest/workflow/github-readme.html

https://docs.seldon.io/projects/seldon-core/en/latest/python/python_wrapping_docker.html
