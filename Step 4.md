In this 4th step, we'll be upgrading the cluster using the mirror-registry.  We are going from `4.19.1` to `4.19.2`.

Following this guide for the CLI-related steps 

https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html-single/disconnected_environments/index#update-disconnected_updating-disconnected-cluster

1. List out the recommended updates against the local Quay mirror-registry

```
$ oc adm upgrade
Cluster version is 4.19.1

Upstream: https://service-route-openshift-update-service.apps.leia.syangsao.net/api/upgrades_info/v1/graph
Channel: stable-4.19 (available channels: candidate-4.19, candidate-4.20, fast-4.19, stable-4.19)

Recommended updates:

  VERSION     IMAGE
  4.19.5      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:bc79be35e8b8a3719a3e16c91b64e5945c6c4ff1a9c9d0816339f14e2b004385
  4.19.4      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:8153a8c010b292c0c4ca7d8b4ca13ebeb634d449982c66568764511c736281b8
  4.19.3      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:0b44c4b526b4743e744cb989c6fc768fdfd9ac9abffc8f43a014bb90b7bf522d
  4.19.2      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:1293f5ccad2a2776241344faecaf7320f60ee91882df4e24b309f3a7cefc04be
```

2.  Update the cluster to the next release, in this case we'll go with `4.19.2` from the list above

Sample command:

```
$ oc adm upgrade --allow-explicit-upgrade --to-image <defined_registry>/<defined_repository>@<digest>
```

The actual command used:

```
$ oc adm upgrade --allow-explicit-upgrade --to-image mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:1293f5ccad2a2776241344faecaf7320f60ee91882df4e24b309f3a7cefc04be
```

3.  Checking the status of the upgrade

```
$ watch oc adm upgrade
```

Which will output the following:

```
Every 2.0s: oc adm upgrade                                                                                                                                               grogu.syangsao.lab: Mon Aug  4 07:53:45 2025

info: An upgrade is in progress. Working towards 4.19.2: 69 of 920 done (7% complete), waiting on config-operator

Upgradeable=False

  Reason: UpdateInProgress
  Message: An update is already in progress and the details are in the Progressing condition

Upstream: https://service-route-openshift-update-service.apps.leia.syangsao.net/api/upgrades_info/v1/graph
Channel: stable-4.19 (available channels: candidate-4.19, candidate-4.20, fast-4.19, stable-4.19)

Recommended updates:

  VERSION     IMAGE
  4.19.5      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:bc79be35e8b8a3719a3e16c91b64e5945c6c4ff1a9c9d0816339f14e2b004385
  4.19.4      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:8153a8c010b292c0c4ca7d8b4ca13ebeb634d449982c66568764511c736281b8
  4.19.3      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:0b44c4b526b4743e744cb989c6fc768fdfd9ac9abffc8f43a014bb90b7bf522d
```

You can also check the status via the UI:

!(https://github.com/syangsao/oc-mirror-v2/blob/main/update-pic.png)

4.  Verify upgrade is completed:

```
$ oc adm upgrade
Cluster version is 4.19.2

Upstream: https://service-route-openshift-update-service.apps.leia.syangsao.net/api/upgrades_info/v1/graph
Channel: stable-4.19 (available channels: candidate-4.19, candidate-4.20, fast-4.19, stable-4.19)

Recommended updates:

  VERSION     IMAGE
  4.19.5      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:bc79be35e8b8a3719a3e16c91b64e5945c6c4ff1a9c9d0816339f14e2b004385
  4.19.4      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:8153a8c010b292c0c4ca7d8b4ca13ebeb634d449982c66568764511c736281b8
  4.19.3      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:0b44c4b526b4743e744cb989c6fc768fdfd9ac9abffc8f43a014bb90b7bf522d
```

You can also check update history:

```
$ oc get clusterversion version -o json | jq '.status.history'
[
  {
    "completionTime": "2025-08-04T13:44:48Z",
    "image": "mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:1293f5ccad2a2776241344faecaf7320f60ee91882df4e24b309f3a7cefc04be",
    "startedTime": "2025-08-04T12:53:33Z",
    "state": "Completed",
    "verified": true,
    "version": "4.19.2"
  },
  {
    "completionTime": "2025-07-29T18:10:25Z",
    "image": "quay.io/openshift-release-dev/ocp-release@sha256:4d7f10e383deb0c5402f871bf66ebdcad6bb670cb3cf1668bfec5166c56f3196",
    "startedTime": "2025-07-29T17:34:23Z",
    "state": "Completed",
    "verified": false,
    "version": "4.19.1"
  }
]
```