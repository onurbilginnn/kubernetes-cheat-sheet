## Kubernetes Cheat Sheet Commands
- alias k=kubectl
- complete -o default -F __start_kubectl k
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
- kubectl set image deployment/<deployment_name> <image_name>=<image> (Updates as defined recreate or rolling)
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
- service kube-apiserver start <br>
**** For all etcd commands add certificates to the commands****
   --endpoints=https://127.0.0.1:2379 \ <br>
   --cacert=/etc/etcd/ca.crt \  <br>
   --cert=/etc/etcd/etcd-server.crt \ <br>
   --key=/etc/etcd/etcd-server.key   <br>

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
- kube-config.yaml have the certificates and key files location, so you don't need to add these on every request
- All cert related processes are managed by controller manager (CSR approving, CSR signing)
--cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt, --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
### Kubernetes TLS Certs Commands
- openssl genrsa -out admin.key 2048 (key generation)
- openssl rsa -in <private_key_file_name>.key -pubout > <public_key_file_name>.pem (Creates public key)
- openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr (Cert signing request)
- openssl x509 -req -in admin.csr -CA car.crt -CAkey ca.key -out admin.crt (Sign certs)
- curl https://kube-apiserver:6443/api/v1/pods --key admin.key --cert admin.crt --cacert ca.crt (send request with certs and key)
- openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout (Check the cert)
- kubectl get csr (Get Certificate Signing Reqs)
- kubectl certificate approve <csr_name> (Approves csr)
- kubectl certificate deny <csr_name> (Rejects csr)
#### ETCD Certs
- peer public and private key certs need to be created in order to be able to use etcd-server in multi clusters as high availability
- etcd.yaml; <br>
  --key-file=/path-certs/etcdserver.key <br>
  --cert-file=/path-certs/etcdserver.crt <br>
  --peer-cert-file=/path-certs/etcdpeer1.crt <br>
  --peer-client-cert-auth=true <br>
  --peer-key-file=/etc/kubernetes/pki/etcd/peer.key <br>
  --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt <br>
  --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt <br>
#### kube-apiserver Certs
--etcd-cafile=/var/lib/kubernetes/ca.pem <br>
--etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt <br>
--etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key <br>
--client-ca-file=/var/lib/kubernetes/ca.pem <br>
--tls-cert-file=/var/lib/kubernetes/apiserver.crt <br>
--tls-private-key-file=/var/lib/kubernetes/apiserver.key <br>
#### kube-apiserver Certs
- system:node:<node_name> group
### KubeConfig
- Instead of writing certs in all commands, you specify these files in kubeconfig
- $HOME/.kube/config default location for kube config file
- Clusters = Development, Production, Google
  Users = Admin, Dev, Prod
  Contexts = Admin@Production, Dev@Google
### KubeConfig Commands
- kubectl config view
- kubectl config view --kubeconfig=my-custom-kubeconfig (Run commands on custom konfig file)
- kubectl config use-context prod-user@production (Changes default context)
- kubectl get nodes --kubeconfig=<custom_kubeconfig_file> (Run commands by using custom kubeconfig file)
### API Groups
- /metrics - /healthz - /version - /api - /apis - /logs
- /api -> Core group
  /apis -> Named group
- Check Kubernetes api reference page
### API Groups Commands
- curl https://kube-master:6443/version (API version check)
- curl https://kube-master:6443/api/v1/pods (get pods)
- curl https://localhost:6443 -k (get all api paths)
- curl https://localhost:6443/apis (get all apis)
- kubectl proxy (Launches a proxy service on localhost:8001 you can check kubernetes apis by curl)
### Authorization
- Limitations for the authenticators
- Authorization types; Node, ABAC (Attribute based), RBAC (Role based), Webhook (3rd Party)
- kube-apiserver.service file --authorization-mode=AlwaysAllow
                              --authorization-mode=AlwaysDeny
                              --authorization-mode=Node,RBAC,Webhook
