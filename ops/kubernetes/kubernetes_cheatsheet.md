# Kubernetes

## Terminology

1. Node

A node is a physical or virtual machine. It is **not created by Kubernetes**. You create those with a cloud operating system, like OpenStack or Amazon EC2, or manually install them. So you need to lay down your basic infrastructure before you use Kubernetes to deploy your apps. But from that point it can define virtual networks, storage, etc. For example, you could use OpenStack Neutron or Romana to define networks and push those out from Kubernetes.

Each Node is managed by the Master. A Node can have multiple pods, and the Kubernetes master automatically handles scheduling the pods across the Nodes in the cluster. The Master's automatic scheduling takes into account the available resources on each Node.

Every Kubernetes Node runs at least:

- Kubelet, a process responsible for communication between the Kubernetes Master and the Node; it manages the Pods and the containers running on a machine.
- A container runtime (like Docker, rkt) responsible for pulling the container image from a registry, unpacking the container, and running the application

2. Pods

Pods are the atomic unit on the Kubernetes platform. A pod is one or more containers that logically go together and are relatively tightly coupled. Containers within pod have same shared resources, for example:

- Shared storage, such as Volumes
- Networking, such as unique cluster IP address
- Information about how to run each container, such as the container image version or specific ports to use

Pods run on nodes. They all share IP address and port space together and can reach each other via localhost. They are also co-located and co-scheduled. But they do not need to run on the same machine as containers can span more than one machine. One node can run multiple pods.

When we create a Deployment on Kubernetes, that Deployment creates Pods with containers inside them (as opposed to creating containers directly). Each Pod is tied to the Node where it is scheduled, and remains there until termination (according to restart policy) or deletion. In case of a Node failure, identical Pods are scheduled on other available Nodes in the cluster

Pods are cloud-aware. For example you could spin up two Nginx instances and assign them a public IP address on the Google Compute Engine (GCE). To do that you would start the Kubernetes cluster, configure the connection to GCE, and then type something like:

```
kubectl expose deployment my-nginx –port=80 –type=LoadBalancer
```

3. Deployment

A set of pods is a deployment. A deployment ensures that a sufficient number of pods are running at one time to service the app and shuts down those pods that are not needed. It can do this by looking at, for example, CPU utilization.

4. Service

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods. A Service is defined using YAML (preferred) or JSON, like all Kubernetes objects. The set of Pods targeted by a Service is usually determined by a LabelSelector (see below for why you might want a Service without including selector in the spec).

## Basic Usages

Common command pattern:

```
kubectl <action> <resource>
```

Common actions:

```
kubectl version
kubectl get <resource>
kubectl describe <resource>
kubectl logs <resource>
kubectl exec <resource>
```

Deploying a container:

```
kubectl get nodes
kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080
```

Pods that are running inside Kubernetes are running on a private, isolated network. By default they are visible from other pods and services within the same kubernetes cluster, but not outside that network.

```
kubectl proxy

export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME

curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/
```

Set labels

```
kubectl label pod $POD_NAME app=v1
```

Create a new service and expose to external traffic

```
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080

export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT
```

Delete existing service

```
kubectl delete service -l run=kubernetes-bootcamp
```

Scale up/down

```
kubectl scale deployments/kubernetes-bootcamp --replicas=4
kubectl get pods -o wide
```

To update the image of the application to version 2, use the set image command, followed by the deployment name and the new image version

```
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
```

Check status of deployment rollout

```
kubectl rollout status deployments/kubernetes-bootcamp
```

Undo the rollout process

```
kubectl rollout undo deployments/kubernetes-bootcamp
```
