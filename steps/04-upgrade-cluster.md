# Step 4: Upgrade the Cluster Using the Mirror-Registry

In this step, we'll upgrade the cluster using images from the local mirror-registry. This example upgrades from `4.19.1` to `4.19.2`.

Refer to the [official docs](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html-single/disconnected_environments/index#update-disconnected_updating-disconnected-cluster).

---

## 1. List Recommended Updates

```bash
$ oc adm upgrade
Cluster version is 4.19.1

Upstream: https://<OSUS_ROUTE>/api/upgrades_info/v1/graph
Channel: stable-4.19

Recommended updates:

  VERSION     IMAGE
  4.19.5      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:bc79...
  4.19.4      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:8153...
  4.19.3      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:0b44...
  4.19.2      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:1293...
```

## 2. Perform the Upgrade

Choose the target version and run:

```bash
$ oc adm upgrade --allow-explicit-upgrade \
    --to-image <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:<DIGEST>
```

> **Note:** The `--allow-explicit-upgrade` flag is required when specifying a non-adjacent version or when using an explicit image digest.

## 3. Monitor the Upgrade Progress

```bash
$ watch oc adm upgrade
```

Output during the upgrade:

```
Every 2.0s: oc adm upgrade

info: An upgrade is in progress. Working towards 4.19.2: 69 of 920 done (7% complete),
       waiting on config-operator

Upgradeable=False
  Reason: UpdateInProgress
  Message: An update is already in progress and the details are in the Progressing condition
```

You can also monitor the upgrade visually in the OpenShift web console (**Administration → Cluster Version**).

![Upgrade in progress](../../screenshots/upgrade-in-progress.png)

## 4. Verify the Upgrade Completed

```bash
$ oc adm upgrade
Cluster version is 4.19.2

Upstream: https://<OSUS_ROUTE>/api/upgrades_info/v1/graph
Channel: stable-4.19

Recommended updates:

  VERSION     IMAGE
  4.19.5      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:...
  4.19.4      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:...
  4.19.3      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:...
```

Check the upgrade history:

```bash
$ oc get clusterversion version -o json | jq '.status.history'
```

Expected output shows the completed upgrade:

```json
[
  {
    "completionTime": "2025-08-04T13:44:48Z",
    "image": "<MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:...1293...",
    "startedTime": "2025-08-04T12:53:33Z",
    "state": "Completed",
    "verified": true,
    "version": "4.19.2"
  },
  {
    "completionTime": "2025-07-29T18:10:25Z",
    "image": "quay.io/openshift-release-dev/ocp-release@sha256:...4d7f...",
    "startedTime": "2025-07-29T17:34:23Z",
    "state": "Completed",
    "verified": false,
    "version": "4.19.1"
  }
]
```

---

**Next:** [Step 5 — Clean Up Unwanted Images from the Registry](05-cleanup-registry.md)
