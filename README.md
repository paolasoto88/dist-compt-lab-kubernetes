# dist-compt-lab-kubernetes
Repository for the Distributed computing lab session about Kubernetes basics

# Exercises
## Install Minikube
General steps are: 
1. Install a Hypervisor, recommended: VirtualBox. Depends on the host OS.
2. Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/): Manage kubernetes clusters through cli. Also depends on the host OS. 
3. Install [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/). Also depends on the host OS. 
4. Run a kubernetes cluster via minikube [start](https://kubernetes.io/docs/setup/minikube/)

```
# make sure that no other cluster is active.
$ minikube delete

# start minikube and then wait around 10 minutes until the cluster is set up  
$ minikube start 

# get the components of master and node
$ kubectl get pods -n kube-system 
 ```
 
 ## Deploy an App
 Taken from [here](https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/)
 
To make clear the difference between a pod and a deployment: 
1. Create the pod. 
```
$ kubectl create -f pod-example.yaml
```
2. Get the created pods across all namespaces
```
$ kubectl get pods --all-namespaces
```
3. Delete the pod 
```
$ kubectl delete pod myapp-pod 
```
4. Get again the pods, see that myapp-pod has been deleted 
```
$ kubectl get pods --all-namespaces
```
5. Create a deployment
```
$ kubectl create -f deployment-example.yaml
```
6. Verify that the deployment has been correctly created
```
$ kubectl get pods 
```
7. Delete the pod that was created, what happens?
```
$ kubectl delete pod <pod_name>
```
-> Proposed exercise, update the deployment to change the number of replicas to one, delete the pod, what happens? 

8. Explore your app
```
# get detailed information about the deployment
$ kubectl describe deployment nginx-deployment
 
# get detailed information about the pod
$ kubectl describe pod <pod_name>
 
# view the logs of the pod
$ kubectl logs <pod_name>
```

## Get into your App
Taken from [here](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/)

1. Get podname
```
$ kubectl get pods  # get podname
```
2. Activate kube-proxy to be able to communicate with the outside-world. In another terminal run:
```
$ kubectl proxy
```
3. Back to the first terminal, type: 
```
$ curl http://localhost:8001/api/v1/namespaces/default/pods/<pod_name>/proxy/
```

 -> Proposed exercise, try to access the IP address especified in `kubectl describe pod <pod_name>`, or `localhost:8080`, `localhost:80`, what happens?
 
 4. Check that we have access from the node to the pod. 
 ```
 # get pod IP addres
 $ kubectl get pods -o yaml | grep podIP
 
 # ssh into minikube node
 $ minikube ssh     
 $ curl <pod_IP>
 ```
 
 5. Expose your app
 ```
 $ kubectl expose deployment my-nginx --type="NodePort" --port 80
 $ kubectl get svc my-nginx
 $ kubectl describe svc my-nginx
 ```
 
 6. Access your application from the outside-world via `<NODE_IP>:<NODE_PORT>` where `<NODE_IP>` is the IP assigned to minikube and `<NODE_PORT>` is the port assigned via service `my-nginx`
 ```
 $ curl $(minikube ip):<NODE_PORT>
 ```
 
 -> Proposed exercise, what happens if you stop the kube-proxy? What happens if you delete the service?
 
 ## Pull an image from private repository 
 Taken from [here](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
 
 The following procedure assumes that you have a private image hosted on [docker hub](https://hub.docker.com/) (and therefore you already have a DockerID).
 
 1. Create a secret which stores your docker credentials. 
 
 ```
 $ kubectl create secret generic <secret-name> \
    --from-file=.dockerconfigjson=<path/to/.docker/config.json> \
    --type=kubernetes.io/dockerconfigjson
 ```
To Mac users: Read the last note from [here](https://kubernetes.io/docs/concepts/containers/images/#configuring-nodes-to-authenticate-to-a-private-registry). macOS saves your docker credentials into the keychain access, thus, the base64-encoded information is stored into the `credStore` . You have two alternatives: 
* Use the command line instead as specified [here](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line)
* Create the secret manually as specified [here](https://kubernetes.io/docs/concepts/configuration/secret/#creating-a-secret-manually)
```
$ echo -n '<docker_server>' | base64  # index.docker.io/v1/
$ echo -n '<docker_username>' | base64
$ echo -n '<docker_password>' | base64
$ echo -n '<docker_email>' | base64
```

2. Create a pod that uses your secret. 
```
$ kubectl create -f pod-private-example.yaml
```