#### Authorization RBAC
- Create role yaml
- Create a roleBinding to link the user to the role
#### Authorization RBAC Commands
- kubectl auth can-i create deployments (Existing user permission check)
- kubectl auth can-i create deployments --as dev-user --namespace ns1 (dev-user permission check in ns1 namespace)
  kubectl get pods as <user_name> (Check permissions for specific user)
### Cluster roles and Role Bindings
- Namespace scoped resources = pods, relicasets, jobs, deployments, services, secrets, roles, rolebindings, configmaps, pvc
- Cluster scoped resources = nodes, PV, clusterroles, clusterrolebindings, certificatesigningrequests, namespaces
- Cluster scoped resources roles and role bindings = cluster roles, cluster role bindings
### Authorization Commands
- kubectl create role | rolebinding | clusterrole | clusterrolebinding
### Service Accounts
- Are used by machines
- When a sevice acc is created, system automatically creates a token, and store it as a secret, then links it to the service account (JWT token)
  You can see the secret by using 'kubectl describe secret <secret_name>'
  There is no expiry date on that token but after v1.22 tokens have expiry dates
- After v1.24 after creating a new service account token is not generatede automatically
  Run 'kubectl create token <serviceacc_name>' to generate a token with an expiry date
- Kubernetes automatically deploy default service account
- When a pod is created the secret token on the service account will be mounted to the new pod
### Service Accounts Commands
- kubectl create serviceaccount <serviceacc_name>
- kubectl exec -it <pod_name> -- ls /var/run/secrets/kubernetes.io/serviceaccount (List all secrets in service account)
- kubectl describe secret <secret_name>
- kubectl create token <serviceacc_name>
### Image Security
- Image fetch structure => <Registry>/<user_acc>/<image_repo>
                           docker.io/library/nginx
### Image Security Commands
- docker login private-registry.io (Login to private registry)
- docker run private-registry.io/apps/internal-app (Run image from private registry)
- kubectl create secret docker-registry regcred \
   --docker-server=private-registry.io         \
   --docker-username=registry-user       \
   --docker-password=registry-password       \
   --docker-email=registry-user@org          \
### Docker Security
- Manage security settings on container level
- Docker uses Linux capabilities to create security
### Docker Security Commands
- docker run --user=<user_name> <image_name> sleep 3600 (Runs image with specified user)
- docker run --cap-add MAC_ADMIN <image_name> (Add MAC_ADMIN permission to image)
  docker run --cap-drop KILL <image_name> (Drop KILL permission from image)
  docker run --privileged <image_name> (Add full privileged permission to image)
### Security Context
- Manage security settings on pod level
### Network Policy
- Ingress = As a web server the traffic coming from the users is ingress traffic
  Egress =  As a web server the traffic sent to the app server is agress traffic
- By default AllowAll is set in Kubernetes that allows any pod to communicate with any pod
- You can configure ingress and egress traffic by network policy
- 3rd parties that support network policies; Kube-router, Calico, Romana, Weave-net
  3rd party that NOT support network policies; Flannel
## General Commands and tags
## Storage
### Docker Storage
- Storage drivers - AUFS | ZFS | BTRFS | DEVICE MAPPER | OVERLAY
- File system -> /var/lib/docker/aufs | containers | image | volumes
- Docker runs in layered architecture, each layer only stores the changes from the previous layer, improves build performance
For example, when you update app code docker updates it very fast
- Layers;
  1 - Image Layer (When image build completed, this layer accepts no change - Read only)
  2 - Container Layer (Logs, temp files - Read Write)
- COPY-ON-WRITE, when you modify source code in image layer, Docker creates a copy of the new source code in container layer, and you can keep on modifying this copied file in container layer.
- Volumes, you can store COPY-ON-WRITE changes in container layer in volumes. Even if container is destroyed the data is persistent as volume.
- Volume mounting = Mounts the volume, bind mounting = Mounts the directory to the container
### Docker Storage 
- docker volume create <volume_name>
- docker run -v /<path>/<volume_name>:/var/lib/mysql <container_name> (Creates volume and mounts on container)
  docker run --mount type=bind,source=/<path>/<volume_name>,target=/var/lib/<container_name> <container_name> (New style for the above command)
