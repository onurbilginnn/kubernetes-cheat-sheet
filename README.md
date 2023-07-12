## Docker
- Container management tool

## Containerd
- Container runtime

# Main Components

## ETCD
- kubeadm deploys etcd as a pod (pod: etcd-master, namespace: kube-system)
- Distributed reliable key-value store
### ETCD Commands
- export ETCDCTL_API=3 (Set env variable etcdctl to use API version 3)
- etcdctl set key1 value1 (API2) - etcdctl put key1 value1 (API3)
- etcdctl get key1 (API2 & API3)
- etcdctl --version (API version is the important one, it is about the commands)

## kube-api Server
- The api server that faces the commands first and distribute them to the responsible component
- /etc/kubernetes/manifests/kube-apiserver.yaml (kube-apiserver creation file)
- /etc/systemd/system/kube-apiserver.service (kube-apiserver options file)
### kube-api Commands
- ps -aux | grep kube-apiserver (see running processes, master node and effective options)

## Kube-Controller-Manager
- kubeadm deploys kube-controller-manager as a pod (pod: kube-controller-manager-master, namespace: kube-system)
- ps -aux | grep kube-controller-manager (see running processes)
- Node-Controller 
- Replication-Controller
- ...
- /etc/systemd/system/kube-controller-manager.service;(kube-controller-manager options file; fileMonitor Period: 5s, Grace Period: 40s, POD Eviction Timeout: 5m...)

## Kube-Scheduler
- Decides which pod goes to which node, not places the pods to nodes
- /etc/kubernetes/manifests/kube-scheduler.yaml (kube-scheduler creation file)
- kubeadm deploys kube-scheduler as a pod (namespace: kube-system)
## Kube-Scheduler Commands
- ps -aux | grep kube-scheduler (see running processes)

## Kubelet
- Captain of the worker nodes
- Registers the worker node with the kubernetes  cluster
- Create and monitors pods, reports to api-server
- Manually install kubelet to worker nodes
## Kubelet Commands
- ps -aux | grep kubelet

## Kube-proxy
- A service can not join the node network, kube-proxy checks new services and every time a new service is created it is forwarding the traffic to the newly created service.
- kubeadm deploys kube-proxy as a daemonset (pod: kube-proxy-xxxx, namespace: kube-system)

# Components

## Pod
- Smallest object that can be created in kubernetes, encapsulates container
- To scale app, add or remove pods
- A pod could have multi containers as app + helper containers

## Pod Commands
- kubectl run <pod_name> --image <image_name> -n <namespace_name> (Runs a pod with provided image name in given namespace)
- kubectl run <pod_name> --image=<image_name> --dry-run=client -o yaml > <file_name>.yaml (simulates pod creation and writes pod config file to yaml file)
- kubectl run <pod_name> --image=<image_name> --port=<port_number> (Creates a pod exposed on specified port with the image provided)
- kubectl run <pod_name> --image=<image_name> --port=<port_number> --expose (Creates a pod and a service with the same name exposed on specified port with the image provided)
- kubectl get pods (Lists all pods in default namespace)
- kubectl get pods -o wide (Lists all pods with more details)
- kubectl get pods (get pods in default namespace)
- kubectl create -f <pod_definition_file>.yaml (create pod by file manual way)
- kubectl apply -f <pod_definition_file>.yaml (create pod by file automatic way)
- kubectl describe pod <pod_name> (detailed information about pod)
- kubectl delete pod <pod_name> (deletes pod)
- kubectl edit pod <pod_name> (edits current pod not changes the yaml file edits the configs on RAM)
- kubectl replace -f <pod_definition>.yaml --force

## ReplicaSets (Old way - Replication Controller)
- Helps creating multi pod replicas for high availability (Scaling)
- Can span across multi nodes
- selector/matchlabels and template/labels should match
## ReplicaSets Commands
- kubectl create -f <replicaset-definition>.yaml
- kubectl get replicaset
- kubectl scale --replicas=<new_replica_amount> -f <replicaset-definition>.yaml
- kubectl scale --replicas=<new_replica_amount> replicaset <replicaset_name>
- kubectl delete replicaset <replicaset_name> (Also deletes underlying pods)
- kubectl replace -f <replicaset_definition>.yaml --force
- kubectl describe rs <replicaset_name> (detailed information about rs)

