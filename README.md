# Kubernetes Crash Course

### Scope
We'll be learning the basics of Kubernetes by locally hosting a toy Go server alongside Nginx.

[./backend/server.go](./backend/server.go) implements a single API endpoint. To start, go ahead and run it locally like so:

````bash
cd backend/
go run server.go

# in another terminal hit the API endpoint
curl http://localhost:8000/fruit
````

Or visit http://localhost:8000/fruit in your browser.


### Getting Started

With that out of the way let's set up your Kubernetes dev environment.

1. Setup [Minikube](https://minikube.sigs.k8s.io/docs/start/)

````bash
# start a kubernetes single node "cluster" (inside Docker)
minikube start --driver docker

# now ensure kubectl is pointed to minikube
kubectl config get-contexts

# you should see the following:
# CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
# *         minikube   minikube   minikube   default
````

2. Build Docker image

````bash
docker build ./backend -t backend:latest

# now make the image available inside minikube
minikube image load backend:latest

# optionally you can test running the image locally
docker run -p 8000:8000 -t backend
````

3. Launching your first pod

Our goal is to run server.go inside a `pod`, which is a thin abstraction around a container in Kubernetes.

Using the [Kubernetes docs](https://kubernetes.io/docs/concepts/workloads/pods/) as a reference, we can describe our pod (in yaml) like so:


````yaml
apiVersion: v1
kind: Pod

metadata:
  name: backend

spec:
  containers:
    - name: backend
      image: backend:latest
      imagePullPolicy: Never # this is a local image
      ports:
      # indicate this pod shall expose port 8000
      - containerPort: 8000
````

Copy that into ./backend.yaml (in the root of this repository) then run:

````bash
# load our pod's definition into kubernetes
kubectl apply -f backend.yaml

# now you can check the status of the pod
kubetl get pods # repeat until status show "Running"

# and stream the pod's logs
kubectl logs -f pods/backend
````

The pod is currently only accessible inside minikube so forward the port locally to access.

````bash
# expose on localhost:8000
kubectl port-forward pods/backend 8000:8000
````
