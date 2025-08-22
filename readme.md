This repo is created with notes for setting up an OpenShift v4.19 cluster in disconnected mode using oc-mirror v2.

The local registry used for this configuration is a Quay mirror-registry running on RHEL that was installed following this [guide](https://www.redhat.com/en/blog/introducing-mirror-registry-for-red-hat-openshift)

`STEP 1` - [Using oc-mirror v2 to mirror the release and operator images from Red Hat to a local registry](step 1.md)

`STEP 2` - Converting a connected OpenShift cluster to a disconnected one while talking to a local registry

`STEP 3` - Installing OSUS in a disconnected environment

`STEP 4` - Upgrading the cluster while talking to the local registry

`STEP 5` - Deleting unwanted images on the local registry
