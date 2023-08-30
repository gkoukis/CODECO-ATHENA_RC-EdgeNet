# EdgeNet demo | CODECO project
This is an [EdgeNet](https://www.edge-net.org/) demo prepared by the [ATHENA-RC](https://www.athenarc.gr/en/home) team during the first 6 months of the [CODECO project](https://he-codeco.eu/).

We will deploy two demo services described in the attached [``.yaml`` files](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) to test some basic functionalities of EdgeNet such as the [Selective Deployment](https://github.com/EdgeNet-project/edgenet/blob/main/docs/custom_resources.md#selective-deployment) which allows users to deploy pods onto nodes based on their locations.

## Demo video
**The following video represents a demo of the deployed ``.yaml`` files**
https://github.com/gkoukis/MyTest/assets/127508084/942e05ad-2af0-484e-a80b-f984d562562d

## Instructions to run experiments on EdgeNet
We provide instructions of the prerequisites a user needs to run an experiment on EdgeNet and the commands to run the examples. The detailed instructions to run experiments on EdgeNet as well as the provided features can be found in [EdgeNet-Testbed site](https://www.edge-net.org/pages/running-experiments.html) and the respected [Github](https://github.com/EdgeNet-project/edgenet).

Additional tutorials can be found in the [EdgeNet Github](https://github.com/EdgeNet-project/edgenet/tree/main/docs/tutorials/old)

### Prerequisites
To run the following examples you will need:
1. to create an EdgeNet account that you can obtain for free by signing up on the landing app available here [https://www.edge-net.org/pages/running-experiments.html](https://www.edge-net.org/pages/running-experiments.html). More information about the registration can be found there.
2. [``kubectl``](https://kubernetes.io/docs/reference/kubectl/overview/), the [Kubernetes](https://kubernetes.io/) command-line interface.

- [Install kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux)
- [Install kubectl on macOS](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos)
- [Install kubectl on Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows)

3. a *kubeconfig file* - a configuration file - downloaded from the [Download my kubeconfig file](https://landing.edge-net.org) tab after logging in to EdgeNet. In what follows, we will assume that it is saved in your working directory on your system as *``mycfg.cfg``*.

** Note that this kubeconfig file needs to be replaced every few days in order to have access to EdgeNet.


4. a *namespace* - in which you have access when registered on EdgeNet - to test the examples. In what follows, we will assume that it named as *``mynamespace``*. 
5. the attached ``.yaml`` files - i.e *``cdnserviceexample.yaml``* and *``ping-me-example.yaml``*.


## Deployed files & description
The example ``.yaml`` files:
> *``cdnserviceexample.yaml``*

The *cdnserviceexample* describes a simple cdn service - a multimedia app - in which EdgeNet selects some nodes from specific geographic locations as we defined with a selector in the ``.yaml`` file to run our service. We can connect to these pods through the CL.

> *``ping-me-example.yaml``*

In the *ping-me-example* EdgeNet selects some nodes from specific geographic locations as we defined with a selector in the ``.yaml`` file which ping a server or ip address. We can observe the log output of the pods pinging a server or ip address and the delay. If we have access to the server or ip address we can also observe through traffic control (e.g. [``tcpdump``](https://www.tcpdump.org/index.html#latest-releases) the received pings from all over the globe. This example highlights the possible use of EdgeNet as a benchmarking tool for the CODECO usecases i.e. setting up clients (around the globe) to test the provided services.


## General commands
> Get a list of the contributed EdgeNet nodes
```bash
kubectl get nodes -o wide --kubeconfig mycfg.cfg -n mynamespace
```
> Get a detailed description of a contributed EdgeNet node
```bash
kubectl describe <node_name> --kubeconfig mycfg.cfg -n mynamespace
```

## Deployment & Check
+ ***``cdnserviceexample.yaml``***
> Deploy the **cdnserviceexample.yaml** file
```bash
kubectl apply -f cdnserviceexample.yaml --kubeconfig mycfg.cfg -n mynamespace
# selectivedeployment.apps.edgenet.io/cdn-deployment created
# service/cdn-service created
```
A Selective Deployment datastructure and a Service are created which let us deploy our service based on the selector we defined in the ``.yaml`` file.
> Check the Selective Deployment, Deployments, Services, Pods
```bash
kubectl get sd --kubeconfig mycfg.cfg -n mynamespace
# NAME                         READY   STATUS    AGE
# cdn-deployment               1/1     Running   3s
```
```bash
kubectl get deployments --kubeconfig mycfg.cfg -n mynamespace
# NAME                READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS          IMAGES               SELECTOR
# cdn-server          4/4     4            4            1m   cdn-server          emamatas/appserver   app=cdn-server
```
```bash
kubectl get services --kubeconfig mycfg.cfg -n mynamespace
# NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE   SELECTOR
# cdn-service   ClusterIP   10.244.1.215     <none>        80/TCP,8080/TCP   3m   app=cdn-server
```
```bash
kubectl get replicaset --kubeconfig mycfg.cfg -n mynamespace
# NAME                    DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES               SELECTOR
# cdn-server-547d5d4765   4         4         4       3m    cdn-server   emamatas/appserver   app=cdn-server,pod-template-hash=547d5d4765
```
```bash
kubectl get pods --kubeconfig mycfg.cfg -n mynamespace
# NAME                           READY   STATUS   RESTARTS  AGE   IP             NODE
# cdn-server-547d5d4765-967C5    1/1     Running  0         2m6s  10.244.1.14    jp-13-c9fe.edge-net.io
# cdn-server-547d5d4765-d72bz    1/1     Running  0         87s   10.244.72.19   geni-us-ga-035b.edge-net.io
# cdn-server-547d5d4765-fdh4z    1/1     Running  0         76s   10.244.74.22   geni-us-hi-1bf2.edge-net.io
# cdn-server-547d5d4765-vb4x6    1/1     Running  0         2m6s  10.244.8.26    geni-us-oh-90fb.edge-net.io
```

> Access a shell within a pod running our service with CL
```bash
kubectl exec -it <cdn-server-podname> bash --kubeconfig mycfg.cfg -n mynamespace
# root@<cdn-server-podname>:/#
```
> Connect a local port to a port of a pod
```bash
kubectl port-forward <cdn-server-podname> 8080:80 --kubeconfig mycfg.cfg -n mynamespace
# Forwarding from 127.0.0.1:8080 -> 80
# Forwarding from [::1]:8080 -> 80
```


---
+ ***``ping-me-example.yaml``***
> Deploy the **ping-me-example.yaml** file
```bash
kubectl apply -f ping-me-example.yaml --kubeconfig mycfg.cfg -n mynamespace
# selectivedeployment.apps.edgenet.io/ping-me-example created
```
A Selective Deployment datastructure is created which let us deploy our service based on the selector we defined in the ``.yaml`` file.
> Check the Selective Deployment, Deployments, Services, Pods
```bash
kubectl get sd --kubeconfig mycfg.cfg -n mynamespace
# NAME                         READY   STATUS    AGE
# ping-me-example              1/1     Running   14s
```
```bash
kubectl get pods --kubeconfig mycfg.cfg -n mynamespace
# NAME                           READY   STATUS   RESTARTS  AGE   IP             NODE
# ping-source-7lntf              1/1     Running  0         20s   10.244.74.24   geni-us-hi-1bf2.edge-net.io
# ping-source-8rzp7              1/1     Running  0         24s   10.244.5.95    de-ni-5793.edge-net.io
# ping-source-dfn4g              1/1     Running  0         20s   10.244.86.20   geni-us-ca-3c7f.edge-net.io
# ping-source-x9vfc              1/1     Running  0         20s   10.244.2.101   edgenet-ingress.planet-lab.eu
```

> Get the log output of a pod
```bash
kubectl logs <ping-source-podname> --kubeconfig mycfg.cfg -n mynamespace
# PING <IP>: 56 data bytes
# 64 bytes from <IP>: seq=0 ttl=49 time=71.246ms
# ...
```
> If server or ip address is accessible, observe the pings from the pods through tcpdump
```bash
sudo tcpdump -i <eth0> icmp
```

## Check the Description of resources
Use the ``kubectl describe`` command to retrieve detailed information about Kubernetes resources, such as SelectiveDeployments, deployments, pods, services, nodes etc.

**Example:**
```bash
kubectl describe sd/ping-me-example --kubeconfig mycfg.cfg -n mynamespace
# Name:         ping-me-example
# Namespace:    athena-rc
# Labels:       <none>
# Annotations:  <none>
# API Version:  apps.edgenet.io/v1alpha1
# Kind:         SelectiveDeployment
# Metadata:
#  Creation Timestamp: ...
# ...
# Status:
#  Message:  The desired workload(s) are created successfully
#  Ready:    1/1
#  State:    Running
# Events:     <none>
```

## Delete the .yaml file
> Delete the **cdnserviceexample.yaml** file
```bash
kubectl delete -f cdnserviceexample.yaml --kubeconfig mycfg.cfg -n mynamespace
# selectivedeployment.apps.edgenet.io "cdn-deployment" deleted
# service "cdn-service" deleted
```
> Delete the **ping-me-example.yaml** file
```bash
kubectl delete -f ping-me-example.yaml --kubeconfig mycfg.cfg -n mynamespace
# selectivedeployment.apps.edgenet.io "ping-me-example" deleted
```