- docker run -it --name mysql --volume-driver rexray/ebs --mount src=ebs-vol,target=/var/lib/mysql mysql (Creates an ebs volume and mounts to the container)
### Container Storage Interface(CSI)
- Universal container orchestration tool interface
- CreateVolume, DeleteVolume, ControllerPublishVolume
### Volumes
- Volume drivers - Local | Azure File Storage | Convoy | AWS EBS ...
- If a pod deleted the data that is processed by that pod gets deleted too, so if we mount a volume to the pod the processed data will be safe in the volume even if the pod gets deleted.
### Persistent Volumes
- If you have a large environment, it is difficult to configure volumes in all pod definition files, every time updating, creating etc is really difficult, so we can use persistent volumes.
- Persistent Volume, cluster-wide pool of storage volumes configured by an administrator to be used by users deploying apps on the cluster. 
### Persistent Volume Claims(PVC)
- As administrator creates persistent volumes, user creates persistent volume claims.
- Once a PVC is created Kubernetes binds it with the persistent volume
- PVC - Persistent Volume match criterias; sufficent capacity, access modes, volume modes, storage class, selector (if any)
- If there are no other options PVC could match with larger Persistent Volume.
- If there is no reliable Persistent Volume, the PVC will stay in Pending state
- Same definition file on deployments, replicasets, pods
### Persistent Volume Claims(PVC) Commands
- kubectl delete persistentvolumeclaim <pvc_name> (Delete PVC)
### Storage Classes
- Static Provisioning; before creation of th pv you should create the storage in AWS, GCP, local etc.
- Dynamic Provisioning; automatically creates the storage when a claim is made;
- To implement dynamic provisioning, you use Storage Class in Kubernetes.
- You don't need to create a persistent volume since the storage provisioning creates both storage and persistent volume automatically.
## Networking
### Switching & Routing
- Switching, connection bridge for 2 seperate computers, systems within the same network
- When the ip CIDRs are assigned, the computers could ping each other through switch
- Routing, connection bridge for 2 seperate networks
- /proc/sys/net/ipv4/ip_forward (IP forward file default value = 0, to enable forwarding change it to=1)
  To enable ip forwarding across reboots you should modify also /etc/sysctl.conf file (net.ipv4.ip_forward = 1)
- To persist ip command changes you should change them in /etc/network/interfaces file
### Switching & Routing Commands
- ip link (List and modify interfaces on the host - eth0)
- ip addr (List ip addresses assigned to the interfaces)
- ip addr add <network_CIDR> dev <interface_no(eth0)> (Assigning network IP address CIDR)
- ping <ip_addr> (Ping the other system)
- route (Displays kernel IP routing)
- ip route add <target_network_CIDR> via <ip_addr> (Add route)
- ip route add default via <ip_addr> (Add default router)
### DNS
- Assign names to IPs
- Local ip name file -> /etc/hosts, can have multi names for one ip (Name Resolution)
- DNS server holds all network names in one place
- /etc/resolv.conf (DNS-nameserver pointer file)
- Local /etc/hosts file has priority over /etc/resolv.conf dns file, if same IP has record on both /etc/hosts & /etc/resolv.conf file, /etc/hosts record will be used.
  But you can change this priority order in /etc/nsswitch.conf file.
- . (Root)
  .com/.net/.edu/.org/.io (Top level domain)
  www (Subdomain)
- DNS server can cache the IP you sent in order to improve search perf
- Record Types;
  A = IPv4
  AAAA = IPv6
  CNAME = food.web-server | eat.web-server | hungry.web-server (name to name mapping)
