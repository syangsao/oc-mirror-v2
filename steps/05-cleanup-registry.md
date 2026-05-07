# Step 5: Delete Unwanted Images from the Local Registry

In this step, we'll clean up unwanted images from the local Quay mirror-registry using the `oc-mirror delete` subcommand.

Refer to the [oc-mirror delete documentation](https://github.com/openshift/oc-mirror?tab=readme-ov-file#delete-sub-command).

---

## 1. Create a Delete ImageSet Configuration

```yaml
# delete-isc.yaml
kind: DeleteImageSetConfiguration
apiVersion: mirror.openshift.io/v2alpha1
delete:
  platform:
    channels:
    - name: stable-4.19
      minVersion: 4.19.1
      maxVersion: 4.19.1
    graph: true
```

The `delete` subcommand works in two phases:

- **Phase 1** — Generates a manifest of images to delete
- **Phase 2** — Executes the deletion on the registry

## 2. Delete Phase 1: Generate the Manifest

```bash
$ oc-mirror delete -c ./delete-isc.yaml --generate \
    --workspace file://<CACHE_DIR>/oc-mirror/mirror1 \
    --delete-id delete1-test \
    --cache-dir <CACHE_DIR> \
    --force-cache-delete \
    docker://<MIRROR_REGISTRY>/<OCMIRROR_PREFIX> --v2
```

This generates two files:

```
working-dir/delete/
├── delete-images-delete1-test.yaml
└── delete-imageset-config-delete1-test.yaml
```

## 3. Delete Phase 2: Execute the Deletion

```bash
$ oc-mirror delete \
    --delete-yaml-file <CACHE_DIR>/oc-mirror/mirror1/working-dir/delete/delete-images-delete1-test.yaml \
    --cache-dir <CACHE_DIR> \
    --force-cache-delete \
    docker://<MIRROR_REGISTRY>/<OCMIRROR_PREFIX> --v2
```

Expected output:

```
🚀 Start deleting the images...
📌 images to delete 382
 ✓  ocp-release:4.19.1-x86_64 ➡️  <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/
 ✓  graph-image:latest ➡️  <MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/
 ...
openshift/release: layer link eligible for deletion: sha256:ffd9...
openshift/release-images: layer link eligible for deletion: sha256:5bcf...
```

> **Note:** The tool deletes image manifests. Blobs shared between multiple images are preserved. You must run the registry's garbage collector to reclaim disk space.

## 4. Verify the Images Were Removed

```bash
$ skopeo list-tags docker://<MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images
{
    "Repository": "<MIRROR_REGISTRY>/<OCMIRROR_PREFIX>/openshift/release-images",
    "Tags": [
        "4.19.2-x86_64",
        "4.19.3-x86_64",
        "4.19.4-x86_64",
        "4.19.5-x86_64"
    ]
}
```

The `4.19.1-x86_64` tag is no longer present — deletion successful.

> **Tip:** After deleting images, log into your Quay registry and run the garbage collector to reclaim disk space. The exact steps depend on your registry configuration.

---

**Next:** [README — Overview and Prerequisites](../README.md)
