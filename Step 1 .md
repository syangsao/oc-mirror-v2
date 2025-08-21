In this first comment, these are the steps we'll take to mirror the images from the public Red Hat repository onto your local Quay repository using `oc-mirror v2`.

1.  Pulled the following `oc-mirror` version

```
$ oc-mirror version
⚠️  oc-mirror v1 is deprecated (starting in 4.18 release) and will be removed in a future release - please migrate to oc-mirror --v2

WARNING: This version information is deprecated and will be replaced with the output from --short. Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"", Minor:"", GitVersion:"4.19.0-202507101307.p0.g9283907.assembly.stream.el9-9283907", GitCommit:"9283907f659b8196cf0a9a45b9c4b0016edb25d5", GitTreeState:"clean", BuildDate:"2025-07-10T13:56:48Z", GoVersion:"go1.23.9 (Red Hat 1.23.9-1.el9_6) X:strictfipsruntime", Compiler:"gc", Platform:"linux/amd64"}
```

From github link, we'll do both the top options below.

`https://github.com/openshift/oc-mirror/blob/main/README.md`

```
mirrorToDisk (m2d for shorter) - pulls the container images from a source specified in the image set configuration and packs them into a tar archive on disk (local directory).
diskToMirror (d2m for shorter) - copy the containers images from the tar archive to a container registry.
mirrorToMirror (m2m for shorter) - copy the container images from the source specified in the image set configuration to the destination (container registry).
```
2.  Listing out the releases and operator for `v4.19`

```
$ time oc-mirror list releases --channel=stable-4.19
[...]
Channel: stable-4.19
Architecture: amd64
4.18.1
[...]
4.18.20
4.19.0
4.19.1
4.19.2
4.19.3
4.19.4
4.19.5
```

Concentrating on 1 operator - odf-operator to sync up and test

```
$ oc-mirror list operators --package odf-operator --catalog registry.redhat.io/redhat/redhat-operator-index:v4.19 
[...]
NAME          DISPLAY NAME  DEFAULT CHANNEL
odf-operator                stable-4.19

PACKAGE       CHANNEL      HEAD
odf-operator  stable-4.18  odf-operator.v4.18.8-rhodf
odf-operator  stable-4.19  odf-operator.v4.19.1-rhodf
```

3.  Create an ImageSetConfiguration 

`https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html-single/disconnected_environments/index#oc-mirror-building-image-set-config-v2_about-installing-oc-mirror-v2`

```
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

4.  Kicking off the 1st mirror, Mirror (Internet) to Disk (local)

Make sure to add the --cache-dir so it'll cache the data outside of ~/ or /home/syangsao

```
$ oc-mirror -c ./isc.yaml --cache-dir /grogu/syangsao file:///grogu/syangsao/oc-mirror/mirror1 --v2
2025/07/25 14:05:46  [INFO]   : 👋 Hello, welcome to oc-mirror
2025/07/25 14:05:46  [INFO]   : ⚙️  setting up the environment for you...
2025/07/25 14:05:46  [INFO]   : 🔀 workflow mode: mirrorToDisk 
2025/07/25 14:05:46  [INFO]   : 🕵  going to discover the necessary images...
2025/07/25 14:05:46  [INFO]   : 🔍 collecting release images...
2025/07/25 14:06:39  [INFO]   : 🔍 collecting operator images...
 ✓   (38s) Collecting catalog registry.redhat.io/redhat/redhat-operator-index:v4.19 
2025/07/25 14:07:17  [INFO]   : 🔍 collecting additional images...
2025/07/25 14:07:17  [INFO]   : 🔍 collecting helm images...
2025/07/25 14:07:17  [INFO]   : 🔂 rebuilding catalogs
 ✓   (43s) Rebuilding catalog docker://registry.redhat.io/redhat/redhat-operator-index:v4.19 
2025/07/25 14:08:01  [INFO]   : 🚀 Start copying the images...
2025/07/25 14:08:01  [INFO]   : 📌 images to copy 768 
 ✓   (6s) ocp-v4.0-art-dev@sha256:0068972abd6f7832be48228f1667187e654d1800a907665b42ad25fb98adf936 ➡️  cache 
 ✓   (6s) ocp-v4.0-art-dev@sha256:7825e84c62ed6f33ceb8b1aeb27333f12008ea9b20d915ab5a43614a7fa776d7 ➡️  cache 
 ✓   (7s) ocp-v4.0-art-dev@sha256:006955bddc5097d4a58714e43c4eac4b9a01340c086953d814743885b797ec44 ➡️  cache 
 ✓   (15s) ocp-v4.0-art-dev@sha256:00e4e6df31a34e027d0a48cd5b66bf8464a92c6774cfd6d6f859d237da518bab ➡️  cache 
