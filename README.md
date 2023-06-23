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
***** Same as replicasets, just write deployment instead of replicasete in the command *****

## General Commands and tags
- kubectl <command_text> --help (lists all the options for the given command)
- kubectl <command_text> -n=<namespace_name> (runs command for given namespace)
- kubectl <command_text> -A (runs command on all namespaces)
- kubectl <command_text> -o <output_options(wide, yaml, json...)>(runs command and outputs the result in given output options (wide: detailed output on terminal, others are file outputs))
- kubectl <command_text> --dry-run=client (simulates command result not runs the command exactly)
- kubectl run <pod_name> --image=<image_name> --dry-run=client -o yaml > <file_name>.yaml (simulates pod creation and writes pod config file to yaml file)
- kubectl replace -f <file_name>.yaml --force (replaces(delete & create) the current component by given file)
- kubectl api-resources (List all commands with short keys)
- kubectl explain <component_type> (explains the definition file for given component type)
- kubectl get all (Lists all components pods, replicasets, deployments ...)