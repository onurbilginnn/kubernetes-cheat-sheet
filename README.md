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
### Multiple Schedulers Commands
- kubectl get events -o wide (Shows all events by schedulers)
- kubectl logs my-custom-scheduler -n=kube-system (Shows logs for the my-custom-scheduler pod or deployment)
### Scheduler Profiles
- Scheduler queues pods as below order
    1 - Scheduling queue => Plugins - PrioritySort => Extensions - QueueSort
    (Bind to priority classname that is defined by PriorityClass definition file)
    2 - Filtering => Plugins - NodeResourcesFit, NodeName, NodeUnschedulable, TaintToleration, NodePorts, NodeAffinity => Extensions - preFilter, filter, postFilter
    3 - Scoring => Plugins - NodeResourcesFit, ImageLocality, TaintToleration, NodeAffinity => Extensions - preScore, score, reserve, permit
    4 - Binding => Plugins - DefaultBinder => Extensions - preBind, bind, postBind
- On custom scheduler run condition, you can configure multi schedulers in one scheduler profile

## Logging and Monitoring
### Monitoring
- Metrics Server for monitoring (Only in memory)
- Kubelet has Cadvisor and that sends logs to metrics server
## Logging and Monitoring Commands
### Monitoring
- minikube addons enable metrics-server
  git clone https://github.com/kubernetes-incubator/metrics-server.git
  kubectl create -f deploy/1.8+
  kubectl top node (Shows resource comsumption by node)
  kubectl top pod (Shows resource comsumption by pod)
### Logging
- kubectl logs -f <pod_name> (Single containered pod will show the logs, multi containered pod will fail)
- kubectl logs -f <pod_name> <container_name> (Shows logs for the specified pod and container)
- kubectl logs <pod_name> -c <container_name> (List logs for the container in a specific pod)

## App Lifecycle Management
### Rolling Updates and Rollbacks
- There 2 types of deployment strategy; 
Recreate (Destroys all pods at once and creates all at once - cons: App is down during recreation process)
Rolling update (Kubernetes Default) (Replaces all pods one by one - cons: Update is seamless)
### Rolling Updates and Rollbacks Commands
- kubectl rollout status <deployment/deployment_name> (See the status of the rollout)
- kubectl rollout history <deployment/deployment_name> (See the history of the rollout)
- kubectl rollout undo <deployment/deployment_name> (Rollback the deployment upgrade)
- kubectl set image <deployment/deployment_name> <image_name>=<image> (Updates as defined recreate or rolling)
### Application Commands
- Initial commands on containers to run when the container first runs
- Kubernetes definition.yaml file always overwrites Docker entrypoint commands in the Dockerfile
- kubectl run <pod_name> --image=<image_name> --command -- <command1> <command2> -- <arg1> <arg2>
(Create pod with command and args)
- kubectl run <pod_name> --image=<image_name> -- <arg1> <arg2> (Create pod with args)
### ConfigMaps
- File to add environment variables to pods
### ConfigMaps Commands
- kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MOD=prod
- kubectl create configmap app-config --from-file=<file_name>.properties
- kubectl get configmaps
### Secrets
- Secret env vars
- Secrets are not encrypted. Only encoded
- Secrets are not ecrypted in ETCD
- Anyone able to create pods/deployments in the same namespace can access the secrets
- After encryption at rest enabled existing secrets won't be encrypted only after creations will be encrypted
### Secrets Commands
- kubectl create secret generic <secret_name> --from-literal=<key>=<value>
- kubectl create secret generic <secret_name> --from-file=<file_name>.properties
- kubectl get secrets
- echo -n <text> | base64 (Convert text to base64 encoded string)
- echo -n <text> | base64 --decode (Decode base64 to string)
### Multi-container Pods
- 3 patterns; sidecar, adapter, ambassador
### Initcontainers
- A task that will be run only  one time when the pod is first created that is implemented by using initContainer.
- When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts. 
- You can configure multiple such initContainers as well, like how we did for multi-containers pod. In that case each init container is run one at a time in sequential order.

## Cluster Maintenance
### OS Updates
- kube-controller-manager --pod-eviction-timeout=5m0s (Node unresponsive time after time passes node considered 
as dead and all pods in the node will be moved to other nodes)
- If the node comes back after --pod-eviction-timeout it will be empty
### OS Updates Commands
- kubectl drain <node_name> (The pods in the node terminated and started in an other node, node becomes unschedulable,
if node comes online it is still unschedulable)
- kubectl drain <node_name> --ignore-daemonsets (Ignores daemonset management to evict pods in the node)
- kubectl uncordon <node_name> (Make unschedulable node schedulable)
- kubectl cordon <node_name> (Make schedulable node unschedulable, no terminates)
### Cluster Upgrade
- Kubernetes components can have different versions.
- kube-apiserver version = 1.10 (X)
  controllor manager & kube-scheduler versions = 1.09 || 1.10 (X-1 to X)
  kubelet & kube-proxy versions = 1.08 || 1.09 || 1.10 (X-2 to X)
  kubectl 1.11 || 1.10 || 1.09 (X-1 to X+1)