[...]
 ✓   (4s) odf-operator-bundle@sha256:0ec6b428ed6c3d4e39e0f5adba9373c6f39a21ecf09cf69af77f3c007b36ed54 ➡️  cache 
 ✓   (46s) ose-kube-rbac-proxy-rhel9@sha256:d37a6d10b0fa07370066a31fdaffe2ea553faf4e4e98be7fcef5ec40d62ffe29 ➡️  cache 
 ✓   (6s) redhat-operator-index:v4.19 ➡️  cache 
768 / 768 (24m8s) [=============================================================================================================================================================================] 100 %
 ✓   (30s) odf-must-gather-rhel9:v4.18 ➡️  cache 
2025/07/25 14:32:10  [INFO]   : === Results ===
2025/07/25 14:32:10  [INFO]   :  ✓  761 / 761 release images mirrored successfully
2025/07/25 14:32:10  [INFO]   :  ✓  6 / 6 operator images mirrored successfully
2025/07/25 14:32:10  [INFO]   :  ✓  1 / 1 additional images mirrored successfully
2025/07/25 14:32:10  [INFO]   : 📦 Preparing the tarball archive...
2025/07/25 14:58:59  [INFO]   : mirror time     : 53m12.226907093s
2025/07/25 14:58:59  [INFO]   : 👋 Goodbye, thank you for using oc-mirror
```

5.  Kicking off the 2nd mirror, Disk (local) to Mirror (Quay)

```
$ oc-mirror -c ./isc.yaml --cache-dir /grogu/syangsao --from file:///grogu/syangsao/oc-mirror/mirror1 docker://mirror.syangsao.net:8443/ocp4 --v2
2025/07/26 07:40:11  [INFO]   : 👋 Hello, welcome to oc-mirror
2025/07/26 07:40:11  [INFO]   : ⚙️  setting up the environment for you...
2025/07/26 07:40:11  [INFO]   : 🔀 workflow mode: diskToMirror 
2025/07/26 07:40:11  [INFO]   : 📦 Extracting mirror archive(s)...
/grogu/syangsao/oc-mirror/mirror1/mirror_000001.tar (66.9 GiB / 78.9 GiB) [=================================================================================================>-------------/grogu/syangsao/oc-mirror/mirror1/mirror_000001.tar (78.9 GiB / 78.9 GiB) [=======================================================================================================] 27m22s
2025/07/26 08:07:33  [INFO]   : 🕵  going to discover the necessary images...
2025/07/26 08:07:33  [INFO]   : 🔍 collecting release images...
2025/07/26 08:07:34  [INFO]   : 🔍 collecting operator images...
 ✓   () Collecting catalog registry.redhat.io/redhat/redhat-operator-index:v4.19 
2025/07/26 08:07:34  [INFO]   : 🔍 collecting additional images...
2025/07/26 08:07:34  [INFO]   : 🔍 collecting helm images...
2025/07/26 08:07:34  [INFO]   : 🚀 Start copying the images...
2025/07/26 08:07:34  [INFO]   : 📌 images to copy 768 
 ✓   (0s) ocp-v4.0-art-dev@sha256:006955bddc5097d4a58714e43c4eac4b9a01340c086953d814743885b797ec44 ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) ocp-v4.0-art-dev@sha256:7825e84c62ed6f33ceb8b1aeb27333f12008ea9b20d915ab5a43614a7fa776d7 ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) ocp-v4.0-art-dev@sha256:0068972abd6f7832be48228f1667187e654d1800a907665b42ad25fb98adf936 ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) odf-operator-bundle@sha256:0ec6b428ed6c3d4e39e0f5adba9373c6f39a21ecf09cf69af77f3c007b36ed54 ➡️  mirror.syangsao.net:8443/ocp4/odf4/ 
 ✓   (0s) odf-must-gather-rhel9:v4.18 ➡️  mirror.syangsao.net:8443/ocp4/odf4/ 
 ✓   (0s) ose-kube-rbac-proxy-rhel9@sha256:d37a6d10b0fa07370066a31fdaffe2ea553faf4e4e98be7fcef5ec40d62ffe29 ➡️  mirror.syangsao.net:8443/ocp4/openshift4/