### DNS Commands
- ping www.google.com
- nslookup www.google.com (query a host name from a dns server)
- dig www.google.com (More detailed query result)
### Network Namespaces
- Host = house, namespace = room, veth=pipe, virtual cable
- You can create a virtual switch to connect many namespaces to each other easily (LINUX BRIDGE)
- Network Creation Steps
  1- Create Network Namespace
  2- Create Bridge Network/Interface
  3- Create veth pairs(pipe, virtual cable)
  4- Attach veth to Namespace
  5- Attach other veth to bridge
  6- Assign IP addresses
  7- Bring the interfaces up
  8- Enable NAT - IP Masquerade
### Network Namespaces Commands
- ps aux (List processes)
- ip netns add <namespace_name> (Add namespace to host)
- ip netns (List namespace in the host)
- ip netns exec <namespace_name> ip link (ip link command will be executed in the given namespace)
- ip -n <namespace_name> link (ip link command will be executed in the given namespace)
- ip link add <veth_name1> type veth peer name <veth_name2> (Add cable between namespaces)
- ip link set <veth_name> netns <namespace_name> (Assign bridge gate to the namespace)
- ip -n <namespace_name> addr add <ip_addr> dev <veth_name> (add ip to veth)
- ip -n <namespace_name> link set <veth_name> up (Waking veth links)
- arp (Check ip statuses)
- ip netns exec <namespace_name> arp (Check ips in the namespace)
- ip link add <bridge_name> type bridge (Create virtual switch)
#### Network Virtual Bridge Commands
- ip link set dev <bridge_name> up (Wake up bridge)
- ip -n <namespace_name> link del <veth_name> (Delete the veth, if you delete one end of the veth, other end will be deleted automatically)
- ip link add <veth_name1> type veth peer name <bridge_name> (Link veth to the virtual switch)
- ip link set <veth_name> netns <namespace_name> (Add veth to the namespace)
- ip link set <bridge_name> master <virtual_switch_name> (Link bridge to the virtual switch)
- ip -n <namespace_name> addr add <ip_addr> dev <veth_name> (add ip to veth)
- ip link set dev <veth_name> up (Wake up veth)
- ip addr add <target_CIDR> dev <virtual_switch_name> (Add ip addr to virtual switch)
- ip addr show eth0 (Shows eth0 only)
  ip addr show bridge (Shows bridge only)
### Network Namespaces Commands
- iptables -t nat -A POSTROUTING -s <CIDR> -j MASQUERADE (Add NAT to the system)
- iptables -t nat -A PREROUTING --dport 80 --to-destination <ip_addr:port> -j DNAT (Open namespace ip to the internet on port 80)
### Docker Networking
- Docker has a switch default in 192.168.1.10 - eth0
- Whenever a container is created Docker creates a network namespace for it
  Also creates the interfaces and attach them to the bridge and to container for networking
### Docker Networking Commands
- docker run --network none <image_name> (Run container without network)
- docker run --network host <image_name> (Run container in the host)
- docker run <image_name> (Run containers connected to the bridge)
- ip link add <bridge_name(docker0)> type bridge (Add bridge)
- docker inspect <namespace_id>
### Container Networking Interface(CNI)
- Set of standards that defines how programs should be developed to solve networking challenges.
- Defines how the plugin should be developed and how container run times should invoke them.
- Default supported plugins; BRIDGE, VLAN, IPVLAN, MACVLAN, WINDOWS, DHCP, host-local
- ADD, DEL
### Cluster Networking
- Each node has eth0
- Ports;
  ETCD=2379, ETCD Clients=2380, Kube-api=6443, kubelet=10250, kube-scheduler=10251, kube-controller-manager=10252, services=30000<->32767
### Pod Networking
- Every pod should have an IP
- Every pod should be able to communicate with every other POD in the same node
- Every pod should be able to communicate with every other POD on other nodes without NAT
- kubelet checks, --cni-conf-dir=/etc/cni/net.d when creating a container
  then checks, --cni-bin-dir=/etc/cni/bin, and executes the script inside the folder
  './net-script.sh add <container> <namespace>'
