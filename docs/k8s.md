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

- Listing nodes

```sh
k get nodes
```

- Display resource usage (cpu/memory) for node

```sh
k top node <node name>
```

- Pods running on a node

```sh
k get pods -o wide | grep <node name>
```

- Get custom info about Nodes

```sh
k get nodes -o custom-columns="NAME:.metadata.name,INTERNAL_IP:.status.addresses[0].address,KERNEL:.status.nodeInfo.kernelVersion,MEMORY_PRESSURE:.status.conditions[0].status,DISK_PRESSURE:.status.conditions[1].status,PID_PRESSURE:.status.conditions[2].status,READY:.status.conditions[3].status"
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

- Get deployment history

```sh
k rollout history deployment <deployment_name>
```

```sh
k rollout history deployment <deployment_name> --revision=<revision_number>
```

- Compare two revisions

```sh
diff <(k rollout history deployment <deployment_name> --revision=<revision_number>) <(k rollout history deployment <deployment_name> --revision=<revision_number>)
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
k get pod <pod_name> -o wide
```

```sh
k describe pod <pod_name>
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
k get pod <pod_name> --output=jsonpath='{..podIP}'
```

```sh
k get pod <pod_name> --output=jsonpath='{..podIPs}'
```

#### Logs

```sh
k logs <pod_name>
```

```sh
k logs --since=1h <pod_name>
```

```sh
k logs --tail=50 <pod_name>
```

```sh
k logs -f <pod_name>
```

```sh
k logs <pod_name> <pod_name>.log
```

```sh
k logs --previous <pod_name>
```

```sh
k logs -c <container name> <pod_name>
```

Logs with label selector
(10 lines if a selector is provided)

```sh
k logs -l app.kubernetes.io/instance=<my_label> -n <namespace> 
```

#### Exec command

```sh
k exec <pod_name> -- ls /
```

```sh
k exec <pod_name> -c <container name> -- ls /
```

```sh
k exec --stdin --tty <pod_name> -- /bin/sh 
```

Get an interactive shell

```sh
k exec -it <pod_name> -- sh
```

#### Port Forward

```sh
k port-forward --address <local_ip_addr> <pod_name or deployment_name> <local_port>:<remote_port>
```

Example

```sh
k port-forward --address 0.0.0.0 pods/mongo-75f59d57f4-4nd6q 28015:27017
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

- Delete Pod

```sh
k delete pod <pod_name>
```

- Force Pod deletion

```sh
k delete pod --grace-period=0 --force <pod_name>
```

- List all Container images

```sh
k get pods -o jsonpath="{.items[*].spec['initContainers', 'containers'][*].image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
```

```sh
k get pods -o jsonpath='{range .items[*]}{"\n"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{", "}{end}{end}' | sort
```

- Get images IDs

```sh
 k get pods -o jsonpath="{.items[*].status.containerStatuses[*].imageID}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
```

#### Get restartCount and state

```sh
k get pods <pod_name> -o jsonpath='{.spec.containers[*].name} {.status.containerStatuses[*].restartCount} {.status.containerStatuses[*].state}'
```

#### Get Pods resources requests/limits

```sh
k get pods -o custom-columns='NAME:.metadata.name,CPU_REQUEST:spec.containers[].resources.requests.cpu,CPU_LIMIT:spec.containers[].resources.limits.cpu,MEMORY_REQUEST:spec.containers[].resources.requests.memory,MEM_LIMIT:spec.containers[].resources.limits.memory'
```

#### Get pod states

- Get Pods start time and ready time

```sh
k get pods -o custom-columns='NAME:.metadata.name,START_TIME:status.startTime,READY_TIME:.status.conditions[?(@.type=="Ready")].lastTransitionTime'
```

```sh
k get pods -o custom-columns='NAME:.metadata.name,START_TIME:status.startTime,READY:.status.conditions[?(@.type=="Ready")].status,READY_TIME:.status.conditions[?(@.type=="Ready")].lastTransitionTime' | (sed -u 1q; sort -k 3)
```

- Get `NAME`, `STARTED_AT` and `READY_AT` using `custom-columns`

```sh
k get pods -o custom-columns='NAME:.metadata.name,STARTED_AT:.status.containerStatuses[].state.running.startedAt,READY_AT:.status.conditions[?(@.type=="Ready")].lastTransitionTime'
```

- Get `Ready` time(Headers ignored and sorted by date)

```sh
k get pods -o custom-columns='POD_NAME:.metadata.name,READY_AT:.status.conditions[?(@.type=="Ready")].lastTransitionTime' | (sed -u 1q; sort -k 2)
```

Running state(using `jq`)

```sh
k get pod <pod_name> -o json | jq '.status.containerStatuses[].state'
```

Ready state(using `jq`)

```sh
k get pod <pod_name> -o json | jq '.status.conditions[] | select(.type=="Ready")'
```

### Events

- Get events with custom output

```sh
k get events --sort-by=.metadata.creationTimestamp -o custom-columns=LAST_SEEN:.lastTimestamp,TYPE:.type,REASON:.reason,OBJECT:.involvedObject.name,COMPONENT:.source.component,COUNT:.count,MESSAGE:.message
```

- Get events using `--field-selector`

```sh
k get events --field-selector involvedObject.kind=Pod
```

- List warnings events

```sh
k get events --sort-by=.metadata.creationTimestamp --field-selector type=Warning
```

- Sorting: Inverse order

```sh
watch -d 'kubectl get events --sort-by=.metadata.creationTimestamp --no-headers -A | tac'
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

## Helm

- Update repo

```sh
helm repo update
```

- List all charts

```sh
helm search repo
```

```sh
helm search repo <chart_name>
```

- List all versions of all charts

```sh
helm search repo -l
```

```sh
helm search repo -l <chart_name>
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

## Netbox

### Installation

- Install Netbox Helm Chart

```sh
helm repo add netbox https://charts.netbox.oss.netboxlabs.com/
```

```sh
helm install netbox netbox/netbox --version 5.0.9 --namespace netbox --create-namespace
```

## Documentaion

- [Cluster Architecture](https://kubernetes.io/docs/concepts/architecture/)
- [kubectl Quick Reference](https://kubernetes.io/docs/reference/kubectl/quick-reference/#bash)
- [Kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start)
