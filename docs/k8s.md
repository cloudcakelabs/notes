# Kubernetes

## Administration

### Setup `~/.bashrc`

Download: Kubernetes prompt for bash

```sh
curl -L https://raw.githubusercontent.com/jonmosco/kube-ps1/refs/heads/master/kube-ps1.sh -o ~/.bash_kube_ps1
```

```sh
source <(kubectl completion bash)
alias k=kubectl
complete -o default -F __start_kubectl k
source ~/.bash_kube_ps1
PS1='[\u@\h \W $(kube_ps1)]\$ '
```

### Cluster and context

#### Display addresses of the control plane and services

```sh
k cluster-info
```

#### Print the client and server version information

```sh
k version
```

#### Get the configuration of the cluster

```sh
k config view
```

#### Display the current context

```sh
k config current-context
```

#### Display the list of contexts

```sh
k config get-contexts
```

#### Set the default context

```sh
k config use-context <context name>
```

#### Merging kubeconfig files

```sh
KUBECONFIG=file1:file2:file3 k config view --merge --flatten > config.merged
```

#### List the API resources

```sh
k api-resources
```

#### List pods, services, daemonsets, deployments, etc...in all namespaces

```sh
k get all -A
```

### Nodes

#### Listing nodes

```sh
k get nodes
```

#### Display resource usage (cpu/memory) for node

```sh
k top node <node name>
```

#### Pods running on a node

```sh
k get pods -o wide | grep <node name>
```

### Namespaces

#### Listing namespaces

```sh
k get namespaces
```

```sh
k get namespace <namespace>
```

#### Display details about namespace

```sh
k describe namespace <namespace>
```

### Deployments

- List deployments

```sh
k get deployment
```

- Get details about a deployment

```sh
k describe deployment <deployment_name>
```

- Scale up/down a deployment

```sh
k scale --replicas=<number> deployment/<deployment_name>
```

### Daemonsets

- List daemonsets

```sh
k get daemonset
```

- Display detailed state of daemonset

```sh
k describe ds <daemonset_name>
```

### StatefulSet

- List StatefulSet

```sh
k get statefulset
```

- Scale Up/Down

```sh
k scale --replicas=<number> sts <sts_name>
```

### Pods

#### List pods

```sh
k get pods
```

```sh
k get pods -o wide
```

```sh
k get pods --show-labels
```

#### Get information about a Pod

```sh
k get pod <pod name> -o wide
```

```sh
k describe pod <pod name>
```

Sort pods list using specified field. The field can be either 'cpu' or 'memory'

```sh
 k top pod --sort-by=memory
```

```sh
 k top pod --sort-by=cpu
```

#### Get IP addr from Pod definition

```sh
k get pod <pod name> --output=jsonpath='{..podIP}'
```

```sh
k get pod <pod name> --output=jsonpath='{..podIPs}'
```

#### Logs

```sh
k logs <pod name>
```

```sh
k logs --since=1h <pod name>
```

```sh
k logs --tail=50 <pod name>
```

```sh
k logs -f <pod name>
```

```sh
k logs <pod name> <pod name>.log
```

```sh
k logs --previous <pod name>
```

```sh
k logs -c <container name> <pod name>
```

#### Exec command

```sh
k exec <pod name> -- ls /
```

```sh
k exec <pod name> -c <container name> -- ls /
```

```sh
k exec --stdin --tty <pod name> -- /bin/sh 
```

#### ReplicaSet

```sh
k get rs
```

- Filter: `DESIRED` != 0

```sh
k get rs | awk '{if ($2 != 0) print $0}'
```

```sh
k describe rs/<rs name>
```

#### Delete Pod

```sh
k delete pod <pod_name>
```

#### Force Pod deletion

```sh
k delete pod --grace-period=0 --force <pod_name>
```

#### List all Container images

```sh
k get pods -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' | sort
```

```sh
k get pods -o jsonpath="{.items[*].spec['initContainers', 'containers'][*].image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
```

- Get image ID

```sh
 k get pods -o jsonpath="{.items[*].status.containerStatuses[*].imageID}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
```

#### Get restartCount and state

```sh
k get pods <pod_name> -o jsonpath='{.spec.containers[*].name} {.status.containerStatuses[*].restartCount} {.status.containerStatuses[*].state}'
```

#### Get Pods resources requests/limits

```sh
k get pods -o custom-columns='NAME:.metadata.name,CPU_REQ:spec.containers[].resources.requests.cpu,CPU_LIM:spec.containers[].resources.limits.cpu,MEMORY_REQ:spec.containers[].resources.requests.memory,MEM_LIM:spec.containers[].resources.limits.memory'
```

#### Get pod states

Get `NAME`, `STARTED_AT` and `READY_AT` using `custom-columns`

```sh
k get pods -o custom-columns='NAME:.metadata.name,STARTED_AT:.status.containerStatuses[].state.running.startedAt,READY_AT:.status.conditions[?(@.type=="Ready")].lastTransitionTime'
```

Running state(using `jq`)

```sh
k get pod <pod_name> -o json | jq '.status.containerStatuses[].state'
```

Ready state(using `jq`)

```sh
k get pod <pod_name> -o json | jq '.status.conditions[] | select(.type=="Ready")'
```


## Maintenance

### Cordon the node(marked as unschedulable)

```sh
k cordon <node-name>
```

### Drain the workloads for the node

```sh
k drain <node-name> --ignore-daemonsets 
```

### Uncordon the node(marked as schedulable)

```sh
k uncordon <node-name>
```

## Rancher, RKE2 and K3S

### crictl

```sh
export CRI_CONFIG_FILE=/var/lib/rancher/rke2/agent/etc/crictl.yaml
```

```sh
/var/lib/rancher/rke2/bin/crictl ps
```

### kubectl

```sh
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
```

```sh
/var/lib/rancher/rke2/bin/kubectl get nodes
```

### Systemd services

RKE2 server

```sh
systemctl status rke2-server
```

RKE2 agent

```sh
systemctl status rke2-agent
```

#### Uninstall

```sh
/usr/local/bin/rancher-system-agent-uninstall.sh
```

### containerd

socket: `/run/k3s/containerd/containerd.sock`

### Install Rancher using Docker

[Installing Rancher on a Single Node with default Rancher-generated Self-signed Certificate](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/other-installation-methods/rancher-on-a-single-node-with-docker)

```sh
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher:v2.9.2
```

#### K3S configuration

- [Requirements](https://docs.k3s.io/installation/requirements)
- [Known Issues](https://docs.k3s.io/known-issues)

## Kind

### Quick Start([doc](https://kind.sigs.k8s.io/docs/user/quick-start/))

- Install `kubectl`[(doc)](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- Install `go`[(doc)](https://go.dev/doc/install)
- Install `kind`

```sh
go install sigs.k8s.io/kind@v0.25.0 
```

- Create a cluster

```sh
kind create cluster --config <config_file> --name <cluster_name>
```

- Delete a cluster

```sh
kind delete cluster --name <cluster_name>
```

### Known Issues

- Pod errors due to "too many open files"[(doc)](https://kind.sigs.k8s.io/docs/user/known-issues/#pod-errors-due-to-too-many-open-files)

## Documentaion

- [Cluster Architecture](https://kubernetes.io/docs/concepts/architecture/)
- [kubectl Quick Reference](https://kubernetes.io/docs/reference/kubectl/quick-reference/#bash)
- [Kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start)
