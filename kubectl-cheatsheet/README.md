# Devoriales - Kubectl Cheatsheet

[Blog Post](https://devoriales.com/post/226)

We've created this Kubectl Cheatsheet as a quick reference guide for you. It contains the most commonly used commands for working with Kubernetes. This cheatsheet is a work in progress, and we'll be adding more commands as we go along. If you have any suggestions, feel free to open an issue or submit a pull request.

## Table of Contents

- [What is Kubectl](#what-is-kubectl)
- [How The Communication Works](#how-the-communication-works)
- [How to Install Kubectl](#how-to-install-kubectl)
- [Cluster Information](#cluster-information)
- [Resource Management](#resource-management)
- [Inspecting Resources](#inspecting-resources)
- [Creating and Updating Resources](#creating-and-updating-resources)
- [Deleting Resources](#deleting-resources)
- [Run Command](#run-command)
- [Scaling Deployments](#scaling-deployments)
- [Exposing Deployments](#exposing-deployments)
- [Managing Rollouts](#managing-rollouts)
- [Horizontal Pod Autoscaling Commands](#horizontal-pod-autoscaling-commands)
- [Working with Logs](#working-with-logs)
- [Executing Commands in Containers](#executing-commands-in-containers)
- [Port Forwarding](#port-forwarding)
- [Create Secrets](#create-secrets)
  - [Create pull secret for private registry](#create-pull-secret-for-private-registry)
  - [Create a generic secret](#create-a-generic-secret)
  - [Get data from a secret](#get-data-from-a-secret)
- [Labeling and Annotating Resources](#labeling-and-annotating-resources)
- [Configuring Kubectl](#configuring-kubectl)
- [Dry Run](#dry-run)
- [Looping Through Resources](#looping-through-resources)
- [Create Resourses with `cat` command](#create-resourses-with-cat)
- [Flattening Kubeconfig](#flattening-kubeconfig-files)
- [Getting API Resources](#getting-api-resources)
- [Use Custom Columns --custom-columns](#use-custom-columns---custom-columns)
- [Use kubectl auth can-i](#use-kubectl-auth-can-i)
- [Troubleshooting](#troubleshooting)
  - [Delete stuck namespaces in Terminating state](#delete-stuck-namespaces-in-terminating-state)
- [Resources](#resources)

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

### Sort resources by --sort-by flag

```bash
kubectl get pods --sort-by=.metadata.creationTimestamp
kubectl get pods --sort-by=.status.phase
kubectl get pods --sort-by=.spec.nodeName
kubectl get pv --sort-by=.spec.capacity.storage

```

### Field Selector flag
This flag is used to filter resources based on specific fields.

```bash
kubectl get pods --field-selector=status.phase=Running # get all running pods
kubectl get pods --field-selector=spec.nodeName=<node-name> # get all pods running on a specific node
kubectl get pods --field-selector=metadata.name=<pod-name> # get a specific pod by name
kubectl get pods --field-selector=status.phase!=Running #
# delete all pods that are not in the Running state
kubectl delete pods --field-selector=status.phase!=Running
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
kubectl create -f <file-name> # This command is used to create resources in the cluster
kubectl apply -f <file-name> # This command is used to create or update resources in the cluster
```
We can also create resources directly from the command line:

```bash
kubectl create deployment <deployment-name> --image=<image-name> # create a deployment
kubectl create service <service-name> --tcp=<port>:<target-port> # create a service
kubectl create role <role-name> --verb=<verb> --resource=<resource> --resource-name=<resource-name> # create a role
kubectl create rolebinding <role-binding-name> --role=<role-name> --user=<user> # create a rolebinding
```

## Deleting Resources

With the following commands, we can delete resources in the cluster.

```bash
kubectl delete pods <pod-name> # delete a pod
kubectl delete pods --all # delete all pods in the namespace
kubectl delete services <service-name>
kubectl delete -f <file-name>
kubectl delete deployment <deployment-name> # delete a deployment
kubectl delete namespace <namespace-name> # delete a namespace
```

## Run command

With the ```run``` command, we can run a container in a pod. It is a quick way to create a pod with a container in it.

```bash
kubectl run <pod-name> --image=<image-name> # run a container in a pod
kubectl run <pod-name> --image=<image-name> --restart=Never # run a container in a pod with a specific restart policy
kubectl run <pod-name> --image=<image-name> --restart=Never --dry-run=client -o yaml # generate a pod manifest without creating it
kubectl run <pod-name> --image=<image-name> --restart=Never --dry-run=client -o yaml > pod.yaml # save the pod manifest to a file
kubectl run <pod-name> --image=<image-name> --restart=Never --dry-run=client -o yaml | kubectl apply -f - # create a pod from a generated manifest

```

Example of using overrides:

```bash
kubectl run <pod-name> --image=<image-name> --overrides='{"apiVersion":"v1","spec":{"containers":[{"name":"<container-name>","image":"<image-name>"}]}}'
kubectl run nginx --image=nginx --overrides='{ "apiVersion": "v1", "spec": { "containers": [ { "name": "nginx", "image": "nginx:1.7.9", "ports": [ { "containerPort": 80 } ] } ] } }'
```

official documentation: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_run/

## Scaling Deployments

With the following command, we can scale your deployments:

```bash
kubectl scale deployment <deployment-name> --replicas=<number-of-replicas>
```
Will it work with statefulsets? Yes, it will work with statefulsets as well.

```bash
kubectl scale statefulset <statefulset-name> --replicas=<number-of-replicas>
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

> **Note:** The revision number can be found in the output of `kubectl rollout history deployment <deployment-name>`.

## Horizontal Pod Autoscaling Commands
Horizontal Pod Autoscaling (HPA) is a powerful feature in Kubernetes that automatically scales the number of pods in a deployment based on observed CPU utilization or other select metrics.
```bash
kubectl autoscale deployment <deployment-name> --cpu-percent=<target-cpu-utilization> --min=<min-replicas> --max=<max-replicas> # Create an HPA for a deployment
kubectl get hpa # List all Horizontal Pod Autoscalers in the cluster
kubectl describe hpa <hpa-name> # Describe a specific Horizontal Pod Autoscaler
kubectl delete hpa <hpa-name> # Delete a Horizontal Pod Autoscaler
```

## Working with Images

Set the image on a deployment:

```bash
kubectl set image deployment/<deployment-name> <container-name>=<new-image>
```

or when having only one container in the pod:

```bash
kubectl set image deployment/<deployment-name> <new-image>
```

Using json patch:

```bash
kubectl patch deployment <deployment-name> -p '{"spec":{"template":{"spec":{"containers":[{"name":"<container-name>","image":"<new-image>"}]}}}}'
```

### Get the image(s) used by a pod
To retrieve the image(s) used by a pod, you can use the following command:

```bash
kubectl get pod <pod-name> -o jsonpath='{.spec.containers[*].image}'
```

## Working with Logs

With the following commands, we can work with the logs of our pods:

```bash
kubectl logs <pod-name> -n <namespace> # print the logs for a pod in a namespace
kubectl logs -f <pod-name> # follow the logs
kubectl logs -p <pod-name> # print the logs for the previous instance of the container in a pod if it exists
```

### Streamlining Log Access:
For a more streamlined approach, you can retrieve logs based on deployments or labels:

```bash
kubectl logs -f deployment/<deployment-name> # follow logs from a pod associated with a deployment
kubectl logs -f -l app=<app-name> # follow logs from all pods with a specific label
```
Using these methods, you can easily monitor logs from all pods associated with a deployment or from all pods with a specific label, saving you time and effort.

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

## Create Secrets
In Kubernetes, we have the following types of secrets:
OpaqueArbitrary user-defined data (default type)kubernetes.io/service-account-tokenServiceAccount tokenkubernetes.io/dockercfgSerialized ~/.dockercfg filekubernetes.io/dockerconfigjsonSerialized ~/.docker/config.json filekubernetes.io/basic-authCredentials for basic authenticationkubernetes.io/ssh-authCredentials for SSH authenticationkubernetes.io/tlsData for a TLS client or serverbootstrap.kubernetes.io/tokenBootstrap token data

A table of the secrets:
| Secret Type | Description |
|-------------|-------------|
| Opaque | Arbitrary user-defined data (default type) |
| kubernetes.io/service-account-token | ServiceAccount token |
| kubernetes.io/dockercfg | Serialized ~/.dockercfg file |
| kubernetes.io/dockerconfigjson | Serialized ~/.docker/config.json file |
| kubernetes.io/basic-auth | Credentials for basic authentication |
| kubernetes.io/ssh-auth | Credentials for SSH authentication |
| kubernetes.io/tls | Data for a TLS client or server |

If you just type `kubectl create secret`, it will give some hints on how to create secrets.

### Create pull secret for private registry
To create a pull secret for a private registry, you can use the following command:

```bash
kubectl create secret docker-registry <secret-name> \
  --docker-server=<registry-server> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>
```

Don't forget to add the secret to the deployment or pod spec so that the Kubernetes can use it to pull images from the private registry:

```yamlspec:
  containers:
  - name: <container-name>
    image: <image-name>
  imagePullSecrets:
  - name: <secret-name>
```

### Create a generic secret
To create a generic secret, you can use the following command:
```bash
kubectl create secret generic <secret-name> \
  --from-literal=<key>=<value> # create a secret from a literal value
kubectl create secret generic <secret-name> \
  --from-file=<path-to-file> # create a secret from a file
kubectl create secret generic <secret-name> \
  --from-env-file=<path-to-env-file> # create a secret from an env file
``` 

### Get data from a secret

Assume we have the following secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: dXNlcm5hbWU= # base64 encoded value of "username" 
  password: cGFzc3dvcmQ= # base64 encoded value of "password"
```

To get the data from the secret, we can use the following command:

```bash
kubectl get secret my-secret -o jsonpath='{.data.username}' | base64 --decode # get the username
kubectl get secret my-secret -o jsonpath='{.data.password}' | base64 --decode # get the password
```

In the following, we have a more complex example where we have a json file with multiple keys and values.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-complex-secret
type: Opaque
data:
  config.json: eyJrZXkiOiAidmFsdWUiLCAibmFtZSI6ICJ0ZXN0In0= # base64 encoded value of {"key": "value", "name": "test"}
```

To get the data from the secret, we can use the following command:

```bash
kubectl get secret my-complex-secret -o jsonpath='{.data.config\.json}' | base64 --decode | jq . # get the config.json file and decode it
```
> **Note:** The `jq` utility is used to parse the JSON output. If you don't have it installed, you can install it using your package manager (e.g., `brew install jq` on macOS or `apt-get install jq` on Ubuntu).

The reason why we need to escape the dot in the jsonpath expression is that the dot is a special character in jsonpath, and we need to escape it to access the key in the JSON object.

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
kubectl config set-context --current --namespace=<namespace> # set the default namespace for the current context
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

Sometimes you need to loop through resources.
For example, you might want to delete all pods in a namespace or get data for all secrets.

### Get all pod names in a namespace

```bash
kubectl get pods -o name | cut -d/ -f2
```

- -o name - get the name of the resource
- cut utility - cut out selected portions of each line of a file
- -d/ - use / as the delimiter
- -f2 - get the second field

Example Output: 

```bash
hostpath-provisioner-58694c9f4b-snv67
coredns-7745f9f87f-5nzv7
calico-kube-controllers-649854c487-pk4dv
calico-node-mjc66
metrics-server-56ffd4f94b-ngzc2
calico-node-t7tgg
calico-node-4wndg
```

Another way is using the jq utility. 
jq is a lightweight and flexible command-line JSON processor.


```bash
kubectl get pods -o json | jq -r '.items[].metadata.name'
```

- -o json - get the output in json format since jq works with json
- items[] - get all items in the array
- metadata.name - get the name of the metadata object

### Delete all pods in a namespace

```bash
kubectl get pods -o name | xargs -I {} kubectl delete {}
```

- xargs - build and execute command lines from standard input
- -I {} - replace {} with the input

Now we can combine the two commands to delete all pods in a namespace (this is more to show how to combine commands, as there is a better way to delete all pods in a namespace)

```bash
kubectl get pods -o name | cut -d/ -f2 | xargs -I {} kubectl delete pod {}
for pod in $(kubectl get pods -o name | cut -d/ -f2); do kubectl delete pod $pod; done
```

Easy way to delete all pods in a namespace

```bash
kubectl delete pods --all
```

### Delete all pods running on a specific node
```bash
kubectl get pods --field-selector spec.nodeName=<node-name> -o name | xargs -I {} kubectl delete {}
```

### Delete pods with a specific name pattern on a specific node

```bash
kubectl get pods --all-namespaces--field-selector spec.nodeName=<node-name> -o name | grep <name-pattern> | xargs -I {} kubectl delete {}
```


### Get data for specific secret

On MacOS we use base64 -D to decode the data

```bash
kubectl get secret <secret-name> -o json | jq -r '.data' | base64 -D
```

or if Linux, we use base64 -d
  
  ```bash
  kube get secret <secret-name> -o json | jq -r '.data' | base64 -d
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

## Use Custom Columns --custom-columns

`--custom-columns` is not very known to many people, at least from my experience.

When managing Kubernetes pods, having the ability to customize your command line output can be incredibly powerful. kubectl provides the `--custom-columns` option to do just that, enabling you to specify and format your own columns for a more tailored display of resources.

```bash
kubectl get pods -o custom-columns=NAME:.metadata.name 
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase # get multiple columns and status of the pod
```

You can have multiple columns separated by comma.

- custom-columns - output in custom columns format
- NAME: create a column with the header NAME

With the above command, you're not only fetching the pod names but also their current phase, neatly organized into the NAME and STATUS columns.

Remember, the --custom-columns option takes a comma-separated list, allowing you to define multiple columns. Each column in the list is defined by a HEADER:JSON_PATH_EXPRESSION pair:

In the following example, we will set some odd (custom) header:
  
  ```bash
   kubectl get pods -o custom-columns=MY_SUPER_PODS:.metadata.name,STATUS:.status.phase
  ```

 Output example:
  
  ```bash
MY_SUPER_PODS                                              STATUS
keda-operator-metrics-apiserver-5bb87b6cc5-hqmrb   Running
devoriales-rabbitmq-0                              Running
keda-admission-webhooks-6b4b4b64fc-4wc8b           Running
keda-operator-7fdd98c445-9ljhj                     Running
consumer-deployment-58f8855bb7-hr7px               Running
```

## Use kubectl auth can-i

This command is used to check if subjects can perform an action on a resource.
The way how you specify this is by using the following syntax:

```bash
kubectl auth can-i <VERB> <RESOURCE> --namespace <NAMESPACE> --as <USER>
```

You can also set --all-namespaces flag to check if the user can perform an action on a resource in all namespaces.

Here is an practical example how we can check if the user can create pods in the default namespace:

```bash
kubectl auth can-i create pods --namespace default
```

### Check if a service account can perform an action on a resource

The command `kubectl auth can-i` can be very handy when you want to check what a specific service account can do.
For example, you can check if the default service account in the default namespace can create pods:

```bash
kubectl auth can-i create pods --namespace default --as=system:serviceaccount:default:default
```

--as flag is used to specify the service account.
system: is a prefix for service accounts.
serviceaccount:default:default is the name of the service account in the default namespace.

Please Note! `auth can-i` assumes that your current user has the permissions to impersonate a service account. If not, you may need to adjust your RBAC policies accordingly.


## Troubleshooting

### Delete stuck namespaces in Terminating state

Full tutorial can be found on https://devoriales.com/post/332/resolve-stuck-namespaces-in-kubernetes-a-step-by-step-tutorial

First, you need to get the list of namespaces in the Terminating state:

```bash
kubectl get namespaces --field-selector status.phase=Terminating
```

Then you should check why the namespace is stuck in the Terminating state:

```bash
kubectl describe namespace <namespace-name>
```

Many times, the namespace is stuck in the Terminating state because of the finalizers. You can remove the finalizers from the namespace to force delete it:

Export the namespace to a file:

```bash
kubectl get namespace <namespace-name> -o json > namespace.json
```

Edit the namespace.json file and remove the finalizers section:

example finalizers section, remove "kubernetes" from the list:

```json
"finalizers": [
    "kubernetes"
]
```

Start the kube proxy:

```bash
kubectl proxy &
```

Then run the following command to delete the namespace:

```bash
curl -X PUT http://127.0.0.1:8001/api/v1/namespaces/<namespace-name>/finalize -H "Content-Type: application/json" --data-binary "@namespace.json"
```

Stop the kube proxy:

```bash
killall kubectl
```

Verify that the namespace is deleted:

```bash
kubectl get namespaces
```

## Resources

- [Kubernetes Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