- Kubernetes supports only 3 versions at a time as download (For ex; 1.10 || 1.11 || 1.12)
- First update master node then worker nodes
- Worker node upgrade strategies;
  1 - All at a time (Has downtime)
  2 - One by one
  3 - By adding new nodes to the cluster one by one
- Upgrade kubeadm & kubelet on nodes one by one after draining the node (First master node)
### Cluster Upgrade Commands
- GCP updates the cluster with one click
- kubeadm upgrade plan (Shows versions and details about the upgrade )
  kubeadm upgrade apply (Upgrades the cluster)
- apt-get upgrade -y kubeadm=1.12.0-00(Upgrades kubeadm tool)
- apt-get upgrade -y kubelet=1.12.0-00 (Upgrades kubelet)
- kubeadm upgrade node config --kubelet-version v1.12.0
- systemctl restart kubelet (Restarts kubelet to perform upgrades)
- systemctl daemon-reload (Restarts daemon)
### Backup & Restore
- etcd.service --data-dir=/var/lib/etcd (Shows the etcd resources save location)
### Backup & Restore Commands
- kubectl get all -A -o yaml > kubernetes-api-resources.yaml (Saves all resources to the file)
#### Restoring ETCD
- In manifests/etcd.yaml check volume and path file locations
#### Restoring ETCD Commands
- vi /etc/systemd/system/etcd.service (etcd service edit)
- run 'etcdctl' commands on controlplane
- ETCDCTL_API=3 etcdctl snapshot save snapshot.db
- ETCDCTL_API=3 etcdctl snapshot status snapshot.db
- service kube-apiserver stop
- ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
--data-dir /var/lib/<file_name>
- systemctl daemon-reload
- service etcd restart
- service kube-apiserver start
**** For all etcd commands add certificates to the commands****
   --endpoints=https://127.0.0.1:2379
   --cacert=/etc/etcd/ca.crt
   --cert=/etc/etcd/etcd-server.crt
   --key=/etc/etcd/etcd-server.key  

## Security
### Primitives
- kube-apiserver is the center of any operation, (first line of defence)
- Who can access kube-apiserver and what can they do?
- By default all pods can access all other pods within the cluster, we can use network policies to configure it.
### Auhentication
- Two types of accounts; User(Admin, developer) - Service Accounts(Bots)
- All user requests managed by kube-apiserver
- kube-apiserver autenticates the user then process the requests
#### kube-apiserver authentication ways
- Static password file
kube-apiserver.service section => --basic-auth-file=<password_file>.csv
- Static token file
kube-apiserver.service section => --token-auth-file=<token_file>.csv
- Certificates
- Identity Services
#### Auth  User
- curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password123"
- curl -v -k https://master-node-ip:6443/api/v1/pods -header "Authorization: Bearer <token>"
### TLS Certs
- 2 encryption types symmetric(one key) & asymmetric(private key & public key(lock))
- Local key location ~/.ssh/authorized_keys
- Public Keys => server.crt, server.pem, client.crt, client.pem
- Private Keys => server.key,  server-key.pem, client.key, client-key.pem
- If you encrypt your data with one of the keys only the other key can decrypt your data, you can not decrypt your data with the same key.
- Encrypt your data with public key only, if you encrypt with private key anyone with the public key can decrypt you data.
- Three types of certs 
  1 - Root Certs (Authority Certs)
  2 - Server Certs
  3 - Client Certs
### TLS Certs Commands
- ssh-keygen (Creates private & public keys)
- ssh -i id_rsa user1@server1 (Login success with key)
- ssh-rsa <key> <user> (For ex. ssh-rsa AAAB32SDFAwer234adfavxcgasg67DSGSDvbvbdg user1)
- openssl genrsa -out <private_key_file_name>.key 1024 (Creates private key)
- openssl rsa -in <private_key_file_name>.key -pubout > <public_key_file_name>.pem (Creates public key)
- openssl req -new -key my-bank.key -out my-bank.csr -subj "/C=US/ST=CA/O=MyOrg, Inc./CN=my-bank.com" (Creates a certificate request sign)
### Kubernetes TLS Certs 
- kube-apiserver, etcd-server, kubelet-server, kube-scheduler, admin user, kube-controller-manager, kube-proxy have keys(public, private) because all of them are communicating
- kube-scheduler, admin user, kube-controller-manager, kube-proxy, kubelet-server send requests to kube-apiserver
- kube-apiserver sends requests to kubelet-server & etcd-server
- Servers that send requests have CLIENT CERTS, servers that get requests have SERVER CERTS

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
- kubectl create -f . (Creates all files within the folder)
- kubectl exec -n <namespace_name> -it <pod_name> -- cat <file_name>  (Checks the file in the pod in given namespace, exec command runs commands in given pod)
- alias k=kubectl (shortcut k defined in terminal to call kubectl)
- cat /etc/*release* (Find current OS)
- export ETCDCTL_API=3 (Configures ETCDCTL_API to 3)
- kubectl config <arg> (kubernetes config command to see, update, add contexts, clusters...)
- ps -ef | grep etcd (Check service configurations)

