# Deploying EdgeNet features/CRDs within a local Kubernetes cluster environment
In the [CODECO_edgenet](https://github.com/swnuom/edgenet) and [CODECO_edgenet_node](https://github.com/swnuom/node) we have forked the code from the official EdgeNet repository and contributed with minor bug fixes and improvements to enhance the overall functionality of the infrastructure as described in [here](https://github.com/EdgeNet-project/edgenet/blob/main/docs/README.md#components) and [here](https://github.com/EdgeNet-project/edgenet/blob/main/docs/tutorials/deploy_edgenet_to_kube.md). In particular, we explore the **Multiprovider** (i.e. contribute a node into an EdgeNet cluster) and **Multitenancy** (enabling the utilization of a shared cluster such as tenant and role requests, slice and subnamespaces) features.

We offer examples and commands to illustrate the utilization of these features within a K8s cluster, which comprises one master node (referred to as 'athm3') and one worker node (referred to as 'athw7'), both running Ubuntu 22.04.2 LTS. Our Kubernetes cluster has been configured using Ansible playbooks to integrate Edgenet.


## Create local K8s - EdgeNet cluster
**On the master node (athm3)** we have cloned the forked version of[CODECO_edgenet](https://github.com/swnuom/edgenet). Additionally, within the repository, there is a _vpnpeer.yaml_ file, which serves the purpose of maintaining the Local Area Network (LAN) connectivity among the nodes within the cluster.

> Check the vpnpeer.yaml on master (athm3)
```bash
cat vpnpeer.yaml
#apiVersion: networking.edgenet.io/v1alpha1
#kind: VPNPeer
#metadata:
#  name: athm3 # Replace with the node name
#spec:
#  addressV4: 10.183.5.27 # Replace with the edgenetmesh0 83.212.134.27v4 address
#  addressV6: fdb4:ae86:ec99:4004::27 # Replace with the edgenetmesh0 83.212.134.27v6 address
#  endpointAddress: 83.212.134.27 # Replace with the public 83.212.134.27 address of the node (e.g. use https://ipinfo.io)
#  endpointPort: 51820
#  publicKey: wLX1qYN9vbN82Ch7remhpR9wfdo93bZ2lzTteHe7CAk= # Replace with the result of `echo "private key generated previously" | wg pubkey`
```

> And the vpnpeers
```bash
-kubectl get vpnpeers
#	NAME    ADDRESS-V4    ADDRESS-V6                ENDPOINT        PORT
#	athm3   10.183.5.27   fdb4:ae86:ec99:4004::27   83.212.134.27   51820
```

> On the worker node (athw7) we have a corresponding _vpnpeer.yaml_ file with the same purpose. This files are used to facilitate LAN connectivity among the nodes in the cluster.
```bash
cat vpnpeer.yaml
#apiVersion: networking.edgenet.io/v1alpha1
#kind: VPNPeer
#metadata:
#  name: athw7 # Replace with the node name
#spec:
#  addressV4: 10.183.5.35 # Replace with the edgenetmesh0 83.212.134.35v4 address
#  addressV6: fdb4:ae86:ec99:4004::35 # Replace with the edgenetmesh0 83.212.134.35v6 address
#  endpointAddress: 83.212.134.35 # Replace with the public 83.212.134.35 address of the node (e.g. use https://ipinfo.io)
#  endpointPort: 51820
#  publicKey: 10yf7B87yEjpiKLHISdrkwmGAAhZsTEngwWDZ/Y0bmU= # Replace with the result of `echo "private key generated previously" | wg pubkey`
```

> Back to the master (athm3) we have to copy the vpnpeer.yaml of the worker node (athw7) 
```bash
sudo pico vpnw7.yaml
# copy the content of vpnpeer.yaml of the athw7
```
> And apply the vpnw7
```bash
kubectl apply -f vpnw7.yaml
#	vpnpeer.networking.edgenet.io/athw7 created
```
> We observe that the vpnpeer from worker/athw7 has been created
```bash
-kubectl get vpnpeers
#	NAME    ADDRESS-V4    ADDRESS-V6                ENDPOINT        PORT
#	athm3   10.183.5.27   fdb4:ae86:ec99:4004::27   83.212.134.27   51820
#	athw7   10.183.5.35   fdb4:ae86:ec99:4004::35   83.212.134.35   51820
```
> Finally, we copy the whole public_ehome.cfg file found in the .kube/ directory into the edgenet/configs/public_ehome.cfg (https://github.com/swnuom/edgenet/blob/main/configs/public_ehome.cfg))
```bash
cat .kube/config-public
```
```bash
sudo pico edgenet/configs/public_ehome.cfg
```

## Multiprovider

To contribute a node, you can follow these simple steps: Begin by establishing an SSH connection to the node you intend to contribute, such as the one named _kubem2_.

> We clone the [EdgeNet / node repo](https://github.com/swnuom/node) in the node (kubem2). 
```bash
git clone https://github.com/swnuom/node.git
```

> Now we need to copy the publig key of the master node (athm3) into the contributed node (kubem2) so any node we want to contribute, it can join the cluster.

> We find and copy the public key of the master node (athm3) found in the ssh folder of the master node (athm3)
```bash
cat .ssh/id_rsa.pub 
#	ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDbKY2yoZPfnwIVHAEtSZpIE6E2+n+SeQ/yvVml/Ix+5NxpLX/X9Z8QLb0g3/KPvVF1qWRg5Xc4odSXJBFoNP8D73i/ODFkx/BeOMRpR3QfeF6MsCWZ6Yud9zFuz+PvdglUcK2uAiu1eYzbNf0iNl70dEKJhu6OGvuKddkUWs6394JiBLnu7v0tRhMW9MEF3lyRPAhIuwM0vOk3mYtbMTzEE+fsAkUGqn/HliRFdnQ9SROlOIotyOXpqPmKmznp7XMoDMIF1XM/34iIGgS0QgPL45g7H0umtOijq8QgS1SqL/FmluJSVmOb/eDIsm3n8VU1il+2Z3r3+6RuNmU+6C9SX/PWmlvsrvdzGXGE0ZuksyXTrPnA8K0hM7Zp0cMaBMD3w+NzkH2jbNab9YSypxPujm/qXHz9l+FnRwMoJaMqWhTizr8r1ttwzK6Mxlm1TO+eluCndesDnsT2wAFJ1PWx2Qw0PdMbtht3UR9y5wF4gZIJNqJp8VdAP21rqlorSzocMloXGDrSLj89RvLCp9Oy0mH2kc9U9Mjr4fP2MI5ZGzO2E0JepDkcBohpraEBF5mKPkPnAvQjhha3WCzxZj/90p86QqekbuZxHRBU52vzpC9pLPGfTrATXzypZmDLkMQ+iWWA+hZXUYWX52e44csZy+IWnruepZRLFbNFSQQOrw== ansible-generated on athm3
```
> And we replace the last line of code found in the contributed node (kubem2), in: _vars/edgenet-codeco.yml_ with the public key of the master node (athm3)
```bash
sudo pico vars/edgenet-codeco.yml
#	# edgenet-kubernetes
#	containerd_version: 1.6.12
#	kubernetes_version: 1.23.17
#	edgenet_node_version: 1.0.17
#	#publicIPv4: 195.251.209.231
#	kubeconfig_url: https://raw.githubusercontent.com/swnuom/edgenet/main/configs/public_ehome.cfg
#	# edgenet-ssh
#	edgenet_ssh_public_key: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDbKY2yoZPfnwIVHAEtSZpIE6E2+n+SeQ/yvVml/Ix+5NxpLX/X9Z8QLb0g3/KPvVF1qWRg5Xc4odSXJBFoNP8D73i/ODFkx/BeOMRpR3QfeF6MsCWZ6Yud9zFuz+PvdglUcK2uAiu1eYzbNf0iNl70dEKJhu6OGvuKddkUWs6394JiBLnu7v0tRhMW9MEF3lyRPAhIuwM0vOk3mYtbMTzEE+fsAkUGqn/HliRFdnQ9SROlOIotyOXpqPmKmznp7XMoDMIF1XM/34iIGgS0QgPL45g7H0umtOijq8QgS1SqL/FmluJSVmOb/eDIsm3n8VU1il+2Z3r3+6RuNmU+6C9SX/PWmlvsrvdzGXGE0ZuksyXTrPnA8K0hM7Zp0cMaBMD3w+NzkH2jbNab9YSypxPujm/qXHz9l+FnRwMoJaMqWhTizr8r1ttwzK6Mxlm1TO+eluCndesDnsT2wAFJ1PWx2Qw0PdMbtht3UR9y5wF4gZIJNqJp8VdAP21rqlorSzocMloXGDrSLj89RvLCp9Oy0mH2kc9U9Mjr4fP2MI5ZGzO2E0JepDkcBohpraEBF5mKPkPnAvQjhha3WCzxZj/90p86QqekbuZxHRBU52vzpC9pLPGfTrATXzypZmDLkMQ+iWWA+hZXUYWX52e44csZy+IWnruepZRLFbNFSQQOrw== ansible-generated on athm3
```
> Finally, we run the start script
```bash
cd node/
./start.sh
```

> We can repeat the process in the Multiprovider section to contribute as many nodes as we want
> Now if we SSH to our master node (athm3) we can observe the contributed node joining the cluster
```bash
kubectl get nodes -o wide
#	NAME                    STATUS   ROLES                  AGE     VERSION    INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
#	athm3                   Ready    control-plane,master   28h     v1.23.17   83.212.134.27     <none>        Ubuntu 22.04.2 LTS   5.15.0-71-generic   containerd://1.6.24
#	athw7                   Ready    <none>                 28h     v1.23.17   83.212.134.35     <none>        Ubuntu 22.04.2 LTS   5.15.0-71-generic   containerd://1.6.24
#	gr-a-7230.edge-net.io   Ready    <none>                 15m     v1.23.17   83.212.134.22     <none>        Ubuntu 22.04.2 LTS   5.15.0-86-generic   containerd://1.5.11
#	gr-b-89c3.edge-net.io   Ready    <none>                 108s    v1.23.17   195.251.209.229   <none>        Ubuntu 22.04.2 LTS   5.15.0-71-generic   containerd://1.5.11
#	gr-b-abf4.edge-net.io   Ready    <none>                 3m26s   v1.23.17   195.251.209.231   <none>        Ubuntu 22.04.2 LTS   5.15.0-71-generic   containerd://1.5.11
```
> And observe the contribution nodes
```bash
kubectl get nodecontributions
#	NAME        ADDRESS         PORT   ENABLED   STATUS          AGE
#	gr-a-7230   83.212.134.22     22     true      Node Accessed   16m
#	gr-b-89c3   195.251.209.229   22     true      Node Accessed   2m34s
#	gr-b-abf4   195.251.209.231   22     true      Node Accessed   4m13s
```

[![asciicast](https://asciinema.org/a/uDQ6Kk152u51uzyrh5b4HLjqD.svg)](https://asciinema.org/a/uDQ6Kk152u51uzyrh5b4HLjqD)

## Multitenancy

>On the master node (athm3) we can access the resource as admins 
```bash
kubectl get pods
#	No resources found in default namespace.
kubectl get tenantrequest
#	No resources found
```
> These rights are accessed through a configuration files (_config_) found in .kube/ directory
```bash
ls .kube/
#	cache  config  config-public
cat .kube/config
#   ...
```

> From the master node (athm3) with the _create-user.sh_ script we can create a username whose name is the provided email (_user@gmail.com_) and also create the respective namespace (_uom_).
```bash
./create-user.sh
#	creating namespace
#	namespace/uom created
#	creating .certs folder in home directory
#	creating private key for user
#	creating certificate sign request
#	certificatesigningrequest.certificates.k8s.io/user@gmail.com created
#	approving sign request
#	certificatesigningrequest.certificates.k8s.io/user@gmail.com approved
#	exporting approved user certification
#	creating and applying edgenet tenantrequest rolebinding
#	clusterrolebinding.rbac.authorization.k8s.io/user@gmail.com-registration created
#	creating config file for user
#	Cluster "edgenet" set.
#	User "user@gmail.com" set.
#	Context "user@gmail.com-context" created.
#	Switched to context "user@gmail.com-context".
#	testing user
#	No resources found
```
> Again we can observe the configuration files which now include a new configuration file with the name of the user's email and the "limited" rights of a user.
```bash
ls .kube/
#	cache  config  config-user@gmail.com  config-public
cat .kube/config
#   ...
```
> If we try to observe the resources (e.g. pods and tenantrequest) from the user perspective, we cannot do that due to rights limitation
```bash
kubectl get pods --kubeconfig config-user@gmail.com -n uom
#	Error from server (Forbidden): pods is forbidden: User "user@gmail.com" cannot list resource "pods" in API group "" in the namespace "uom"
kubectl get tenantrequest/uom --kubeconfig config-user@gmail.com -n uom
#	Error from server (Forbidden): tenantrequests.registration.edgenet.io "uom" is forbidden: User "user@gmail.com" cannot get resource "tenantrequests" in API group "registration.edgenet.io" at the cluster scope
```
> As user we will ask for tenant rights with the _tenantrequest.yaml_
```bash
kubectl create -f tenantrequest.yaml --kubeconfig config-user@gmail.com -n uom
#	tenantrequest.registration.edgenet.io/uom created
```
> We can observe the tenantrequest in pending status
```bash
kubectl get tenantrequest/uom
#	NAME   OFFICIAL NAME             SHORT NAME   URL                 CITY           COUNTRY   EXPIRY                 STATUS    AGE
#	uom    University of Macedonia   uom          http://www.uom.gr   Thessaloniki   Greece    2023-10-17T21:35:57Z   Pending   103s
```
> As admins we **approve** the tenant request by adding the "_approved: true_" under the "_spec_" field
```bash
kubectl edit tenantrequest/uom --kubeconfig config
#	tenantrequest.registration.edgenet.io/uom edited
```
> Which can be observed here
```bash
kubectl get tenantrequest/uom --kubeconfig config
#	NAME   OFFICIAL NAME             SHORT NAME   URL                 CITY           COUNTRY   EXPIRY                 STATUS    AGE
#	uom    University of Macedonia   uom          http://www.uom.gr   Thessaloniki   Greece    2023-10-17T21:35:57Z   Created   17m
```
> Now as a user/tenant-owner we have the credentials to observe the resources like rolebindings, pods etc
```bash
kubectl get rolebindings --kubeconfig config-user@gmail.com
#	No resources found in default namespace.
kubectl get pods --kubeconfig config-user@gmail.com
#	No resources found in uom namespace.
```
"We can apply services e.g. create a simple nginx app in the uom namespace"
```bash
kubectl apply -f nginx-uom.yaml --kubeconfig config-user@gmail.com -n uom
#	service/nginx-service created
#	deployment.apps/nginx-deployment created
```
```bash
kubectl get pods -o wide --kubeconfig config-user@gmail.com -n uom
#	NAME                                READY   STATUS    RESTARTS   AGE    IP            NODE    NOMINATED NODE   READINESS GATES
#	nginx-deployment-56f68b846d-95ql7   1/1     Running   0          107s   10.244.1.9    athw7   <none>           <none>
#	nginx-deployment-56f68b846d-f5xvf   1/1     Running   0          107s   195.251.209.229    gr-b-89c3.edge-net.io   <none>           <none>
#	nginx-deployment-56f68b846d-hj8df   1/1     Running   0          107s   195.251.209.231   gr-b-abf4.edge-net.io   <none>  
```
```bash
kubectl get services -o wide
#	NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE     SELECTOR
#	nginx-service   ClusterIP   10.109.130.222   <none>        80/TCP    2m40s   app=nginx
```
> If we curl the cluster-IP of the nginx-service we get the output indicating **inter-pod connectivity** which was not possible through the EdgeNet test-bed.
```bash
curl 10.109.130.222
#   ...
```

[![asciicast](https://asciinema.org/a/ASLdFO7kzjdzFJOLfpQMK3OBs.svg)](https://asciinema.org/a/ASLdFO7kzjdzFJOLfpQMK3OBs)

> With the nessesary changes we can run the demo examples provided in [here](https://gitlab.eclipse.org/eclipse-research-labs/codeco-project/experimentation-framework-and-demonstrations/edgenet-framework/-/tree/main/demo?ref_type=heads)
