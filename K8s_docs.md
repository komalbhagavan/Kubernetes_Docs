# Cluster Architecture
-------------------------------------------
![clusrter_architecture](https://devopscube.com/wp-content/uploads/2022/12/k8s-architecture.drawio-1.png)

--------------------------------------------------------------------------------
### Nodes

Kubernetes runs your workload by placing containers into Pods to run on Nodes. A node may be a virtual or physical machine, depending on the cluster. **Each node is managed by the control plane** and contains the services necessary to run Pods.

Typically you have several nodes in a cluster; in a learning or resource-limited environment, you might have only one node.

The components on a node include the kubelet, a container runtime, and the kube-proxy

**In Kubernetes, you can perform several operations with nodes. Here are some of the key operations:**
1. **Node Management**: You can add new nodes to the cluster or remove existing nodes as needed. This is important for scaling your cluster up or down based on workload requirements.
2. **Node Maintenance**: You can mark nodes as "unschedulable" to prevent new workloads from being assigned to them. This is useful when you want to perform maintenance tasks on a node.
3. **Node Labels and Taints**: You can assign labels to nodes, allowing you to categorize and group nodes based on various criteria. Taints can be applied to nodes to repel certain workloads, ensuring that only specific pods can be scheduled on those nodes.
4. **Node Inspections**: You can inspect the status and resource utilization of nodes in your cluster. This helps in monitoring and optimizing resource allocation.
5. **Node Upgrades**: You can perform rolling updates of node software (like the container runtime or the OS) to ensure nodes are running the latest and most secure versions.
6. **Node Eviction**: If a node is experiencing issues (like excessive resource usage or hardware failures), you can evict workloads from that node and reschedule them on other healthy nodes.
7. **Node Troubleshooting**: You can diagnose and troubleshoot node-related issues, such as connectivity problems, resource constraints, or misconfigurations.
8. **Node Annotations**: You can attach annotations to nodes, providing additional metadata for management purposes.

### Node Components
#### kubelet

An agent that runs on each node in the cluster. It makes sure that containers are running in a Pod.

**TASK1** -> Register a node as a worker node with api-srver in the kubernates cluster

**TASK2** -> Creates/Destroys/Runs the PODS on the worker nodes based on POD Specs received from Kubernetes API-server primarily

**TASK3** -> Reports the status of the PODS and the nodes to the Kubernetes API-Server regularly

The kubelet takes a set of PodSpecs that are provided and ensures that the containers described in those PodSpecs are running and healthy.

![kubelet](https://devopscube.com/wp-content/uploads/2023/01/kubelet-architecture-830x1024.png "kubelet")
#### kube-proxy

kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.

kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.

-> when we create a deployment and make that deployment as a service in which the kube-proxy comes in picture

Service in Kubernetes is a way to expose a set of pods internally or to external traffic. When you create the service object, it gets a virtual IP assigned to it. It is called clusterIP. It is only accessible within the Kubernetes cluster.

**When you expose pods using a Service (ClusterIP), Kube-proxy creates network rules to send traffic to the backend pods (endpoints)** grouped under the Service object. Meaning, all the load balancing, and service discovery are handled by the Kube proxy.

[**So how does Kube-proxy work?**](https://www.youtube.com/watch?v=QUqQ7HjJ4Qw)

Kube proxy talks to the API server to get the details about the Service **(ClusterIP)/(NodePort)** and respective pod IPs & ports (endpoints). It also monitors for changes in service and endpoints.

**kube-proxy modes**

>**IPTables**: It is the default mode. In IPTables mode, the traffic is handled by IPtable rules. In this mode, kube-proxy chooses the backend pod random for load balancing. Once the connection is established, the requests go to the same pod until the connection is terminated.
**IPVS**: For clusters with services exceeding 1000, IPVS offers performance improvement.

![kube-proxy](https://devopscube.com/wp-content/uploads/2023/01/image-14.png "kube-proxy")

### Container Runtime
container runtime is a software component that is required to run containers.

Container runtime runs on all the nodes in the Kubernetes cluster. It is responsible for pulling images from container registries, running containers, allocating and isolating resources for containers, and managing the entire lifecycle of a container.

**To understand this better, let’s take a look at two key concepts:**

**Container Runtime Interface (CRI):** It is a set of APIs that allows Kubernetes to interact with different container runtimes. It allows different container runtimes to be used interchangeably with Kubernetes. The CRI defines the API for creating, starting, stopping, and deleting containers, as well as for managing images and container networks.
**Open Container Initiative (OCI):** It is a set of standards for container formats and runtimes

![container runtime](https://devopscube.com/wp-content/uploads/2022/12/image-5.png "container runtime")

#### How is communication between the control plane and worker nodes secured in Kubernetes?

Communication between the control plane and worker nodes is secured using PKI certificates and communication between different components happens over TLS. This way, only trusted components can communicate with each other.

#### Node to Control Plane

All API usage from nodes (or the pods they run) terminates at the API server. None of the other control plane components are designed to expose remote services. The API server is configured to listen for remote connections on a secure HTTPS port (typically 443) with one or more forms of client authentication enabled. One or more forms of authorization should be enabled

Nodes should be provisioned with the public root certificate for the cluster such that they can connect securely to the API server along with valid client credentials. A good approach is that the client credentials provided to the kubelet are in the form of a client certificate

Pods that wish to connect to the API server can do so securely by leveraging a service account so that Kubernetes will automatically inject the public root certificate and a valid bearer token into the pod when it is instantiated.

#### Control plane to node

There are two primary communication paths from the control plane (the API server) to the nodes. The first is from the API server to the kubelet process which runs on each node in the cluster. The second is from the API server to any node, pod, or service through the API server's proxy functionality.

**The connections from the API server to the kubelet are used for:
**
Fetching logs for pods.
Attaching (usually through kubectl) to running pods.
Providing the kubelet's port-forwarding functionality.

These connections terminate at the kubelet's HTTPS endpoint. By default, the API server does not verify the kubelet's serving certificate, which makes the connection subject to man-in-the-middle attacks and unsafe to run over untrusted and/or public networks.

To verify this connection, use the --kubelet-certificate-authority flag to provide the API server with a root certificate bundle to use to verify the kubelet's serving certificate.

If that is not possible, use SSH tunneling between the API server and kubelet if required to avoid connecting over an untrusted or public network.

### Controllers
In Kubernetes, controllers are control loops that watch the state of your cluster, then make or request changes where needed. Each controller tries to move the current cluster state closer to the desired state.


controllers basically work on two things
-> Monitor the curent state of the one or more kubernetes resources
-> Take action to  bring it to the desired state
Example:
>Let’s say you want to create a deployment, you specify the desired state in the manifest YAML file (declarative approach). For example, 2 replicas, one volume mount, configmap, etc. The in-built deployment controller ensures that the deployment is in the desired state all the time. If a user updates the deployment with 5 replicas, the deployment controller recognizes it and ensures the desired state is 5 replicas.

**Multiple controllers **
**Following is the list of important built-in Kubernetes controllers.
**Deployment controller
- Replicaset controller
- DaemonSet controller 
- Job Controller (Kubernetes Jobs)
- CronJob Controller
- endpoints controller
- namespace controller
- service accounts controller.
- Node controller
-> Custom-Controller: create your own controller for your specific use case
-> Multiple controllers can be managing the same kind of kubernetes object
for example: 
                    ```Deployment and jobs both create pods```

**Control via APIserver**
Most commonly a controller will request the API server to do something, for example, create a pod if needed. API Server creates a pod object without assigning a node. Then scheduler and Kubelet components will perform the remaining steps to schedule and run the pod on an optimal node.

The Job controller is an example of a Kubernetes built-in controller. Built-in controllers manage state by interacting with the cluster API server.

When the Job controller sees a new task it makes sure that, somewhere in your cluster, the kubelets on a set of Nodes are running the right number of Pods to get the work done. The Job controller does not run any Pods or containers itself. Instead, the Job controller tells the API server to create or remove Pods. Other components in the control plane act on the new information and eventually the work is done.

![controller](https://devopscube.com/wp-content/uploads/2023/01/image-9-800x1188.png "controller")

---

### Cloud Controller Manager
When kubernetes is deployed in cloud environments, the cloud controller manager acts as a bridge between Cloud Platform APIs and the Kubernetes cluster.

This way the core kubernetes core components can work independently and allow the cloud providers to integrate with kubernetes using plugins. (For example, an interface between kubernetes cluster and AWS cloud API)

Cloud controller integration allows Kubernetes cluster to provision cloud resources like instances (for nodes), Load Balancers (for services), and Storage Volumes (for persistent volumes).

![Cloud Controller Manager](https://devopscube.com/wp-content/uploads/2023/01/Cloud-Controller-Manager.png "Cloud Controller Manager")

Cloud Controller Manager contains a set of specific controllers that ensure the desired state of cloud-specific components (nodes, Loadbalancers, storage, etc). Following are the three main controllers that are part of the cloud controller manager.

**Node controller**: This controller updates node-related information by talking to the cloud provider API. For example, node labeling & annotation, getting hostname, CPU & memory availability, nodes health, etc.

**Route controller**: It is responsible for configuring networking routes on a cloud platform. So that pods in different nodes can talk to each other.

**Service controller**: It takes care of deploying load balancers for kubernetes services, assigning IP addresses, etc.

### kube-apiserver
The kube-api server is the central hub of the Kubernetes cluster that exposes the Kubernetes API.

End users, and other cluster components, talk to the cluster via the API server. Very rarely monitoring systems and third-party services may talk to API servers to interact with the cluster.

So when you use kubectl to manage the cluster, at the backend you are actually communicating with the API server through HTTP REST APIs. However, the internal cluster components like the scheduler, controller, etc talk to the API server.

The communication between the API server and other components in the cluster happens over TLS to prevent unauthorized access to the cluster.

![kube-apiserver](https://devopscube.com/wp-content/uploads/2022/10/kube-api-server.drawio-1.png.webp "kube-apiserver")

**Kubernetes api-server is responsible for the following
**
- **API management:** Exposes the cluster API endpoint and handles all API requests.
- Authentication (Using client certificates, bearer tokens, and HTTP Basic Authentication) and Authorization (ABAC and RBAC evaluation)
- Processing API requests and validating data for the API objects like pods, services, etc. (Validation and Mutation Admission controllers)
- It is the only component that communicates with etcd.
- api-server coordinates all the processes between the control plane and worker node components.
--------

#### etcd

etcd is a distributed reliable key-value store that is simple secure and fast.

when we install etcd in our cluster there will be a default client called ectd control client it is a command line client for etcd. to retrive data we use etcdctl command.
- open source distributed key-value store
- primary data store for kubernetes
-  strongly consistent, distributed key-value store,high secure,reliability
- Runs on master node
- uses RAFT algorthim - For leader election process
- etcd uses watch function 

To put it simply, when you use kubectl to get kubernetes object details, you are getting it from etcd. Also, when you deploy an object like a pod, an entry gets created in etcd.

![etcd](https://devopscube.com/wp-content/uploads/2023/01/image-5-1024x948.png "etcd")

---

### kube-scheduler
The kube-scheduler is responsible for scheduling pods on worker nodes.

When you deploy a pod, you specify the pod requirements such as CPU, memory, affinity, taints or tolerations, priority, persistent volumes (PV),  etc. The scheduler’s primary task is to identify the create request and choose the best node for a pod that satisfies the requirements.

![kube-scheduler](https://devopscube.com/wp-content/uploads/2023/01/image-8-2048x1921.png "kube-scheduler")

In a Kubernetes cluster, there will be more than one worker node. So how does the scheduler select the node out of all worker nodes?

Here is how the scheduler works.

- To choose the best node, the Kube-scheduler uses filtering and scoring operations.
- In filtering, the scheduler finds the best-suited nodes where the pod can be scheduled. For example, if there are five worker nodes with resource availability to run the pod, it selects all five nodes. If there are no nodes, then the pod is unschedulable and moved to the scheduling queue. If It is a large cluster, let’s say 100 worker nodes, and the scheduler doesn’t iterate over all the nodes. There is a scheduler configuration parameter called percentageOfNodesToScore. The default value is typically 50%. So it tries to iterate over 50% of nodes in a round-robin fashion. If the worker nodes are spread across multiple zones, then the scheduler iterates over nodes in different zones. For very large clusters the default percentageOfNodesToScore is 5%.
- In the scoring phase, the scheduler ranks the nodes by assigning a score to the filtered worker nodes. The scheduler makes the scoring by calling multiple scheduling plugins. Finally, the worker node with the highest rank will be selected for scheduling the pod. If all the nodes have the same rank, a node will be selected at random.
- Once the node is selected, the scheduler creates a binding event in the API server. Meaning an event to bind a pod and node.

---
