# Devoriales - Kubectl Cheatsheet

[Blog Post](https://devoriales.com/post/226)

We created this  Kubectl Cheatsheet for you to use as a quick reference guide. It contains the most common commands used when working with Kubernetes.
It is a work in progress and we will be adding more commands as we go along. If you have any suggestions, please feel free to open an issue or a pull request.
## Table of Contents

- [What is Kubectl](#what-is-kubectl)
- [How The Communication Works](#how-the-communication-works)
- [How to Install Kubectl](#how-to-install-kubectl)
- [Cluster Information](#cluster-information)
- [Resource Management](#resource-management)
- [Inspecting Resources](#inspecting-resources)
- [Creating and Updating Resources](#creating-and-updating-resources)
- [Deleting Resources](#deleting-resources)
- [Scaling Deployments](#scaling-deployments)
- [Exposing Deployments](#exposing-deployments)
- [Managing Rollouts](#managing-rollouts)
- [Working with Logs](#working-with-logs)
- [Executing Commands in Containers](#executing-commands-in-containers)
- [Port Forwarding](#port-forwarding)
- [Labeling and Annotating Resources](#labeling-and-annotating-resources)
- [Configuring Kubectl](#configuring-kubectl)
- [Dry Run](#dry-run)
- [Looping Through Resources](#looping-through-resources)
- [Create Resourses with `cat` command](#create-resourses-with-cat)
- [Flattening Kubeconfig](#flattening-kubeconfig-files)
- [Getting API Resources](#getting-api-resources)

## What is Kubectl
This is the tool that most Kubernetes engineers primarily use when interacting with a Kubernetes cluster.

Behind the scenes, kubectl interacts directly with the Kubernetes API, converting the commands you type into API requests, which are then executed on the Kubernetes cluster.

With kubectl, you can perform a wide range of operations on the cluster, such as creating, deleting, and updating deployments, exposing your application, checking the logs of your running pods, and monitoring the health and capacity of your nodes and the overall cluster health.

To begin using kubectl, you must first install it on your local machine. Please refer to the next section for installation instructions.


## How The Communication Works

The communication between kubectl and the Kubernetes cluster is based on RESTful HTTP requests and responses.
The following diagram illustrates how kubectl interacts with Kubernetes resources:

```
   Me
                                                     +--------------------------+
   x x                                               |    Kubernetes cluster    |
  x   x                                              |                          |
   x x                                               |      +---------+         |
    x                +---------+                     |      |         |         |
    x   commands     |         | GET, POST, PUT etc. |      |   api   |         |
    x -------------->| kubectl +---------------------+------+  server |         |
   x x               |         <---------------------+------+         |         |
  x x x <--------    +---------+     Response        |      +---------+         |
 x  x  x   displays                                  |                          |
x   x   x  commands                                  |                          |
    x                                                |                          |
    x                                                |                          |
    x                                                +--------------------------+
    x
```

When you issue a command using kubectl, the tool reads the kubeconfig file on your local machine and obtains all the necessary information to authenticate with the Kubernetes API server. This file includes details about the Kubernetes server's address, your credentials, and context information.

The connection between kubectl and the API server is usually secured by TLS encryption to ensure that no tampering or eavesdropping occurs.

Authentication and authorization occur before the API server processes your request. The API server validates your credentials. There are various ways Kubernetes can authenticate you, which are covered in detail in the following article: Kubernetes - RBAC And Admission Controllers

Once you have been authenticated and authorized, the kubectl command is translated into an API request that conforms to the Kubernetes API specification.

The request is typically a RESTful HTTP request, containing a specific method (GET, POST, PUT, DELETE, etc.). It also targets the relevant API endpoint (e.g., /api/v1/namespaces/default/pods) and includes the necessary data in JSON format (if applicable). The request is then sent over the secure connection to the Kubernetes API server, along with the required headers and your credentials.

Upon receiving the API request, the API server processes it and performs the necessary operations, such as creating or updating resources.

Kubectl receives the API response from the Kubernetes API server, which typically includes data in JSON format, log messages, and more.

Finally, kubectl displays the message to the user in the terminal.

## How To Install Kubectl
Download the latest version:

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
```
Download a specific version:

```
curl -LO "https://dl.k8s.io/release/v1.24.0/bin/darwin/amd64/kubectl"
To make the kubectl binary executable:

chmod +x ./kubectl
To be able to run kubectl without specifying the path, move the kubectl binaries to a file location on your system PATH:

sudo mv ./kubectl /usr/local/bin/kubectl
sudo chown root: /usr/local/bin/kubectl
to check the version

kubectl version --client

## Output

Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.7", GitCommit:"1dd5338295409edcfff11505e7bb246f0d325d15", GitTreeState:"clean", BuildDate:"2021-01-13T13:23:52Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"darwin/amd64"}
```


## Cluster Information
With the following command, we can get information about the cluster, like the version of Kubernetes that is running, the IP address of the master, and the names of the nodes in the cluster.

```bash
kubectl cluster-info
```

## Resource Management
The following commands are used to manage resources in the cluster

```bash
kubectl get nodes # list all nodes in the cluster
kubectl get pods # list all pods in the namespace or with --all-namespaces flag
kubectl get services
kubectl get deployments
kubectl get secrets
kubectl edit <resource-type> <resource-name> # edit a resource in the default editor
watch -n 2 kubectl get pods  # Continuously monitor pod status every X seconds
```

Flags:
```bash
--all-namespaces # list the requested object(s) across all namespaces
--watch # after listing/getting the requested object, watch for changes
-o # output format example -o yaml or -o json
--show-labels # show labels in the last column
--selector # filter results by label selector example --selector env=prod
--sort-by # sort list types using a jsonpath expression example --sort-by='{.metadata.name}'
--field-selector # filter results by a field selector example --field-selector metadata.name=nginx
--no-headers # when using the default or custom-column output format, don't print headers (default print headers) 
--output-watch-events # output watch event objects when --watch or --watch-only is used example kube get pods --watch --output-watch-events
```

## Inspecting Resources
The following commands are used to inspect resources in the cluster.
Also we have included the `kubectl explain` command which is used to get the documentation of a resource type.


```bash
kubectl describe nodes <node-name>
kubectl describe pods <pod-name>
kubectl describe services <service-name>
kubectl describe deployments <deployment-name>
kubectl explain <resource-type> # get the documentation of a resource type example kubectl explain pods
```

## Creating and Updating Resources
With the following commands, we can create and update resources in the cluster.

```bash
kubectl create -f <file-name>
kubectl apply -f <file-name>
```

## Deleting Resources
With the following commands, we can delete resources in the cluster. 

```bash
kubectl delete pods <pod-name>
kubectl delete services <service-name>
kubectl delete -f <file-name>
kubectl delete deployment <deployment-name> # delete a deployment
kubectl delete namespace <namespace-name> # delete a namespace
```

## Scaling Deployments
With the following command, we can scale your deployments:

```bash
kubectl scale deployment <deployment-name> --replicas=<number-of-replicas>
```

## Exposing Deployments

In this section, we'll learn how to expose Kubernetes deployments using kubectl with different service types: ClusterIP, NodePort, and LoadBalancer. 
Exposing a deployment allows us to make the application accessible to users or other services, either within the cluster or externally.

```bash
kubectl expose deployment <deployment-name> --type=ClusterIP --port=<port> # Exposes the service on a cluster-internal IP. Choosing this value makes the service only reachable from within the cluster.
kubectl expose deployment <deployment-name> --type=NodePort --port=<port>
kubectl expose deployment <deployment-name> --type=LoadBalancer --port=<port> # In cloud providers that support load balancers, an external IP address would be provisioned to access the service.
```

The loadbalancer service type is only supported in cloud providers that support load balancers.

## Managing Rollouts
With the following commands, we can manage the rollouts of our deployments. 
Rollouts are an essential part of the deployment process, as they allow you to update your applications while minimizing downtime and ensuring  transitions between versions.

```bash
kubectl rollout status deployment <deployment-name> # Check the status of a rollout
kubectl rollout history deployment <deployment-name> # Check the history of a rollout
kubectl rollout undo deployment <deployment-name> # Rollback to the previous revision
kubectl rollout undo deployment <deployment-name> --to-revision=<revision-number> # Rollback to a specific revision
```

## Working with Logs
With the following commands, we can work with the logs of our pods:

```bash
kubectl logs <pod-name> -n <namespace> # print the logs for a pod in a namespace
kubectl logs -f <pod-name> # follow the logs
kubectl logs -p <pod-name> # print the logs for the previous instance of the container in a pod if it exists
```

## Executing Commands in Containers
Sometimes we need to execute commands in a container:

```bash
kubectl exec <pod-name> -- <command> # execute a command in a container
kubectl exec -it <pod-name> -- <command> # execute a command in a container interactively
# enter the container's shell
kubectl exec -it <pod-name> -- /bin/bash # bash shell, very common to get into a container
# copy files to and from a container
kubectl cp <pod-name>:<path-to-file> <path-to-local-file> -c <container-name>
kubectl cp <path-to-local-file> <pod-name>:<path-to-file> -c <container-name>
```


## Port Forwarding
With the following command, we can forward a local port to a port on a pod or service.
This is very handy if the application is not exposed to the outside world, but you still want to access it.


```bash
kubectl port-forward <pod-name> <local-port>:<container-port>
kubectl port-forward <service-name> <local-port>:<container-port>
```

## Labeling and Annotating Resources
With the following commands, we can add labels and annotations to resources in the cluster.
This is useful for organizing and grouping resources, as well as for attaching metadata to resources.

```bash
kubectl label pods <pod-name> <label-key>=<label-value>
kubectl annotate pods <pod-name> <annotation-key>=<annotation-value>
```

## Taints and Tolerations
Taints and tolerations are a way to control how pods are scheduled on nodes. 
For example, we can use taints to prevent pods from being scheduled on nodes that have sensitive data.

```bash
kubectl taint nodes <node-name> <key>=<value>:<effect> # taint a node
kubectl taint nodes <node-name> <key>:<effect> # remove a taint from a node
kubectl taint nodes <node-name> <key>:NoSchedule # prevent pods from being scheduled on the node
kubectl taint nodes <node-name> <key>:NoExecute # prevent pods from being scheduled on the node and evict existing pods
kubectl taint nodes <node-name> <key>:PreferNoSchedule # prefer not to schedule pods on the node
```

Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.

Example pod spec with tolerations:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
    containers:
    - name: nginx
        image: nginx
    tolerations:
    - key: "key"
        operator: "Equal"
        value: "value"
        effect: "NoSchedule"
```



## Configuring Kubectl

```bash
kubectl config view # display merged kubeconfig settings or a specified kubeconfig file
kubectl config view --minify --output 'jsonpath={..namespace}' # display the namespace within the context
kubectl config current-context # display the current-context
kubectl config use-context <context-name> # set the current-context in a kubeconfig file
kubectl config set-context <context-name> # set a context entry in kubeconfig
kubectl config set-cluster <cluster-name> # set a cluster entry in kubeconfig
kubectl config set-credentials <user-name> # set a user entry in kubeconfig
kubectl config unset users.<name> # unset a user entry in kubeconfig
```

### set a cluster entry in kubeconfig
```bash
kubectl config set-cluster <cluster-name> \
  --certificate-authority=<path-to-ca-file> \
  --embed-certs=<true|false> \
  --server=<address-and-port-of-api-server> \
  --kubeconfig=<path-to-kubeconfig-file>
```

## Dry Run
Dry run allows you to preview the changes that will be made to the cluster without actually making them. This is useful for debugging and testing.


```bash
kubectl apply --dry-run=client -f <file-name>
```

## Looping Through Resources
Sometimes you need to loop through resources. For example, you want to delete all pods in a namespace. we can use the following commands to do that.

```bash
kubectl get pods -o name | cut -d/ -f2 | xargs -I {} kubectl delete pod {}
for pod in $(kubectl get pods -o name | cut -d/ -f2); do kubectl delete pod $pod; done
```

Loop through secret data items and print them out

```bash
 kubectl get secrets -o json | jq -r '.items[] | .metadata.name as $name | .data | to_entries[] | "\($name) \(.key): \((.value|@base64d))"'
```

## Create Resourses with `cat` command

With `cat` we can create resources from stdin 

```bash
cat <<EOF | kubectl create -f -
# create a n nginx pod
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
    containers:
    - name: nginx
        image: nginx
EOF
```

## Getting API Resources


```bash
kubectl api-resources
kubectl api-resources --namespaced=true # namespaced resources
# how to get curl call from kubectl with log level 9
kubectl get pods -v=9  2>&1 | grep curl # get curl call for a specific resource
kubectl get pods -v=9  2>&1 | grep curl | sed 's/.*curl -k -v -XGET/https/' | sed 's/ -H .*//' # get curl call for a specific resource

```

## Flattening Kubeconfig Files

Assume that you have multiple kubeconfig files and you want to merge them into one file. You can use the following command to do that.

```bash
KUBECONFIG=~/.kube/config:~/.kube/config2:~/.kube/config3 kubectl config view --flatten > ~/.kube/merged-config
```
then you can run the following command to use the merged kubeconfig file:

```bash
export KUBECONFIG=~/.kube/merged-config
```

## Resources

* [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [Kubernetes Documentation](https://kubernetes.io/docs/home/)
