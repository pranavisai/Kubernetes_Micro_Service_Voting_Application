# Kubernetes_Micro_Service_Voting_Application



# Kubernetes Notes

A container orchestration technology that manages and deploys thousands of containers in a cluster.

1. Cluster: It is a set of nodes. Used so that in case one of the nodes fails, the others are available to keep the services running.
2. Cluster has a Control Plane (Master Node) and workers. 
3. The information of the cluster is stored, and the management of the nodes is done through the Control Plane.
4. The Control Plane is another node that has Kubernetes components installed in it.
5. The Kubernetes components installed are: kube-apiserver, etcd, controller-manager, and kube-scheduler.
6. kube-apiserver: acts as the frontend for Kubernetes. All outside interactions happen through this.
7. etcd: It is a distributed, reliable key-value store by Kubernetes to store all the data required to manage the cluster.
8. controller-manager: The brain behind the orchestration. Responsible for noticing and responding when the modes, containers, or endpoints go down. The controllers make decisions to bring up new containers in such cases.
9. kube-scheduler: It is responsible for distributing work or containers across multiple modes for newly created containers and assigns them to nodes.
10. On worker nodes, we have the kublet: This is the agent that runs on each node in the cluster. This agent makes sure that the containers are running on the nodes as expected.
11. On the worker nodes, we also have kube-proxy: This is responsible for maintaining the rules on the nodes. It helps in communicating with each other. The worker nodes and the containers need to communicate with each other.
12. We also have "container runtime" on the worker nodes: It is responsible for running the containers. This is important because we are running applications in the form of containers.
13. Kubernetes was originally made to support the runtime of Docker containers specifically, but over time, it evolved to support other container runtimes. It no longer supports the runtime component of Docker, which is managed by containerd.

# Kubectl
1. It is the command-line utility of Kubernetes.
2. It is the tool to operate the Kubernetes cluster. Eg: To view the status of the cluster, to provision applications, to scale up, down, delete, and so on.
## commands
1. kubectl version
2. kubectl --help
3. kubectl get nodes -> for the name and number of the nodes, the status, roles, age (how long it has been up), version of Kubernetes running on the system,
4. kubectl get nodes -o wide -> for all the ones above and extra -> internal-ip, external-ip, os-image, kernel-version, container-runtime.

# PODS
1. Kubernetes does not deploy containers directly on the worker nodes. The containers are encapsulated into a Kubernetes object known as pods.
2. A pod is a single instance of an application. It is the smallest object that can be created in Kubernetes.
3. When we want to scale up, we do not add more containers to the pods. We create more pods.

## commands
1. Example of creating a pod from an image: kubectl run podname --image=imagename
2. The images are pulled from the Docker Hub, which has the latest images of various applications. Kubernetes can be configured to pull various images from the public or private repository according to the organization's rules.
3. Checking if the pod is running? -> kubectl get pods
4. to describe the pods -> kubectl describe pod "pod-name".
5. to delete all pods -> kubectl delete pods --all
6. to delete a specific pod -> kubectl delete pod pod-name

# YMAL files
1. Important information to be put into every YMAL file: apiVersion, kind, metadata, and spec.
2. The apiVersions are different for different kinds. Pod -> v1, Service -> v1, ReplicaSet -> apps/v1, Deployment -> apps/v1.
3. metadata is generally a dictionary.
4. To create the pod through a YAML file -> kubectl create -f YAML_file_name.yml
5. If we use an image registry other than Docker Hub, then give the whole path directory.

# Replica sets
1. The replicas are created so that there is no downtime in case of error.
2. Replicaset makes sure that at a point in time, there are as many replicas running as given in the YAML file.
3. So, in case you delete a pod, another pod is spawned in its place.
4. To identify the pods in one replicaset, the selector matchLabels type is used in the YAML file.
5. To make changes to a replicaset, we have two options:
   1. Either delete and recreate the ReplicaSet
   2. Update the existing ReplicaSet and then delete all PODs, so new ones with the correct image will be created.
6. To scale a replicaset, use the command -> kubectl scale replicaset replica-set-name --replicas=no-of-replicas-number.
7. But, if you actually use the original YAML file is not updated. Only the replicas are updated. So, it's always a good move to correct the YAML file to scale.

## commands
1. To create a replicaset using a YAML file -> kubectl create -f yaml-filename.yaml
2. To check the replicaset -> kubectl get replicaset
3. Delete a replicaset -> kubectl delete replicaset replicaset-name
4. to replace a replicaset -> kubectl replace -f replicaset-name
5. Scaling the replicaset:
   1. One way -> kubectl edit rs replicaset-name
   2. Second way:
     1. kubectl get rs yaml-filename -o yaml > yaml-filename.yaml
     2. Edit the YAML file -> vi yaml-filename.yaml
     3. apply YAML changes -> kubectl apply -f replicaset-name
  
# Deployments
1. A Deployment is a Kubernetes object that manages ReplicaSets and Pods, ensuring the desired number of application instances are running.
2. The hierarchy is: Deployment -> Replicasets -> Pods
3. Why do we use Deployment:
   1. Maintains the desired number of Pods.
   2. Supports scaling up/down.
   3. Enables rolling updates (e.g., update image versions without downtime).
   4. Supports rollbacks to previous versions.
   5. Self-healing through the underlying ReplicaSet.

## commands
1. Check deployment: kubectl get deployments
2. to scale a deployment -> kubectl scale deployment nginx --replicas=5 (if to change without a YAML file)
3. Create a deployment through a YAML file -> kubectl create -f yaml-filename.yaml
4. Creating without a YAML file -> kubectl create deployment name --image=image_name --replicas=no_of_replicas

# Services
1. A Service in Kubernetes is an abstraction that provides a stable network endpoint for a group of Pods. Since Pods are ephemeral (last for a short time) and their IP addresses can change when they are recreated, a Service ensures that applications can reliably communicate with Pods without needing to know their individual IP addresses.
2. A Service uses labels and selectors to identify and route traffic to the appropriate Pods.
3. Services provide load balancing across multiple Pods.
4. Services enable communication within and outside the cluster.
5. Traffic sent to the Service is automatically distributed among the matching Pods with the random algorithm.
6. Types of Kubernetes Services:
   1. ClusterIP (Default): Exposes the Service on an internal cluster IP. Used for communication between applications inside the cluster, backend databases, and APIs. It is only accessible within the cluster and provides internal load balancing.
   2. NodePort: Exposes the Service on a static port on every cluster node. Used for accessing applications from outside the cluster for testing or development. It is accessible externally and uses ports typically only in the range 30000–32767. It is suitable for labs and testing. If we do not assign a port, then from the range it is automatically assigned.
   3. LoadBalancer: Exposes the Service externally using a cloud provider's load balancer. Used for production applications that require external access. It automatically provisions an external load balancer and provides a public IP address. They are common in AWS, Azure, and Google Cloud.
   4. ExternalName: Maps a Service to an external DNS name. Used for accessing external services using Kubernetes DNS. There is no proxying. It returns a DNS CNAME record. It is useful for integrating external systems.
7. Types of Service Ports:
   1. port: exposed by the service
   2. targetPort: port on Pod/Container
   3. nodePort: External port used by NodePort Services
   4. If targetPort is not specified, Kubernetes automatically sets it equal to the value of port.
