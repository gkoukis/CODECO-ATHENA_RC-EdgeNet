# EdgeNet | CODECO project
This repository is prepared by the [ATHENA-RC](https://www.athenarc.gr/en/home) team to demonstrate some [EdgeNet](https://www.edge-net.org/) demo and functionalities for the WP5 of the [CODECO project](https://he-codeco.eu/).

## Demo Folder

The [demo](https://github.com/gkoukis/MyTest/blob/main/demo/) folder showcases demo examples created by our team during the initial 6 months of the CODECO project. Here, one can explore two [``.yaml`` files](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/), the [*``cdnserviceexample.yaml``*](https://github.com/gkoukis/MyTest/blob/main/demo/cdnserviceexample.yaml) and [*``ping-me-example.yaml``*](https://github.com/gkoukis/MyTest/blob/main/demo/ping-me-example.yaml). These files enable users to test fundamental EdgeNet functionalities, such as the [Selective Deployment](https://github.com/EdgeNet-project/edgenet/blob/main/docs/custom_resources.md#selective-deployment) which allows users deploy pods onto nodes based on their locations. In addition, instructions outlining the **prerequisites** for conducting experiments on EdgeNet, along with the requisite **commands** to execute the provided examples, are included. A demonstrative [**video**](https://github.com/gkoukis/MyTest/assets/127508084/942e05ad-2af0-484e-a80b-f984d562562d) featuring the deployment and outcome of these ``.yaml`` files is provided as well to enrich the understanding of the whole process.

## Tutorials Folder

In the [tutorials](https://github.com/gkoukis/MyTest/tree/main/tutorials) folder we have included some ``.yaml`` files designed by EdgeNet to enable the utilization of a shared cluster environment. Within this folder, one can discover various ``.yaml`` files that serve different cluster management purposes. For instance, these files enable [tenants to assign roles per user](https://github.com/gkoukis/MyTest/blob/main/tutorials/role_binding-ath-admin-george.yaml), [allow users to request roles within tenants or subnamespaces](https://github.com/gkoukis/MyTest/blob/main/tutorials/rolerequest-ath-george.yaml) and support the creation of subclusters, [slices](https://github.com/gkoukis/MyTest/blob/main/tutorials/sliceclaim-ath.yaml) and [subnamespaces](https://github.com/gkoukis/MyTest/blob/main/tutorials/subnamespace-Workspace-ath.yaml). For more detailed information, please refer to the EdgeNet repository [here](https://github.com/EdgeNet-project/edgenet/blob/main/docs/README.md#multitenancy) and [here](https://github.com/EdgeNet-project/edgenet/tree/main/docs/tutorials).


Both in the [demo](https://github.com/gkoukis/MyTest/blob/main/demo/) and [tutorials](https://github.com/gkoukis/MyTest/tree/main/tutorials) folders please ensure that you **update the namespace** and the related information to match your own. In our demonstrations the namespace is the *athena-rc*.


## Upcoming Tasks
**[TODO]:** We are currently in the process of testing the deployment of EdgeNet functionalities within our Kubernetes cluster environment as claimed in [here](https://github.com/EdgeNet-project/edgenet/blob/main/docs/tutorials/deploy_edgenet_to_kube.md).

**[TODO]:** Aligning with EdgeNet's instructions for [contribution guidelines](https://github.com/EdgeNet-project/edgenet/blob/main/docs/guides/contribution_guides.md).

For a more in-depth exploration of **EdgeNet** one can visit the [EdgeNet-Testbed site](https://www.edge-net.org/pages/running-experiments.html) and its corresponding [Github repository](https://github.com/EdgeNet-project/edgenet).
