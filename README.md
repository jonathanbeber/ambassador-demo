# Setup

This demo expects a running [kubernetes](https://kubernetes.io) environment with [helm](https://helm.sh) installed. To setup it fast and easily (non-production), the following commands can be used: 

```
minikube start --kubernetes-version=v1.14.2 -p ambassador
kubectl apply -f 01_rbac_helm.yaml
helm init
```
In this example, we're using [minikube](https://github.com/kubernetes/minikube). You can check kubernetes [official docs](https://kubernetes.io/docs/tasks/tools/install-minikube/), for more details on how to install it.

# Ambassador installation

The following steps install [Ambassador](https://www.getambassador.io/) using the [official helm charts](https://github.com/helm/charts/tree/master/stable/ambassador):

```
kubectl create namespace ambassador
helm upgrade --install --namespace=ambassador --values=02_ambassador_values.yaml ambassador stable/ambassador
```

To check ambassador deployment status, you can check for the resources status:

```
kubectl get all -n ambassador
```

The following command will map ambassador admin ui for your computer:
```
kubectl port-forward -n ambassador svc/ambassador-admins 8877
```

The interface is accessible now [here](http://localhost:8877/ambassador/v0/diag/) and it's used mainly for debugging purposes.

# Simple Service v1

We're going to deploy a simple-service and its mappings for ambassador:

```
kubectl apply -f 03_deploy_v1.yaml
kubectl apply -f 04_mapping_v1.yaml
```

Again, in the [admin ui](http://localhost:8877/ambassador/v0/diag/) address is possible to see Ambassador created mapping resources.

In order to access the `simple-service` trough Ambassador it's necessary to map its service to our computer as well. The following command it's just necessary because this demo it's not running on a cloud environment. On a cloud environment we'd be accessing it trough a Cloud Load Balancer solution.

```
kubectl port-forward svc/ambassador -n ambassador 8000:80
```

Test the access to `simple-service` with a simple http request: `curl http://localhost:8000/simple-service/v1/`

# Simple Service v2

Now, we're going to create a new version of the same service and its mappings:

```
kubectl apply -f 05_deploy_v2.yaml
kubectl apply -f 06_mapping_v2.yaml
```

Again, in the [admin ui](http://localhost:8877/ambassador/v0/diag/) address is possible to see Ambassador created mapping resources.
Test the access to `simple-service` `v2` with a simple http request: `curl http://localhost:8000/simple-service/v2/`

# Simple Service Canary

This example, try to represent a simple and manually [Canary Release](https://martinfowler.com/bliki/CanaryRelease.html):

```
kubectl apply -f 07_mapping_weight.yaml
```

The [admin ui](http://localhost:8877/ambassador/v0/diag/) will present to mappings with its defined weights. We can test it making multiple http requests to the mapped endpoint:
```
for i in {1..20}; do curl http://localhost:8000/simple-service/; sleep 0.5; echo; done
```

## Change weights

Let's edit the file [07_mapping_weight.yaml](./07_mapping_weight.yaml) changing the mapping weights. This action could be performed since the team is confident with the change, increasing the percentage of clients accessing it. After it, apply this changes running `kubectl apply -f 07_mapping_weight.yaml` again.

Then check if the applied configs are reflected on the [admin ui](http://localhost:8877/ambassador/v0/diag/). Once again, it's possible to check the configuration running multiple requests to the service:

```
for i in {1..20}; do curl http://localhost:8000/simple-service/; sleep 0.5; echo; done
```

# Headers

As a final example, it's possible to route traffic based on the request's headers. Apply the following mapping example:

```
kubectl apply -f 08_mapping_header.yaml
```

Check if the mapping is on the [admin ui](http://localhost:8877/ambassador/v0/diag/) and modify your http request to include the header defined on [08_mapping_header.yaml](./08_mapping_header.yaml):

```
for i in {1..20}; do curl -H "am-i-a-test: true" http://localhost:8000/simple-service/; sleep 0.5; echo; done
```

# Next steps

Refer to the [Ambassador official documentation](https://www.getambassador.io/docs) for more features and details using Ambassador.

# Cleaning up

```
minikube delete -p ambassador
```