[...]
 ✓   (0s) ocp-release:4.19.2-x86_64 ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) ocp-release:4.19.4-x86_64 ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) ocp-release:4.19.3-x86_64 ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) graph-image:latest ➡️  mirror.syangsao.net:8443/ocp4/openshift/ 
 ✓   (0s) odf-console-rhel9@sha256:63be93044c5b8006020ea5438d39bf0d700789f7571f9b1f7e5e67f429e8cc6c ➡️  mirror.syangsao.net:8443/ocp4/odf4/ 
 ✓   (0s) odf-rhel9-operator@sha256:e6a178ad42175d779adf6aa17fab3221326adbe5d6b6a8dc2619f5e601735267 ➡️  mirror.syangsao.net:8443/ocp4/odf4/ 
 ✓   (0s) odf-cli-rhel9@sha256:be34da8f9474c57ca1e3ae003777b5b7e687068e529fd0890dde565810652991 ➡️  mirror.syangsao.net:8443/ocp4/odf4/ 
 ✓   (0s) odf-operator-bundle@sha256:0ec6b428ed6c3d4e39e0f5adba9373c6f39a21ecf09cf69af77f3c007b36ed54 ➡️  mirror.syangsao.net:8443/ocp4/odf4/ 
 ✓   (0s) odf-must-gather-rhel9:v4.18 ➡️  mirror.syangsao.net:8443/ocp4/odf4/ 
 ✓   (0s) ose-kube-rbac-proxy-rhel9@sha256:d37a6d10b0fa07370066a31fdaffe2ea553faf4e4e98be7fcef5ec40d62ffe29 ➡️  mirror.syangsao.net:8443/ocp4/openshift4/ 
768 / 768 (1m31s) [================================================================================================================================================================] 100 %
 ✓   (0s) redhat-operator-index:v4.19 ➡️  mirror.syangsao.net:8443/ocp4/redhat/ 
2025/07/26 08:09:06  [INFO]   : === Results ===
2025/07/26 08:09:06  [INFO]   :  ✓  761 / 761 release images mirrored successfully
2025/07/26 08:09:06  [INFO]   :  ✓  6 / 6 operator images mirrored successfully
2025/07/26 08:09:06  [INFO]   :  ✓  1 / 1 additional images mirrored successfully
2025/07/26 08:09:06  [INFO]   : 📄 Generating IDMS file...
2025/07/26 08:09:06  [INFO]   : /grogu/syangsao/oc-mirror/mirror1/working-dir/cluster-resources/idms-oc-mirror.yaml file created
2025/07/26 08:09:06  [INFO]   : 📄 Generating ITMS file...
2025/07/26 08:09:06  [INFO]   : /grogu/syangsao/oc-mirror/mirror1/working-dir/cluster-resources/itms-oc-mirror.yaml file created
2025/07/26 08:09:06  [INFO]   : 📄 Generating CatalogSource file...
2025/07/26 08:09:06  [INFO]   : /grogu/syangsao/oc-mirror/mirror1/working-dir/cluster-resources/cs-redhat-operator-index-v4-19.yaml file created
2025/07/26 08:09:06  [INFO]   : 📄 Generating ClusterCatalog file...
2025/07/26 08:09:06  [INFO]   : /grogu/syangsao/oc-mirror/mirror1/working-dir/cluster-resources/cc-redhat-operator-index-v4-19.yaml file created
2025/07/26 08:09:06  [INFO]   : 📄 Generating Signature Configmap...
2025/07/26 08:09:06  [INFO]   : /grogu/syangsao/oc-mirror/mirror1/working-dir/cluster-resources/signature-configmap.json file created
2025/07/26 08:09:06  [INFO]   : /grogu/syangsao/oc-mirror/mirror1/working-dir/cluster-resources/signature-configmap.yaml file created
2025/07/26 08:09:06  [INFO]   : 📄 Generating UpdateService file...
2025/07/26 08:09:06  [INFO]   : /grogu/syangsao/oc-mirror/mirror1/working-dir/cluster-resources/updateService.yaml file created
2025/07/26 08:09:06  [INFO]   : mirror time     : 28m55.198402235s
2025/07/26 08:09:06  [INFO]   : 👋 Goodbye, thank you for using oc-mirror
```

6.  Verifying Quay for the operator and release images are uploaded on the local registry

```
$ oc-mirror list operators --package odf-operator --catalog mirror.syangsao.net:8443/ocp4/redhat/redhat-operator-index:v4.19
[...]

NAME          DISPLAY NAME  DEFAULT CHANNEL
odf-operator                stable-4.19

PACKAGE       CHANNEL      HEAD
odf-operator  stable-4.18  odf-operator.v4.18.8-rhodf
odf-operator  stable-4.19  odf-operator.v4.19.1-rhodf

$ skopeo list-tags docker://mirror.syangsao.net:8443/ocp4/openshift/release-images
{
    "Repository": "mirror.syangsao.net:8443/ocp4/openshift/release-images",
    "Tags": [
        "4.19.1-x86_64",
        "4.19.2-x86_64",
        "4.19.3-x86_64",
        "4.19.4-x86_64"
        "4.19.5-x86_64"
    ]
}
```