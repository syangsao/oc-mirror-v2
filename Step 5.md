In the 5th and final step, we'll be deleting the unwanted images from the local quay mirror registry using oc-mirror v2

1.  Following the guide here using the new `DeleteImageSetConfiguration` option via oc-mirror v2

https://github.com/openshift/oc-mirror?tab=readme-ov-file#delete-sub-command

```
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

`Delete sub command`

There is also a delete sub command to delete images specified in the Delete Image Set Configuration from a remote registry. This command is split in two phases:

`Delete Phase 1:` Using a delete image set configuration as an input, oc-mirror discovers all images that needed to be deleted. These images are included in a delete-images file to be consumed as input in the second phase.

```
./bin/oc-mirror delete -c ./delete-isc.yaml --generate --workspace file:///home/<user>/oc-mirror/delete1 --delete-id delete1-test docker://localhost:6000 --v2
```

`Delete Phase 2:` Using the file generated in the first phase, oc-mirror will delete all container image manifests specified in this file on the destination specified on the command line. It is up to the container registry to run the garbage collector to clean up all the blobs which are not referenced by a manifest. Deleting only manifests is safer since blobs shared between more than one image are not going to be deleted.

```
./bin/oc-mirror delete --delete-yaml-file /home/<user>/oc-mirror/delete1/working-dir/delete/delete-images-delete1-test.yaml docker://localhost:6000 --v2
```

These were commands used to mirror originally, so we will use some of the options from this.

```
$ oc-mirror -c ./isc.yaml --cache-dir /grogu/syangsao file:///grogu/syangsao/oc-mirror/mirror1 --v2
$ oc-mirror -c ./isc.yaml --cache-dir /grogu/syangsao --from file:///grogu/syangsao/oc-mirror/mirror1 docker://mirror.syangsao.net:8443/ocp4 --v2
```

2.  `Delete Phase 1` 

```
$ oc-mirror delete -c ./delete-isc.yaml --generate --workspace file:///grogu/syangsao/oc-mirror/mirror1 --delete-id delete1-test --cache-dir /grogu/syangsao --force-cache-delete docker://mirror.syangsao.net:8443/ocp4 --v2
2025/08/14 13:22:00  [INFO]   : 👋 Hello, welcome to oc-mirror
2025/08/14 13:22:00  [INFO]   : ⚙️  setting up the environment for you...
2025/08/14 13:22:00  [INFO]   : 🔀 workflow mode: diskToMirror / delete
2025/08/14 13:22:00  [INFO]   : 🕵  going to discover the necessary images...
2025/08/14 13:22:00  [INFO]   : 🔍 collecting release images...
2025/08/14 13:22:00  [INFO]   : 🔍 collecting operator images...
2025/08/14 13:22:00  [INFO]   : 🔍 collecting additional images...
2025/08/14 13:22:00  [INFO]   : 🔍 collecting helm images...
2025/08/14 13:22:00  [INFO]   : 📄 Generating delete file...
2025/08/14 13:22:00  [INFO]   : /grogu/syangsao/oc-mirror/mirror1/working-dir/delete file created
2025/08/14 13:22:00  [INFO]   : delete time     : 223.13868ms
odf4/cephcsi-operator-bundle
odf4/cephcsi-operator-bundle: marking manifest sha256:326310df08e7c01c962f686733cacad88814892d19788e57ea6a18a06743ecfb 
[...]
redhat/redhat-operator-index: layer link eligible for deletion: sha256:f6509a29002722177aa04e958f32f72ff043d42096e7925b2e33b39da60d6aca
2025/08/14 13:22:54  [INFO]   : 👋 Goodbye, thank you for using oc-mirror

$ tree /grogu/syangsao/oc-mirror/mirror1/working-dir/delete/
/grogu/syangsao/oc-mirror/mirror1/working-dir/delete/
├── delete-images-delete1-test.yaml
└── delete-imageset-config-delete1-test.yaml
```

3.  `Delete Phase 2`

```
$ oc-mirror delete --delete-yaml-file /grogu/syangsao/oc-mirror/mirror1/working-dir/delete/delete-images-delete1-test.yaml --cache-dir /grogu/syangsao --force-cache-delete docker://mirror.syangsao.net:8443/ocp4 --v2

2025/08/14 13:35:08  [INFO]   : 👋 Hello, welcome to oc-mirror
2025/08/14 13:35:08  [INFO]   : ⚙️  setting up the environment for you...
2025/08/14 13:35:08  [INFO]   : 🔀 workflow mode: diskToMirror / delete
2025/08/14 13:35:08  [INFO]   : 👀 Reading delete file...
2025/08/14 13:35:08  [INFO]   : 🚀 Start deleting the images...
2025/08/14 13:35:08  [INFO]   : 📌 images to delete 382 
 ✓   (0s) ocp-release:4.19.1-x86_64 ➡️  cache 
 ✓   (0s) ocp-release:4.19.1-x86_64 ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) graph-image:latest ➡️  cache 
 ✓   (0s) ocp-v4.0-art-dev@sha256:e96045efdc85ad51ad3c086c1958c4293f2cc4f281d52b93dca68948d32eccea ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) graph-image:latest ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) ocp-v4.0-art-dev@sha256:e96045efdc85ad51ad3c086c1958c4293f2cc4f281d52b93dca68948d32eccea ➡️  cache 
 ✓   (0s) ocp-v4.0-art-dev@sha256:977539eb68b40c6aca1560da3699f5cdc2bf21d3251403642d9e09f2d52b88cb ➡️  cache 
 ✓   (0s) ocp-v4.0-art-dev@sha256:977539eb68b40c6aca1560da3699f5cdc2bf21d3251403642d9e09f2d52b88cb ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
[...]
openshift/release: layer link eligible for deletion: sha256:ffd9f6812856586e3ad9c32eba69add16ea1c98375c3021c6397df1de1c2ff94
openshift/release-images: layer link eligible for deletion: sha256:5bcf31358da389d8e94e0f85fed58a91da398d0f112d73bef0186d2c765beb1e
openshift/release-images: layer link eligible for deletion: sha256:eb48f2522566c2f1ba491eb7e18d9c520b5f7636dd55d613f5cedd562e28df6e
2025/08/14 13:35:50  [INFO]   : 👋 Goodbye, thank you for using oc-mirror
```

4.  Verify the release image `4.19.1-x86_64` is gone

```
$ skopeo list-tags docker://mirror.syangsao.net:8443/ocp4/openshift/release-images
{
    "Repository": "mirror.syangsao.net:8443/ocp4/openshift/release-images",
    "Tags": [
        "4.19.2-x86_64",
        "4.19.3-x86_64",
        "4.19.4-x86_64",
        "4.19.5-x86_64"
    ]
}
```