### Cluster Networking in Kubernetes (CNI)
- CNI config = kubelet.service -> --network-plugin=cni, --cni-bin-dir=/opt/cni/bin, --cni-conf-dir=/etc/cni/net.d
- /opt/cni/bin includes all supported CNI plugins and executables
- /etc/cni/net.d includes config files
### CNI Weave
- Creates daemonsets on all nodes to communicate with the pods
- Creates its own bridge
- Check Kubernetes installing add-ons section
- Deploy Weave; 
  kubectl apply -f "https://cloud.weave.works/k8s/net...." 
- If you use kubeadm tool to deploy Weave you can see weave pods
### IP Address Management - Weave
- CNI is responsible on assigning IPs
- ip-list.txt is the file on every node that is keeping ip addresses
- CNI has two plugins that are managing IPs; DHCP and host-local
- /etc/cni/net.d/net-script.conf (Config file for CNI)
- Weave default IP range = 10.32.0.0/12
### Service Networking
- Services are cluster wide components, it is a virtual object.
- A service is accessible through all pods on the cluster, irrespective of what nodes are the pods are on (Cluster IP)
- NodePort enables external access to the node.
- kube-proxy component creates or deletes IP:Port / Service Forward To rules for services when a service is created or deleted.
- kube-proxy --proxy-mode [userspace | iptables | ipvs] ...
  iptables -> IP:Port / Forward To (IP)
- kube-api-server --service-cluster-ip-range ipNet (Default: 10.0.0.0/24) (Service IP assignment range)
### Service Networking Commands
- ps aux | grep kube-api-server (Gets kube-api-server configs)
- iptables -L -t nat | grep <service_name> (Get ip table for the service)
- cat /var/log/kube-proxy.log (See kube-proxy logs)
- kubectl logs <kube-proxy_pod_name> (Get proxy configured in kube-proxy)
### DNS in Kubernetes
- Built in DNS-server = Kube DNS
- When a service is created a record is created accordinglyl in Kube DNS
- <service_name>.<namespace_name>.svc.cluster.local (Service FQDN)
- <pod_IP_with_dashes>.<namespace_name>.pod.cluster.local (Pod FQDN)
  http://10-244-2-5.apps.pod.cluster.local
### CoreDNS in Kubernetes
- /etc/hosts file is local DNS name server
- /etc/resolv.conf file is pointer for global DNS name server
- Pod entries are done by Kubernetes on global DNS name server are as below;

  10-244-2-5  10.244.2.5 <br> 
  10-244-1-5  10.244.1.5 <br>
  10-244-2-15 10.244.2.15 <br>
  web-service 10.107.37.188 <br>

- CoreDNS is deployed as a replicaset in kube-system namespace
- ./Coredns executable
- cat /etc/coredns/Corefile includes plugins and configs
- /etc/resolv.conf file could be passed as a configMap object to the CoreDNS
- CoreDNS creates a service to be reachable by other components in Kubernetes
  kube-dns service is default name for it
- cat /var/lib/kubelet/config.yaml (You can see IP addres of the DNS server under clusterDNS: <IP> options)
- /etc/resolv.conf file has a search entry;
  (search default.svc.cluster.local svc.cluster.local cluster.local)
### CoreDNS in Kubernetes Commands
- curl <service_name>
- host <service_name> (See service FQDN and IP address, you can reach a service its name)
  host web-service - web-service.default.svc.cluster local has address 10.107.37.188
- host <pod_FQDN> (You can not reach a pod using its name, you should use pod FQDN)
### Ingress
- Ingress helps your users access your app using a single externally accessible URL that you can configure to route traffic to different services within your cluster based on the URL path, at the same time implements SSL.
- Ingress is like a layer 7 load balancer built in to Kubernetes cluster.
- To expose it to outside you need a NodePort or Load Balancer
- Kubernetes cluster has no built in Ingress controller, you must deploy one. (GCP Load Balancer, NGINX, Contour, Haproxy, Traefik, Istio...)
- The solution you deploy for Ingress is called = INGRESS CONTROLLER
  The rules that you configure is called = INGRESS RESOURCES 