## Deployments
- Deployment is one level up on replicaset in the hierarchy
- Provides seamless upgrades using rolling updates, pause-resume changes
## Deployments Commands
***** Same as replicasets, just write deployment instead of replicaset in the command *****
- kubectl create deployment --image=nginx nginx (Creates deployment)
- kubectl create deployment --image=nginx nginx --dry-run=client -o yaml (Generate Deployment YAML file (-o yaml). Don't create it(--dry-run))
- kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml (Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.)

## Services
- Enable connectivity between apps with other apps or users
- Listens a port and forwards the request to other port
- Uses random algorithm to balance the load between pods
- Service is created span across all nodes with same NodePort
### Types
- NodePort service; listens a port on a node and forwards the request to pods
- ClusterIP service; creates a virtual IP to enable communication between different services in the cluster (Ex; app -> db)
- LoadBalancer service; a service for load balancers in cloud providers
### Port Types
- Port (Service port)
- TargetPort (Forwarded port)
- NodePort (External port)
## Services Commands
***** Same commands as get, describe, delete, create -f... ***** 
- curl <ip:port>

## Namespaces
- Resource house
- Have policies, quotas
- in namespace = mysql.connect("db-service")
- in other namespace = mysql.connect("db-service.dev.svc.cluster.local")
- Automatically created DNS convention = service_name.namespace.resource_type.domain
## Namespace Commands
- kubectl get pods --namespace=<namespace_name> (Get pods in specified namespace)
- kubectl create -f <pod_definition>.yaml --namespace=<namespace_name> (Creates pod in specified namespace)
- kubectl create namespace <namespace_name> (Create namespace)
- kubectl config set-context $(kubectl config current-context) --namespace=dev (Changes default namespace to dev)

## Imperative Commands
### Create Commands
- kubectl run --image=nginx
- kubectl create deploy --image=nginx nginx
- kubectl expose <resource_type> nginx --port 80 (Exposes a resource(pod, deployment) by creating a service for it
 by using it is labels as selectors on port 80)
### Update Commands
- kubectl edit deploy nginx
- kubectl scale deploy nginx --replicas=5
- kubectl set image deploy nginx nginx=nginx:1.18
### CRD Commands
- kubectl create -f nginx.yaml
- kubectl replace -f nginx.yaml
- kubectl replace -f nginx.yaml --force
- kubectl delete -f nginx.yaml
## Declarative Commands
- kubectl apply -f nginx.yaml
- kubectl apply;
1 - Creates the object if it does not exist, the yaml file that was used in creation 
stored as json as last applied configuration
2 - If the object exists, kubectl apply command compares the file with last applied configuration json and
updates it, if something deleted from the new yaml, you can revert the changes.
3 - Last applied configuration object stored under metadata - annotations - kubectl.kubernetes.io/last-applied-configuration
## Scheduling
### Taints and Tolerations
- Scheduler limitations when placing pods in nodes, if the pod has toleration to the taint 
defined on the node the pod could be placed in the node.
- Taints => nodes, tolerations => pods
- Taints and tolerations do not tell the pod to go to a particular node, taints tell the nodes to accept pods with
certain tolerations.
- If you want complex & detailed logic, it is being achieved by node affinity.
### Taints and Tolerations Commands
- kubectl taint nodes <node_name> <key>=<value:taint-effect> (Adds taint to node, 
taint-effect types = NoSchedule, PreferNoSchedule, NoExecute)
Ex: kubectl taint nodes node01 app=blue:NoSchedule
- kubectl taint nodes <node_name> <key>=<value:taint-effect>- (Removes taint from the node)
- kubectl describe node <node_name> | grep Taint
### Node Selectors & Node Affinity
- Node selectors selects nodes by label
- For complex node selections use node affinity
- A combination of taints/tolerations and node affinity will make node selection most accurate
### Node Selectors & Node Affinity Commands
- kubectl label nodes <node_name> <key>=<value>
Ex: kubectl label nodes node01 size=large
### Resource Requirements & Limits
- By default node has no requests & limits on cpu or memory
- if no requests defined but limits are defined, Kubernetes automatically define requests as limit thresholds.
- if no limits defined but requests are defined, each pod guaranteed to have requested resources
and unlimited resources if needed. (Best scenario)
- Request set is really important that makes you sure you will have sufficient resources
- If you define a limit in pod definition file on container prop, scheduler will throttle the cpu and 
pod can not exceed cpu limit, if memory limit exceeds scheduler will terminate the pod with OOM(out of memory) alert.
- You can create LimitRange object in namespace level that will set default limits and requests for all
pod under same namespace.
- You can create ResourceQuota object to limit all total pods not to exceed defined resource quota in the namespace.
### Daemon Sets
- Makes sure that one replica of the pod runs in all the nodes in the cluster.
- For Ex; kube-proxy, weave-net etc
### Daemon Sets Commands
- kubectl get daemonsets
- kubectl describe daemonsets <daemon_name>
### Static Pods
- If there is only worker node with kubelet and NO kube-apiserver, etcd, controller manager, kube-scheduler
- kubelet managed pod independently is static pod.
- /etc/kubernetes/manifests folder could include pod.yaml files and kubelet will use these files to create 
static pods.
- The folder can be configured under kubelet.service config file as --pod-manifest-path=/etc/kubernetes/manifests
- You can provide another config file under kubelet.service file as --config=<file_name>.yaml
(In the file use prop; staticPodPath: /etc/kubernetes/manifests)
- /var/lib/kubelet/config.yaml is the default location for the kubelet config file.
- You can see the static pods by running 'kubectl get pods'
- You can not edit static pods, you can delete or modify them by using manifests folder files only
- kubescheduler ignores static pods.
### Static Pods Commands
- docker ps
- systemctl cat kubelet.service (see kubelet.service file)
- kubectl get pod <pod_name> -o yaml (Get detailed info for the pod as yaml and if you check ownerReferences: you can
find if the pod is static or not)
### Multiple Schedulers
- Custom scheduler runs as pod or deployment.
- kube-scheduler.service - ExecStart=/usr/local/bin/kube-scheduler (--config=/etc/kubernetes/config/kube-scheduler.yaml)
- Check documentation [scheduler docs](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)
## General Commands and tags
- kubectl <command_text> -o <output_options(wide, yaml, json...)>(runs command and outputs the result in given output options (wide: detailed output on terminal, others are file outputs))
- kubectl <command_text> --help (lists all the options for the given command)
- kubectl <command_text> -n=<namespace_name> (runs command for given namespace)
- kubectl <command_text> -A (runs command on all namespaces)
- kubectl <command_text> --dry-run=client (simulates command result not runs the command exactly)
- kubectl replace -f <file_name>.yaml --force (replaces(delete & create) the current component by given file)
- kubectl api-resources (List all commands with short keys)
- kubectl explain <component_type> (explains the definition file for given component type)
- kubectl get all (Lists all components pods, replicasets, deployments ...)
- k get pods | wc -l (Counts lines on the output)
- kubectl get <resource_name> --selector env=dev,bu=finance --no-headers (selects resources with env=dev and bu=finance label)
