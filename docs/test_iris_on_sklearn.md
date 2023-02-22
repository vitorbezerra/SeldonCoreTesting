# Testing Installation

To test our seldon core installation we are going to install a iris model implemented on SKLEARN using the SKLEARN server from Seldon Core.

## Creating deployment using SKLEARN server

Create the name space of your deployment

```
kubectl create namespace seldon
```

Deploy the model
```yaml
kubectl apply -f - << END
apiVersion: machinelearning.seldon.io/v1
kind: SeldonDeployment
metadata:
  name: iris-model
  namespace: seldon
spec:
  name: iris
  predictors:
  - graph:
      implementation: SKLEARN_SERVER
      modelUri: gs://seldon-models/v1.16.0-dev/sklearn/iris
      name: classifier
    name: default
    replicas: 1
END
```

You can verify the deployment status on the Kubernets Dashboard or running:

```
kubectl get pods -n seldon
```

## Send API Requests

You can test using curl, remember to be running the Istio port fowarding:

```
curl -X POST http://localhost:8080/seldon/seldon/iris-model/api/v1.0/predictions \
    -H 'Content-Type: application/json' \
    -d '{ "data": { "ndarray": [[1,2,3,4]] } }'
```

Then the output will be:

```
{
   "meta" : {},
   "data" : {
      "names" : [
         "t:0",
         "t:1",
         "t:2"
      ],
      "ndarray" : [
         [
            0.000698519453116284,
            0.00366803903943576,
            0.995633441507448
         ]
      ]
   }
}
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

https://docs.seldon.io/projects/seldon-core/en/latest/workflow/github-readme.html