- If a user tries to access a URL that does not match any of the rules on Ingress, then the user is directed to the service specified as the default backend. (Must remember to deploy a default service. For ex; error page)
### Ingress Commands
- kubectl describe ingress <ingress_name>
## Design and Install a Kubernetes Cluster
- Minikube for single node cluster, for testing developing
- Kubeadm for on-prem
  GKE for Google Cloud
  Kops for AWS
  Azure Kubernetes Service
- Master can host workloads , best practice is not to host workloads on Master nodes.
- You can seperate ETCD from the master node as a seperate node, on complex clusters
### High Availability in Kubernetes
- HA multi master nodes' kube-api-server needs a load balancer in front
- HA multi master nodes' kube-controller-manager --leader-elect=true prop should be selected to make other controller managers in standby mode
  --leader-elect=true <br>
  --leader-elect-lease-duration=15s <br>
  --leader-elect-renew-deadline=10s <br>
  --leader-elect-retry-period=2s <br>
- ETCD stacked topology, for HA seperate ETCD node = external ETCD Topology - harder to setup <br>
  If you use external ETCD topology config kube-api-server as --etcd-servers=https://<ip>:<port>,...
### ETCD HA
- Distributed key-value store
- HA ETCD, since the same data is available across all nodes, you can easily read from any ETCD node.
  For writes, internally two nodes elects a leader among them, and when one node becomes a leader other nodes become followers. <br>
  if the writes came through in the leader node, leader node processes the write and make sure that other nodes are sent a copy of data. <br>
  If the writes came in through any of the follower nodes, they forward the writes to the leader internally and leader processes the writes and send copies of the writes to the followers. <br>
  Writes is only considered completed only when all nodes finish writes. If some nodes fail, a write is considered to be complete if it is written to the majority(Quorum) of the nodes. <br>
  If the failed node comes back, the write will be written to it. <br>
  Majority - Quorum = ROUNDDOWN(N/2 + 1) <br>
  Fault Tolerance = Instance Qty - Quorum <br>
  When selecting the number of nodes it is logical to select odd numbers, because fault tolerance is same for 3 & 4 for example.
- etcd.service --initial-cluster peer-1=https://${PEER1_IP}:<port>, peer-2=....
## Install Kubernetes with kubeadm
- kubeadm helps installing
  Master node; kube-apiserver, etcd, node-controller, replica-controller
  Worker node; kubelet
- Steps;
  1- Need multiple systems or VMs provisioned, a couple of nodes for our Kubernetes cluster
  2- Install containerd on every node
  3- Install kubeadm on every node
  4- Initialize master server
  5- Setup pod network
  6- Join worker nodes to master node
### Deployment with kubeadm
- Installing kubeadm is the main page to look
- Linux systems have control groups(Cgroup drivers) that are used to constrain resources that are allocated to different processes running on you machine.
  There are 2 types of Cgroup drivers; cgroupfs, systemd
  If containerd is using systemd all kubelet drivers must use systemd.
### Deployment with kubeadm Commands
- ps -p 1 (Checks cgroup driver type)
- sudo systemctl restart containerd (Restarts containerd service)
#### Installing kubeadm, kubelet,kubectl
- Forwarding IPv4 and letting iptables see bridged traffic <br>
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf <br>
br_netfilter <br>
EOF <br>

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf <br>
net.bridge.bridge-nf-call-ip6tables = 1 <br>
net.bridge.bridge-nf-call-iptables = 1 <br>
EOF <br>
- install kubeadm, kubectll, kubelet
sudo apt-get update <br>
sudo apt-get install -y apt-transport-https ca-certificates curl <br>

