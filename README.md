```bash
 _          _                          _
| | ___   _| |__   ___ _ __ _ __   ___| |_ ___  ___
| |/ / | | | '_ \ / _ \ '__| '_ \ / _ \ __/ _ \/ __|
|   <| |_| | |_) |  __/ |  | | | |  __/ ||  __/\__ \
|_|\_\\__,_|_.__/ \___|_|  |_| |_|\___|\__\___||___/

```

Table of contents
=================

<!--ts-->
0. [Prerequisites](#prerequisites)
1. Into Kubernetes
  * [What is Kubernetes](#what-is-kubernetes)
  * [Why Kubernetes](#why-kubernetes)
  * [Kubernetes or Swarm](#kubernetes-or-swarm)
2. Setup & First pods
  * [Basic Terms](#basic-terms)
  * [Local Setup](#local-setup)
    * [MicroK8s](#microk8s)
  * [Kubernetes Container Abstractions](#kubernetes-container-abstractions)
  * [Kubernetes Run Create and Apply](#kubernetes-run-create-and-apply)
  * [Creating Pods with kubectl](#creating-pods-with-kubectl)
  * [Inspecting Kubernetes Objects](#inspecting-kubernetes-objects)
3. Exposing Kubernetes Ports
  * [kubectl expose](#kubectl-expose)
  * [Basic Service Types](#basic-service-types)
    * [Creating a ClusterIP Service](#creating-a-clusterip-service)
    * [Creating NodePort](#creating-nodeport)
    * [Creating LoadBalancer](#creating-loadbalancer)
    * [Services DNS](#services-dns)
4. Managment techniques
  * [Run create and Expose generators](#run-create-and-expose-generators)
  * [Generator Examples](#generator-examples)
5. Moving to Declarative YAML
  * [kubectl apply](#kubectl-apply)
  * [Kubernetes Configuration YAML](#kubernetes-configuration-yaml)
  * [Building YAML files](#building-yaml-files)
  * [Building YAML Spec](#building-yaml-spec)
  * [Dry runs and diffs](#dry-runs-and-diffs)
  * [Labels and Label Selectors](#labels-and-label-selectors)

<!--te-->

Prerequisites
=============
- Docker Installed
- kubectl Installed, The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters

What is Kubernetes
==================
 - Kubernetes = popular container orchestrator
 - Container Orchestration = Make many servers act like one
 - Released by Google in 2015, maintained by large community
 - Runs on top of Docker (usually) as a set of APIs in containers
 - Providers API/CLI to manage containers across servers

[🔝](#table-of-contents)

Why Kubernetes
==============
 - Orchestration: Next logical step in journey to faster DevOps
 - First, understand why you *may* need orchestration
 - Not every solution needs orchestration
 - Servers + Change Rate = Benefit of orchestration
 - If Kubernetes, decide which distribution
     - cloud or self-managed (Docker Enterprise, Rancher, OpenShift, Canonical, VMWare PKS)

[🔝](#table-of-contents)

Kubernetes or Swarm
===================
 - Kubernetes and Swarm are both container orchestrators
 - Both are solid platforms with vendor backing
 - Swarm: Easier to deploy/manage
 - Kubernetes: More features and flexibility
 - Understand both and know your requirements


| Advantages of Swarm | Advantages of Kubernetes |
| ------------------- | ------------------------ |
| Comes with Docker, single vendor container platform | test | Clouds will deploy/manage Kubernetes for you |
| Easiest orchestrator to deploy/manage yourself | Infrastructure vendos are making their own distros |
| Follows 80/20 rule, 20% of features for 80% of use cases | Widest adoption and community |
| Runs anywhere Docker does: local, cloud, datacenter, ARM, Windows, 32-bit | Flexible: Covers widest set of use cases |
| Secure by default | "Kubernetes first" vendor support |
| Easier to troubleshoot | |

[🔝](#table-of-contents)

Basic Terms
============
 - Kubernetes: The whole orchestration system
     - K8s "k-eights" or Kube for short
 - Kubectl: CLI to configure Kubernetes and manage apps
     - Using "cube control" official pronunciation
 - Node: Single server in the Kubernetes cluster (workers)
 - Kubelet: Kubernetes agent running on nodes
 - Control Plane: Set of containers that manage the cluster
     - Includes API server, scheduler, controller manager, etcd, and more..
     - Sometimes called the "master"
![](imgs/kubernetes-cluster.svg)

[🔝](#table-of-contents)

Local Setup
============

MicroK8s
--------
1. Install `microk8s`
```bash
snap install microk8s --classic
```
2. Add the user to the 'microk8s' group
```bash
sudo usermod -a -G microk8s <USER>
sudo chown -f -R <USER> ~/.kube
```
3. Create an alias in your `.zshrc` or `.bashrc`
```bash
alias "kubectl"="microk8s kubectl"
```
4. Reload the user groups either via a reboot or by running 'newgrp microk8s'.

[🔝](#table-of-contents)

Kubernetes Container Abstractions
=================================
 - Pod: One or more containers running together on one Node
     - Basic unit of deployment. Containers are always in pods
 - Controller: For creating/updating pods and other objects
     - Many types of Controllers inc Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, Cronjob, etc.
 - Service: network endpoint to connect to a pod
 - Name space: filtered group of objects in cluster
 - Secrest, COnfigMaps and more

Kubernetes Run Create and Apply
===============================
 - Kubernetes is evolving, and so is the CLI
 - We get three ways to create pods from the kubectl CLI
    - kubectl run (changing to be only for pod creation, simular to `docker run`)
    - kubectl create (create some resources via CLI or YAML, simular to `docker swarm create`)
    - kubectl apply (create/update anything via YAML, simular to `docker stack deploy`)

[🔝](#table-of-contents)

Creating pods with kubectl
==========================
 - Two ways to deploy Pods (containers): commands or YAML

Creating a pod named `serv01` using the `nginx` image
```bash
kubectl create deployment serv01 --image nginx
```

Scaling ReplicaSets
```bash
kubectl scale deployment/serv01 --replicas 3
```

Inspecting Kubernetes Objects
-----------------------------

```bash
kubectl get pods                                    # Get list the pods
kubectl logs deployment/serv01                      # Displaying the full log of the pod
kubectl logs deployment/serv01 --follow --tail 1    # Watch the log and it will display the last line
kubectl logs -l run=serv01                          # ?????????????
kubectl get pods -w                                 # -w stands for 'watch', simular to docker stats
```

[🔝](#table-of-contents)

kubectl expose
=========================

- `kubectl expose` creates a **service** for existing pods
- A **service** is a stable address for pods
- If we want to connect to pods, we need a service
- CoreDNS allows us to resolve **services** by name
- There are different types of **services**


Basic Service Types
-------------------

- **ClusterIP** (default)
  - Single, internal virtual IP allocated
  - Only reachable from within cluster (nodes and pods)
  - Pods can reach service on apps port number
- **NodePort**
  - High port allocated on each node
  - Port is open on every node's IP
  - Anyone can connect (if they reach node)
  - Other pods need to be updated to this port
*These services are always available in kubernetes*
- LoadBalancer (mostly used in the cloud)
  - Controls a LB endpoint external to the cluster
  - Only available when infra provider gives you a LB (AWS ELB, etc)
  - Creates NodePort+ClusterIP services, tells LB to send to NodePort
*This is for traffic coming into your cluster from an external source and it requires an infrastucture provider that gives k8s the abiity to talk to it remotely.*
- ExternalName
  - Adds CNAME DNS record to CoreDNS only
  - Not used for Pods, but for giving pods a DNS name to use for something outsite kubernetes
- Kubernetes Ingress [?]

Creating a ClusterIP Service
----------------------------

```bash
kubectl create deployment/httpenv --image bretfisher/httpenv
```

Scaling the pod:
```bash
kubectl scale deployment/httpenv --replicas 5
```

Expoising a port:
```bash
kubectl expose deployment/httpenv --port 8888
```

Now if you check on services, you will see that `httpenv`'s type is `ClusterIP`. 
*ClusterIP is only usable inside the cluster for nodes and other pods to be able to access it.*


kubectl run tmp-shell --rm -i --tty --image=bretfisher/netshoot -- zsh

https://github.com/nicolaka/netshoot

[🔝](#table-of-contents)

Creating NodePort
-----------------

```bash
kubectl expose deployment/httpenv --port 8888 --name httpenv-lb --type NodePort
```

Ports:  
```bash
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.152.183.1    <none>        443/TCP          24h
service/httpenv      ClusterIP   10.152.183.13   <none>        8888/TCP         48m
service/httpenv-np   NodePort    10.152.183.88   <none>        8888:32396/TCP   7s
```

|8888|32396|
| --- | --- |
| port inside the cluster | node port exposed to the outside world. <br /> default node port range 30000-32767 |

[🔝](#table-of-contents)

Creating LoadBalancer
---------------------

```bash
kubectl expose deployment/httpenv --port 8888 --name httpenv-np --type LoadBalancer
```

Clearning up:
```bash
kubectl delete service/httpenv service/httpenv-np service/httpenv-lb
```
[🔝](#table-of-contents)

Services DNS
------------
- Starting with 1.11, internal DNS is provided by CoreDNS
- Like Swarm, this is DNS-Based Service Directory
- So far we've been using hostnames to access Services
 - `curl <hostname>`
- But that only works for Services in the same Namespace
 - `kubectl get namespaces`
- Services also have a FQDN
  - `curl <hostname>.<namespace>.svc.cluster.local`

[🔝](#table-of-contents)

Managment Techniques  
====================

Run create and Expose generators
--------------------------------

- These commands use helper templates called "generators"
- Every resource in Kubernetes has a specification or "spec"
  > `kubectl create deployment sample --image nginx --dry-run=client -o yaml`[^1]
[^1:] https://stackoverflow.com/questions/64279343/kubectl-dry-run-is-deprecated-and-can-be-replaced-with-dry-run-client
- You can output those templated with `--dry-run -o yaml`
- You can use those YAML defaults as a starting point
- Generators are "opinionated defaults"

[🔝](#table-of-contents)

Generator Examples
------------------
```bash
> kubectl create deployment test --image nginx --dry-run=client -o yaml
> kubectl create job test --image nginx --dry-run=client -o yaml

> kubectl expose deployment/test --port 80 --dry-run=client -o yaml
  - You need the deployment to exist before this works.
```

[🔝](#table-of-contents)

Imperative vs Declarative
-------------------------

- "Imperative" is a command - like "create 42 widgets".
- "Declarative" is a statement of the desired end result - Like "I want 42 widgets to exist".

Typically, your yaml file will be declarative in nature: it will say that you want 42 widgets to exist. You'll give that to Kubernetes, and it will execute the steps necessary to end up with having 42 widgets.

"Create" is itself an imperative command, but what you're creating is a Kubernetes cluster. What the cluster should look like is determined by the declarations in the yaml file.

[Read more..](https://medium.com/payscale-tech/imperative-vs-declarative-a-kubernetes-tutorial-4be66c5d8914)

[🔝](#table-of-contents)

Three Managment Approches
-------------------------
- Imperative commands: `run`, `expose`, `scale`, `edit`, `create deployment`
  - Best for dev/learning/personal projects
  - Easy to learn, hardest to manage over time
- Imperative objects: `create -f file.yml`, `replace -f file.yml`, `delete`...
  - Good for prod of small environments, single file per command
  - Store your changes in git based yaml files
  - Hard to automate
- Declarative objects: `apply -f file.yml` or `dir\`, `diff`
  - Best for prod, easier to automate
  - Harder to understand and predict changes

Declarative Kubernetes YAML
===========================

kubectl apply
-------------
- create/update resources in a file </br>
```kubectl apply -f nginx.yaml```
- create/update a whole directory of yaml </br>
```kubectl apply -f ~/path-to-dir/```
- create/update from a URL </br>
```kubectl apply -f https://raw.githubusercontent.com/x0rCTF/test3nity/main/nginx.yml```


[🔝](#table-of-contents)


Kubernetes Configuration YAML
-----------------------------
- Each file contains one or more manifests
- Each manifest describes an API object (deployment, job, secret)
- Each manifest needs four parts (root key:values in the file)
```bash
apiVersion:       # k api-versions 
kind:             # k api-resources -o wide | https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/
metadata:         # https://kubernetes.io/docs/reference/kubernetes-api/common-definitions/object-meta/#ObjectMeta
spec:             # What state you desire for the object
```

Building YAML files
-------------------





[kubectl commands with examples](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create)
[🔝](#table-of-contents)
