# Tekton
## Links
- https://tekton.dev/
- https://github.com/joellord/handson-tekton


## Installation
```bash
# Create kind cluster
kind create cluster --name tekton-cluster
# Install Tekton in cluster
kubectl apply -f ./tekton/release.yaml
# Validate you got pods running
kubectl get pods --namespace tekton-pipelines
# Configure persistent volume for Tekton requests
kubectl create configmap config-artifact-pvc \
                         --from-literal=size=10Gi \
                         --from-literal=storageClassName=manual \
                         -o yaml -n tekton-pipelines \
                         --dry-run=client | kubectl replace -f -

# Install the cli (for Ubuntu and deb distros)
sudo apt update && sudo apt install -y gnupg
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3EFE0E0A2F2F60AA
echo "deb http://ppa.launchpad.net/tektoncd/cli/ubuntu eoan main"|sudo tee /etc/apt/sources.list.d/tektoncd-ubuntu-cli.list
sudo apt update && sudo apt install -y tektoncd-cli

```

## First task
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello
spec:
  steps:
    - name: say-hello
      image: alpine
      command:
        - /bin/sh
      args: ['-c', 'echo Hello World']
```

You can register the task to the cluster using a kubsctl command.
```bash
kubectl apply -f task/hello.yaml
tkn task start hello --dry-run > taskRun/hello.yaml
```

And update it accordingly:
```yaml
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  creationTimestamp: null
  generateName: hello-run-
  namespace: default
spec:
  taskRef:
    name: hello
  timeout: 0h1m0s
```

You can now run it using the kubectl command or the tkn task start command.

```bash
kubectl create -f taskRun/hello.yaml
# Visualize logs with tkn cli
tkn task logs hello
# Visualize your logs using kubctl
kubectl logs hello-run-xxxxxxx
```


## Second task with params
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello
spec:
  params:
    - name: person
      description: Name of person to greet
      default: World
      type: string
  steps:
    - name: say-hello
      image: alpine
      command:
        - /bin/sh
      args: ['-c', 'echo Hello $(params.person)']
```

You can register the task to the cluster using a kubsctl command.
```bash
kubectl apply -f task/hello-with-params.yaml
tkn task start hello-with-params --dry-run > taskRun/hello-with-params.yaml
```

And update it accordingly:
```yaml
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  creationTimestamp: null
  generateName: hello-with-params-run-
  namespace: default
spec:
  taskRef:
    name: hello-with-params
  timeout: 0h1m0s
```

You can now run it using the kubectl command or the tkn task start command.

```bash
kubectl create -f taskRun/hello-with-params.yaml # or tkn task start --showlog  hello-with-params
# Visualize logs with tkn cli
tkn task logs hello-with-params
# Visualize your logs using kubctl
kubectl logs hello-with-params-run-xxxxxxx

# Not working
tkn task start --showlog  hello-with-params -p person=Bob
```

## Dashboard
```bash
kubectl apply -f tekton/dashboard.yaml
kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 9097:9097
```
