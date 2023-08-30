# EdgeNet demo | CODECO project
This is an [EdgeNet](https://www.edge-net.org/) demo prepared during the first 6 months of the [CODECO project](https://he-codeco.eu/)

We will deploy two demo services described in the attached [``.yaml`` files](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) to test some basic functionalities of EdgeNet such as the [Selective Deployment](https://github.com/EdgeNet-project/edgenet/blob/main/docs/custom_resources.md#selective-deployment) which allows users to deploy pods onto nodes based on their locations.

## Instructions to run experiments on EdgeNet
The detailed instructions to run experiments on EdgeNet can be found in [EdgeNet-Testbed site](https://www.edge-net.org/pages/running-experiments.html) and the respected [Github](https://github.com/EdgeNet-project/edgenet)

### Prerequisites
To run the following examples you will need:
1. to create an EdgeNet account that you can obtain for free by signing up on the landing app available here [https://www.edge-net.org/pages/running-experiments.html](https://www.edge-net.org/pages/running-experiments.html). More information about the registration can be found there.
2. [``kubectl``](https://kubernetes.io/docs/reference/kubectl/overview/), the [Kubernetes](https://kubernetes.io/) command-line interface.

- [Install kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux)
- [Install kubectl on macOS](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos)
- [Install kubectl on Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows)

3. a *kubeconfig file* - a configuration file - downloaded from the [Download my kubeconfig file](https.//landing.edge-net.org) tab after logging in to EdgeNet. In what follows, we will assume that it is saved in your working directory on your system as *``mycfg.cfg``*.

** Note that this kubeconfig file needs to be replaced every few days in order to have access to EdgeNet.
4. a *namespace* - in which you have access when registered on EdgeNet - to test the examples. In what follows, we will assume that it named as *``mynamespace``*. 
5. the attached ``.yaml`` files - i.e *``cdnserviceexample.yaml``* and *``ping-me-example.yaml``*.


## Deployed services & description
The example ``.yaml`` files:
> *``cdnserviceexample.yaml``*

The *cdnserviceexample* describes a simple cdn service - a multimedia app - in which EdgeNet selects some nodes from specific geographic locations as we defined with a selector in the ``.yaml`` file to run our service. We can connect to these pods through the CL.

> *``ping-me-example.yaml``*

In the *ping-me-example* EdgeNet selects some nodes from specific geographic locations as we defined with a selector in the ``.yaml`` file which ping a server or ip address. We can observe the log output of the pods pinging a server or ip address and the delay. If we have access to the server or ip address we can also observe through traffic control (e.g. [``tcpdump``](https://www.tcpdump.org/index.html#latest-releases) the received pings from all over the globe. This example highlights the possible use of EdgeNet as a benchmarking tool for the CODECO usecases i.e. setting up clients (around the globe) to test the provided services.


## General commands
> Get a list of the contributed EdgeNet nodes
~~~~
kubectl get nodes -o wide --kubeconfig <mycfg.cfg> -n <mynamespace>
~~~~
> Get a detailed description of a contributed EdgeNet node
~~~~
kubectl describe <node_name> --kubeconfig <mycfg.cfg> -n <mynamespace>
~~~~

## Deployment & Check
+ *``cdnserviceexample.yaml``*
> Deploy the cdnserviceexample service
~~~~
kubectl apply -f cdnserviceexample.yaml --kubeconfig <mycfg.cfg> -n <mynamespace>
# selectivedeployment.apps.edgenet.io/cdn-deployment created
# service/cdn-service created
~~~~
A Selective Deployment datastructure and a Service are created which let us deploy our service based on the selector we defined in the ``.yaml`` file.
> Check the Selective Deployment, Deployments, Services, Pods
~~~~
kubectl get sd --kubeconfig <mycfg.cfg> -n <mynamespace>
# NAME                         READY   STATUS    AGE
# cdn-deployment               1/1     Running   3s
# ping-me-example              1/1     Running   3m14s
~~~~
~~~~
kubectl get deployments --kubeconfig <mycfg.cfg> -n <mynamespace>
~~~~
~~~~
kubectl get services --kubeconfig <mycfg.cfg> -n <mynamespace>
~~~~
~~~~
kubectl get pods --kubeconfig <mycfg.cfg> -n <mynamespace>
~~~~

> Connect to a pod running our service with CL
~~~~
kubectl exec -it <cdn-server-podname> bash --kubeconfig <mycfg.cfg> -n <mynamespace>
~~~~
> Connect a local port to a port of a pod
~~~~
kubectl port-forward <cdn-server-podname> 8080:80 --kubeconfig <mycfg.cfg> -n <mynamespace>
~~~~


---
+ *``ping-me-example.yaml``*
> Deploy the ping-me-example service
~~~~
kubectl apply -f ping-me-example.yaml --kubeconfig <mycfg.cfg> -n <mynamespace>
# selectivedeployment.apps.edgenet.io/ping-me-example created
~~~~
A Selective Deployment datastructure is created which let us deploy our service based on the selector we defined in the ``.yaml`` file.
> Check the Selective Deployment, Deployments, Services, Pods
~~~~
kubectl get sd --kubeconfig <mycfg.cfg> -n <mynamespace>
# NAME                         READY   STATUS    AGE
# ping-me-example              1/1     Running   14s
~~~~
~~~~
> Get the log output of a pod
~~~~
kubectl logs <ping-source-podname> --kubeconfig <mycfg.cfg> -n <mynamespace>
~~~~
> If server or ip address is accessible, observe the pings from the pods through tcpdump
~~~~
sudo tcpdump -i <eth0> icmp
~~~~

## Delete the services
~~~~
kubectl delete -f cdnserviceexample.yaml --kubeconfig <mycfg.cfg> -n <mynamespace>
# selectivedeployment.apps.edgenet.io "cdn-deployment" deleted
# service "cdn-service" deleted
~~~~
~~~~
kubectl delete -f ping-me-example.yaml --kubeconfig <mycfg.cfg> -n <mynamespace>
# selectivedeployment.apps.edgenet.io "ping-me-example" deleted
~~~~

The following video represents a demo of the deployed ``.yaml`` files
https://github.com/gkoukis/MyTest/assets/127508084/942e05ad-2af0-484e-a80b-f984d562562d