mkdir -p /etc/apt/keyrings <br>

curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg <br>

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list <br>

sudo apt-get update <br>

sudo apt-get install -y kubelet=1.27.0-00 kubeadm=1.27.0-00 kubectl=1.27.0-00 <br>

sudo apt-mark hold kubelet kubeadm kubectl <br>

- controlplane node initialize kubeadm
  kubeadm init --apiserver-advertise-address=192.20.4.9 --apiserver-cert-extra-sans=controlplane --pod-network-cidr=10.244.0.0/16 <br>

  mkdir -p $HOME/.kube <br>
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config <br>
  sudo chown $(id -u):$(id -g) $HOME/.kube/config <br>
- ssh to other nodes and run
  kubeadm join 192.20.4.9:6443 --token m6y12t.x4zutx5kymj0jalw \ <br>
        --discovery-token-ca-cert-hash sha256:f2665e279232d19bd661c87f5bedad9ade756d21ada4cd7e3b2fbe53e8fcd24f <br>
## Troubleshooting
### Application Failure
- Check accessibility
  1- use curl for web server if app is accessible
  2- check service (Compare selectors)
  3- check pod
  4- check pod logs <br>
     kubectl logs <pod_name> <br>
     kubectl logs <pod_name> -f --previous <br>
  5- check dependent services, pods
### Controlplane Failure
- Check node status
- Check pods
- Check manifests
- Check controlplane pods
- Check controlplane services <br>
  service kube-apiserver status <br>
  service kube-controller-manager status <br>
  service kube-scheduler status <br>
  service kube-proxy status <br>
  service kubelet status <br>
- Check pod logs <br>
  kubetl logs <controlplane_pod_name> -n kube-system <br>
  sudo journalctl -u kube-apiserver <br>
### Worker Node Failure
- Check node status <br>
  Commands; <br>
  top <br>
  df -h <br>
- Check kubelet; <br>
  service kubelet status <br>
  sudo journalctl -u kubelet <br>
  openssl x509 -in /var/lib/kubelet/worker-1.crt -text <br>
### Networking Failure
- Kubernetes resources for coreDNS are:   <br>
  1- A service account named coredns, <br>
  2- Cluster-roles named coredns and kube-dns <br>
  3- Clusterrolebindings named coredns and kube-dns,  <br>
  4- A deployment named coredns, <br>
  5- A configmap named coredns and a <br>
  6- Service named kube-dns. <br>
- Port 53 is used for for DNS resolution.
#### Troubleshooting issues related to coreDNS
- If you find CoreDNS pods in pending state first check network plugin is installed.
- Coredns pods have CrashLoopBackOff or Error state <br>
  If you have nodes that are running SELinux with an older version of Docker you might experience a scenario where the coredns pods are not starting. To solve that you can try one of the following options: <br>
  1- Upgrade to a newer version of Docker. <br>
  2- Disable SELinux. <br>
  3- Modify the coredns deployment to set allowPrivilegeEscalation to true: <br>
     kubectl -n kube-system get deployment coredns -o yaml | \ <br>
     sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \ <br>
     kubectl apply -f - <br>
  4- Another cause for CoreDNS to have CrashLoopBackOff is when a CoreDNS Pod deployed in Kubernetes detects a loop. <br>
      There are many ways to work around this issue, some are listed here: <br>
      - Add the following to your kubelet config yaml: resolvConf: <path-to-your-real-resolv-conf-file> This flag tells kubelet to pass an alternate resolv.conf to Pods. For systems using systemd-resolved, /run/systemd/resolve/resolv.conf is typically the location of the "real" resolv.conf, although this can be different depending on your distribution. <br>
      - Disable the local DNS cache on host nodes, and restore /etc/resolv.conf to the original. <br>
      - A quick fix is to edit your Corefile, replacing forward . /etc/resolv.conf with the IP address of your upstream DNS, for example forward . 8.8.8.8. But this only fixes the issue for CoreDNS, kubelet will continue to forward the invalid resolv.conf to all default dnsPolicy Pods, leaving them unable to resolve DNS. <br>
