## Set up the required multicluster egine configurations in the ACM Hub cluster
- Follow instructions in [documentation](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.13/html/install/installing#install-on-disconnected-networks) to install the Red Hat Advanced Cluster Management for Kubernetes on the disconnected OpenShift cluster
- We will need to host `rhcos-live-rootfs.x86_64.img` and `rhcos-4.18.1-x86_64-live.x86_64.iso` either on a Apache Web Server or in a S3 Object Storage Bucket so that it can be download
    - The files can be download from [link1](https://mirror.openshift.com/pub/openshift-v4/amd64/dependencies/rhcos/4.18/4.18.1/rhcos-live-rootfs.x86_64.img) and [link2](https://mirror.openshift.com/pub/openshift-v4/amd64/dependencies/rhcos/4.18/4.18.1/rhcos-live.x86_64.iso) respectively.
- The Central Infrastructure Management (CIM) Service is part of the Multicluster Engine Operator, and it needs to be enabled as it runs the Assisted Installer
- Ensure that relevant ports are open
```
$ sudo ss -tuln | grep -E ':67|:68|:69|:80|:123|:5050|:5051|:6180|:6183|:6385|:6388|:8080|:8083|:9999'
$ sudo firewall-cmd --list-ports
```

- Adopt and deploy the following YAML files
    - Ensure that the `registries.conf` are properly populated, pointing to the relevant mirrored registry
    - The release image in the `ClusterImageSet` can be retrieved from the mirror registry
        - Note that ClusterImageSet manages the images required for the cluster
```
$ oc create -f assisted-service-configmap.yaml
$ oc create -f cluster-image-set.yaml
$ oc create -f mirror-config-configmap.yaml
```

- Enable the CIM by configuring AgentServiceConfig
```
$ oc project multicluster-engine
$ oc create -f agent-service-config.yaml
```
```shell
$ oc adm release info 4.18.7
Name:           4.18.7
Digest:         sha256:91037938dc2ebc2732e7baa6eb4192fa4376abab19f0f545848a87ab7c91931d
Created:        2025-03-27T12:40:25Z
OS/Arch:        linux/amd64
Manifests:      758
Metadata files: 2

Pull From: quay.io/openshift-release-dev/ocp-release@sha256:91037938dc2ebc2732e7baa6eb4192fa4376abab19f0f545848a87ab7c91931d

Release Metadata:
  Version:  4.18.7
  Upgrades: 4.17.11, 4.17.12, 4.17.13, 4.17.14, 4.17.15, 4.17.16, 4.17.17, 4.17.18, 4.17.19, 4.17.20, 4.17.21, 4.17.22, 4.17.23, 4.18.1, 4.18.2, 4.18.3, 4.18.4, 4.18.5, 4.18.6
  Metadata:
    url: https://access.redhat.com/errata/RHBA-2025:3293

Component Versions:
  kubectl          1.31.1
  kubernetes       1.31.6
  kubernetes-tests 1.31.6
  machine-os       418.94.202503241359-0 Red Hat Enterprise Linux CoreOS # The spec.osImages.version

Images:
  NAME                                           DIGEST
  agent-installer-api-server                     sha256:eb8f91f85360d27bc1761e82e41d8f5e5e6a725511fc6ee78e8f73a5323a0b33
  agent-installer-csr-approver                   sha256:051938670821bd3083243cb70f91ef3241d77ab8d4230d95cf46647bae1ee8b5
  agent-installer-node-agent                     sha256:1d643311344a4654f56b3b0294d345d83998969cc08ed04eb38bf51111742a03
```

- Check that the pods are up and running properly
```
$ oc get pods -n multicluster-engine | grep -i assisted-service
$ oc get pods -n multicluster-engine | grep -i assisted-image-service
```