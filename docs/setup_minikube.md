# How to run Seldon on Minikube

To test images inside Kubernets using seldon-core we can use minikube. The seldon-core documentation refers to use kind, but on our testing we had more success on minikube.

## Install Docker

```
curl -fsSL https://get.docker.com -o get-docker.sh
```
```
sudo sh get-docker.sh`
```

Manage Docker as non root:
```
sudo groupadd docker
```
```
sudo usermod -aG docker $USER
```

Reboot the system:

```
reboot
```
Test:
```
docker run hello-world
```

## Install Minikube
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
```
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

Seldon-core works on versions higher than 1.18 and < 1.25, thus we need to specify our kubernets version:
```
minikube start --kubernetes-version=v1.24.10
```
It takes some time to download all files, once finished you can check the installation by opening the dashboard:
```
minikube dashboard
```

## Install Kubectl on Linux
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
Test:
``` 
kubectl version --client
```

## Install Helm
```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
## Setup Istio

Download Istio:
```
curl -L https://istio.io/downloadIstio | sh -
```

Installation:
```
cd istio*
```
```
export PATH=$PWD/bin:$PATH
```
```
istioctl install --set profile=demo -y
```

```
kubectl label namespace default istio-injection=enabled
```

Create Gateway

```
kubectl apply -f - << END
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: seldon-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
END
```
## Install Seldon Core

```
kubectl create namespace seldon-system
```

```
helm install seldon-core seldon-core-operator \
    --repo https://storage.googleapis.com/seldon-charts \
    --set usageMetrics.enabled=true \
    --set istio.enabled=true \
    --namespace seldon-system
```

You can check the status of the installation by running:

```
kubectl get pods -n seldon-system
```

## Enable Port Fowarding

This step is necessary for every time you start your cluster.

```
kubectl port-forward -n istio-system svc/istio-ingressgateway 8080:80
```

This will forward any traffic from port 8080 on your local machine to port 80 inside your cluster.

# References

https://docs.docker.com/engine/install/ubuntu/

https://minikube.sigs.k8s.io/docs/start/

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

https://helm.sh/docs/intro/install/

https://docs.seldon.io/projects/seldon-core/en/latest/install/kind.html