- 3. If CoreDNS pods and the kube-dns service is working fine, check the kube-dns service has valid endpoints. <br>
     - kubectl -n kube-system get ep kube-dns <br>
     If there are no endpoints for the service, inspect the service and make sure it uses the correct selectors and ports. <br>
#### Troubleshooting issues related to kube-proxy
- Check kube-proxy pod in the kube-system namespace is running.
- Check kube-proxy logs.
- Check configmap is correctly defined and the config file for running kube-proxy binary is correct.
- kube-config is defined in the config map.
- check kube-proxy is running inside the container
  netstat -plan | grep kube-proxy
## Other Topics
### YAML Notes
- Dictionary is an unordered list, array/list is an ordered list.
- Dictionaries are equal if all their props and values are same, regardless of the order.
- Arrays/Lists are NOT equal if the order is different even if the items are same.
### JSON Path
- $ sign means root curly paranthesis in json = {}
- Get parent value = $.<parent_dictionary_name>
- Get child value = $.<parent_dictionary_name>.<child_dictionary_name>...
- Get list value = $[0] - First element <br>
                   $[2] - 3rd element <br>
                   $[0, 2] - 1st & 3rd element <br>
                   $[0:2] - 1st to 3rd element not included 3rd element, 1st element included <br>
                   $[-1:] - Gets the last element on the list <br>
                   $[-3:] - Gets the last 3 elements on the list <br>
                   $[?(@ > 40)] - Gets values greater than 40 <br>
                   $[?(@ == 40)] - Gets values equals to 40 <br>
                   $[?(@ != 40)] - Gets values not equals to 40 <br>
### JSON Path Wild Cards
- $.*.color (Get all keys color prop values)
- $.[*].model (Get all items model prop values)
- $.*.wheels[*].model (Get all props wheels model prop values)
- * returns array always
### JSON Path Kubernetes
- kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'
- kubectl get nodes -o=jsonpath='{.items[*].metadata.name'} {"\n"} {.items[*].status.capacity.cpu}'
- kubectl get nodes -o=jsonpath='{range .items[*]}{"\t"}{.status.capacity.cpu}{"\n"}{end}'
- kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
- kubectl get nodes --sort-by=.metadata.name
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
- kubectl get pods | wc -l (Counts lines on the output)
- kubectl get <resource_name> --selector env=dev,bu=finance --no-headers (selects resources with env=dev and bu=finance label)
- kubectl create -f . (Creates all files within the folder)
- kubectl exec -n <namespace_name> -it <pod_name> -- cat <file_name>  (Runs the command after -- sign in the pod)
- kubectl exec <pod_name> <command> (Runs command on the pod)
- kubectl logs <pod_name> (Get pod logs)
- alias k=kubectl (shortcut k defined in terminal to call kubectl)
- cat /etc/*release* (Find current OS)
- export ETCDCTL_API=3 (Configures ETCDCTL_API to 3)
- kubectl config <arg> (kubernetes config command to see, update, add contexts, clusters...)
- ps -ef | grep etcd | grep ... (Check service configurations)
- <command> | grep \-\-etcd (Equals to 'grep --etcd')
- docker ps -a (List all containers)
- docker logs <container_id> (View container logs)
- cat <file> | base64 -w 0 (convert file to base64 as single line)
- echo $HOME (shows home folder name - /root)
- ps aux (List running processes)
- whoami (Gets current user)
- netstat -anp |grep <port_no> |wc -l (Count active connections on port)
- k exec <pod_name> -- nslookup <service_name> > /<path>/<file_name> (Writes nslookup output to the file that is specified)
- service <service_name> start (Starts service)
- curl -LO https://dl.k8s.io/release/v1.27.0/bin/linux/amd64/kubectl
- ping <ip> (You can check communication by pinging)
