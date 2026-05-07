# Step 1: Mirror Images from Red Hat to Your Local Registry

In this step, we'll mirror release and operator images from the public Red Hat repository onto your local Quay registry using `oc-mirror v2`.

This assumes you have already followed Red Hat's [documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html-single/disconnected_environments/index) on downloading the latest `oc-mirror` CLI and setting up your registry credentials.

---

## 1. Verify Your oc-mirror Version

```bash
$ oc-mirror version
⚠️  oc-mirror v1 is deprecated (starting in 4.18 release) and will be removed in a future release
     - please migrate to oc-mirror --v2
```

> **Note:** The version output may include a deprecation warning. As long as `--v2` flag works, you're good.

The `oc-mirror` tool supports three workflow modes ([source](https://github.com/openshift/oc-mirror/blob/main/README.md)):

- **mirrorToDisk** (`m2d`) — Pulls images from a source and packs them into a tar archive on disk
- **diskToMirror** (`d2m`) — Copies images from a tar archive to a container registry
- **mirrorToMirror** (`m2m`) — Copies images directly from source to destination registry

## 2. List Available Releases and Operators

List available releases for `v4.19`:

```bash
$ oc-mirror list releases --channel=stable-4.19
Channel: stable-4.19
Architecture: amd64
4.19.0
4.19.1
4.19.2
4.19.3
4.19.4
4.19.5
```

List operators (example: `odf-operator`):

```bash
$ oc-mirror list operators --package odf-operator \
    --catalog registry.redhat.io/redhat/redhat-operator-index:v4.19
NAME          DISPLAY NAME  DEFAULT CHANNEL
odf-operator                stable-4.19

PACKAGE       CHANNEL      HEAD
odf-operator  stable-4.18  odf-operator.v4.18.8-rhodf
odf-operator  stable-4.19  odf-operator.v4.19.1-rhodf
```

## 3. Create an ImageSetConfiguration

Refer to the [official docs](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html-single/disconnected_environments/index#oc-mirror-building-image-set-config-v2_about-installing-oc-mirror-v2).

```yaml
# isc.yaml
kind: ImageSetConfiguration
apiVersion: mirror.openshift.io/v2
mirror:
  platform:
    channels:
    - name: stable-4.19
      minVersion: 4.19.1
      maxVersion: 4.19.5
    graph: true
  operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.19
      packages:
        - name: odf-operator
        - name: ocs-operator
        - name: cephcsi-operator
        - name: rook-ceph-operator
        - name: mcg-operator
        - name: odf-csi-addons-operator
        - name: recipe
        - name: ocs-client-operator
        - name: odf-prometheus-operator
        - name: odf-dependencies
        - name: odf-multicluster-orchestrator
        - name: odr-cluster-operator
        - name: odr-hub-operator
  additionalImages:
    - name: registry.redhat.io/odf4/odf-must-gather-rhel9:v4.19
```

## 4. Mirror to Disk (Internet → Local)

```bash
$ oc-mirror -c ./isc.yaml --cache-dir <CACHE_DIR> \
    file://<CACHE_DIR>/oc-mirror/mirror1 --v2
```

> **Tip:** Always use `--cache-dir` to keep cached data outside your home directory. This saves space on your home filesystem and makes backups easier.

Expected output summary:
```
✓  761 / 761 release images mirrored successfully
✓  6 / 6 operator images mirrored successfully
✓  1 / 1 additional images mirrored successfully
📦 Preparing the tarball archive...
```

> **Note:** This step can take 30–60 minutes depending on network speed and image size (~79 GiB in this example).

## 5. Mirror to Registry (Disk → Quay)

```bash
$ oc-mirror -c ./isc.yaml --cache-dir <CACHE_DIR> \
    --from file://<CACHE_DIR>/oc-mirror/mirror1 \
    docker://<MIRROR_REGISTRY>/<OCMIRROR_PREFIX> --v2
```

Expected output summary:
```
✓  761 / 761 release images mirrored successfully
✓  6 / 6 operator images mirrored successfully
✓  1 / 1 additional images mirrored successfully
```

The tool also generates cluster resource files in the working directory:

```
working-dir/cluster-resources/
├── cc-redhat-operator-index-v4-19.yaml      # ClusterCatalog
├── cs-redhat-operator-index-v4-19.yaml      # CatalogSource
├── idms-oc-mirror.yaml                       # ImageDigestMirrorSet
├── itms-oc-mirror.yaml                       # ImageTagMirrorSet
├── signature-configmap.json                  # Release signatures
├── signature-configmap.yaml                  # Release signatures (YAML)
└── updateService.yaml                        # OSUS config (optional)
```

## 6. Verify Images on the Local Registry

```bash
$ oc-mirror list operators --package odf-operator \
    --catalog <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/redhat/redhat-operator-index:v4.19
```

List release image tags:

```bash
$ skopeo list-tags docker://<MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images
{
    "Repository": "<MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images",
    "Tags": [
        "4.19.1-x86_64",
        "4.19.2-x86_64",
        "4.19.3-x86_64",
        "4.19.4-x86_64",
        "4.19.5-x86_64"
    ]
}
```

---

**Next:** [Step 2 — Convert a Connected Cluster to Disconnected](02-convert-to-disconnected.md)
