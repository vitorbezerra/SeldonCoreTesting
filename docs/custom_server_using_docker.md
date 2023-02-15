# Create Custom Server Image with Docker

Seldon Core offers to use custom images as servers, in this file we are going to use a custom server made using the Seldon Core Python API.

## Prepare Image

# TODO: Explaing interface and setup files

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

```
apiVersion: machinelearning.seldon.io/v1
kind: SeldonDeployment
metadata:
  name: iris-model
  namespace: model-namespace
spec:
  name: iris
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - name: classifier
          image: sklearn_iris:0.1
          imagePullPolicy: Never
    graph:
      name: classifier
    name: default
    replicas: 1
```

Where `iris-model` is the name of our model, `sklearn_iris:0.1` is the name of our Docker image and `imagePullPolicy` is set to never because we are using a local image, if you want to use a image published on DockerHub remove this flag.

Then run the following comand to deploy your model:

```
kubectl apply -f deployment_seldon.yaml
```

## Send API Requests

You can test using curl, remember to be running the Istio port fowarding:

```
curl -X POST http://localhost:8080/seldon/seldon/iris-model/api/v1.0/predictions \
    -H 'Content-Type: application/json' \
    -d '{ "data": { "ndarray": [[1,2,3,4]] } }'
```


You can also test by accessing the web browser version on the following URL on your browser: http://localhost:8080/seldon/seldon/iris-model/api/v1.0/doc/

![](https://raw.githubusercontent.com/SeldonIO/seldon-core/master/doc/source/images/rest-openapi.jpg)

## Removing Deployment

You can remove the deployment by deleting the namespace.

WARNING: This will delete all you deployments on your namespace

```
kubectl delete namespace seldon
```

# References
https://levelup.gitconnected.com/two-easy-ways-to-use-local-docker-images-in-minikube-cd4dcb1a5379

https://docs.seldon.io/projects/seldon-core/en/latest/workflow/github-readme.html
