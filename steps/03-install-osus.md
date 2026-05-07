# Step 3: Install OpenShift Update Service (OSUS) in a Disconnected Environment

In this step, we'll install the **OpenShift Update Service (OSUS)** so your disconnected cluster can receive update recommendations.

In a connected environment, clusters query Red Hat's public OSUS API. In a disconnected environment, you must deploy your own instance. See the [official docs](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html-single/updating_clusters/index#update-service-overview_updating-restricted-network-cluster-osus).

---

## 1. Configure the ConfigMap with the Full-Chain Certificate

Create a ConfigMap using your local registry's **full chain** certificate:

```bash
$ oc create configmap registry-config \
    --from-file=<MIRROR_REGISTRY>.pem=<REGISTRY_CERT_CHAIN> \
    -n openshift-config
```

> **Important:** You must use the **full chain** certificate (not just the leaf certificate). This includes the intermediate and root CA certificates. See [Red Hat Solution 7040684](https://access.redhat.com/solutions/7040684).

## 2. Add the `updateservice-registry` Entry

Edit the ConfigMap to add a duplicate entry for `updateservice-registry`:

```bash
$ oc edit cm registry-config -n openshift-config
```

Copy the three lines for your mirror registry and create matching entries for `updateservice-registry`:

```yaml
  <MIRROR_REGISTRY>.pem: "-----BEGIN CERTIFICATE-----\r\n...\r\n-----END CERTIFICATE-----\r\n..."
  updateservice-registry: "-----BEGIN CERTIFICATE-----\r\n...\r\n-----END CERTIFICATE-----\r\n..."
```

## 3. Configure Additional Trust

Ensure the image config references the ConfigMap:

```bash
$ oc edit image.config.openshift.io cluster
```

Verify the spec includes:

```yaml
spec:
  additionalTrustedCA:
    name: registry-config
```

## 4. Deploy the OpenShift Update Service

```bash
$ NAMESPACE=openshift-update-service
$ NAME=service
$ RELEASE_IMAGES=<MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images
$ GRAPH_DATA_IMAGE=<MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/graph-image:latest

$ oc -n "${NAMESPACE}" create -f - <<EOF
apiVersion: updateservice.operator.openshift.io/v1
kind: UpdateService
metadata:
  name: ${NAME}
spec:
  replicas: 2
  releases: ${RELEASE_IMAGES}
  graphDataImage: ${GRAPH_DATA_IMAGE}
EOF
```

### Troubleshooting: Certificate Verify Failed

If your pods fail with `certificate verify failed` errors:

```
ERROR graph_builder::graph] http transport error: error sending request for url
(https://<MIRROR_REGISTRY>/v2/): error trying to connect: error:1416F086:SSL
routines:tls_process_server_certificate:certificate verify failed
(unable to get local issuer certificate)
```

**Solution:** Make sure your registry is configured with a **full-chain** certificate (leaf + intermediate + root). A self-signed leaf certificate alone is not sufficient — the graph builder needs the entire chain to validate the connection.

## 5. Verify the Graph Is Available

Run the following on the OSUS cluster:

```bash
$ NAMESPACE=openshift-update-service
$ NAME=service
$ POLICY_ENGINE_GRAPH_URI="$(oc -n "${NAMESPACE}" get -o jsonpath='{.status.policyEngineURI}/api/upgrades_info/v1/graph{"\n"}' updateservice "${NAME}")"
$ echo "${POLICY_ENGINE_GRAPH_URI}"
```

You should see output like:

```
https://<OSUS_ROUTE>/api/upgrades_info/v1/graph
```

> **Note:** Ensure the OSUS route (`*.apps.example.net`) also uses a valid certificate. Your disconnected cluster needs to trust this certificate as well.

Test the endpoint:

```bash
$ curl -kvv https://<OSUS_ROUTE>/api/upgrades_info/v1/graph
```

## 6. Verify OSUS on the Client Cluster

Update the upstream address on your disconnected cluster, then verify:

```bash
$ oc adm upgrade
Cluster version is 4.19.1

Upstream: https://<OSUS_ROUTE>/api/upgrades_info/v1/graph
Channel: stable-4.19 (available channels: candidate-4.19, candidate-4.20, fast-4.19, stable-4.19)

Recommended updates:
  VERSION     IMAGE
  4.19.5      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:...
  4.19.4      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:...
  4.19.3      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:...
  4.19.2      <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images@sha256:...
```

Check for recommended releases with known issues:

```bash
$ export OC_ENABLE_CMD_UPGRADE_RECOMMEND=true
$ oc adm upgrade recommend
```

Check upgrade status:

```bash
$ export OC_ENABLE_CMD_UPGRADE_STATUS=true
$ oc adm upgrade status
```

---

**Next:** [Step 4 — Upgrade the Cluster via the Local Registry](04-upgrade-cluster.md)
