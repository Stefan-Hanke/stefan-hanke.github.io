---
layout: post
title:  "For me, the best k8s talk"
date:   2020-05-10
categories: k8s yt aws
---

For me, the very best talk about k8s on yt. This talk _will_ stress your mental capacity, and I'm glad I'm not sitting in the audience, breathing bad air, maybe sweating, and trying to ingest all that information.

This post serves as a personal refresher to k8s terms. It covers basically to 19 mins 45 secs into the talk.

The talk I'm talking about is [Kubernetes Deconstructed: Understanding Kubernetes by Breaking It Down - Carson Anderson, DOMO](https://www.youtube.com/watch?v=90kZRyPcRZw). Actually, I just found out that there is an [unabriged version of the talk](https://vimeo.com/245778144/4d1d597c5e). However, I like the YT version more since it is before a real audience.

----
----

## k8s terms

Working set - need to know:

- **pod**: unit of deployment for k8s
- **deployment**: describes pods, replicas, and metadata to be deployed.
- [**service**](https://kubernetes.io/docs/concepts/services-networking/service/): defines an cluster IP and DNS name, forwards requests to selected pods.
- **ingress**: Defines how to get from outside the cluster to inside the cluster.

Occipital set - good to know:

- **endpoints**: _They're actually kind-of the routing meat of k8s_

----
----

## k8s programs and their relationships

#### running on master nodes

A k8s server is using [etcd](https://etcd.io/) for persisting, syncronizing, and watching data between all master nodes. It seems like a perfect fit for this task.

- **kube-apiserver**. Provides the API server, persistence and watching support via etcd (see above). Doing an `kubectl apply` ends up here.

- **kube-scheduler**. _Maitre D_. Gets notified by the api server about changes, schedules pod to worker nodes.

- **kube-controller-manager**. Knows what k8s objects _exactly_ to create for e.g. a deployment (replica-set, serviceaccount, secret, ...). Manages loads of sub-controllers, e.g. _replicaset-controller_, _deployment-controller_ and so on.

#### running on worker nodes

- **kubelet**. Hooks up to the api server, makes pods real. Is using a _container runtime_ (e.g. docker, [rkt](https://coreos.com/rkt/)). k8s has abstracted this via the [CRI](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/), the _container runtime interface_. Performs liveness probes and readyness checks.

- **kube-proxy**. Hooks up to the api server, makes services real on nodes inside the cluster.

----
----

## k8s networking

#### useful links

- [The Kubernetes network model](https://kubernetes.io/docs/concepts/cluster-administration/networking/#the-kubernetes-network-model)
- Note that everything that has an IP inside the cluster can also be [reached via DNS names](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/).
- [Very good looking in-depth explanation of kube-proxy](http://arthurchiao.art/blog/cracking-k8s-node-proxy/).

#### facts about IP networking

- every pod has an cluster-unique IP
- every node has an cluster-unique IP
- every node has a CIDR range. every pod on that node must have an IP inside this range.

#### facts about networking

- all containers can communicate with all other containers without NAT
- all nodes can communicate with all containers (and vice-versa) without NAT
- the IP that a container sees itself as is the same IP that others see it as.

#### services in-depth

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

A service can be of type:

- ClusterIP (default): assign a static, unique IP to the service. Other pods can use this cluster IP to talk to the selected pods, even when these are reshuffled on the nodes.

```yaml
spec:
  type: ClusterIP
  clusterIP: 10.0.171.239
```

For the implementation side, by default ClusterIP addresses are virtual - they serve as target IP for iptables rules (using the default kube-proxy).

- NodePort: also a ClusterIP, assigns a TCP port to all nodes, reachable from the outside.

> Using a NodePort gives you the freedom to set up your own load balancing solution, to configure environments that are not fully supported by Kubernetes, or even to just expose one or more nodes’ IPs directly.

```yaml
spec:
  type: NodePort
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```

- LoadBalancer: also a NodePort, implemented by k8s providers like AWS, GCR etc. This basically declares that there is an external load balancer.

----
----

## Follow-Ups

 - Example of a custom resource definition and what it needs to make it real in k8s?

----
----

## post scriptum

 - Jekyll aggregates tags on a post in the final URL like in 

    > http://127.0.0.1:4000/k8s/yt/aws/2020/05/10/for-me-best-k8s-talk.html

    (notice the k8s, yt, aws text). Why?