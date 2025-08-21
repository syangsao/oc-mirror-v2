In this 2nd step, we'll be converting an OpenShift cluster from `connected` to `disconnected` to talk to the local Quay mirror-registry hosting the release and operator images from the 1st step.

https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html-single/disconnected_environments/index#connected-to-disconnected-config-registry_connected-to-disconnected

1.  Make a copy of the pull-secret

```
$ oc get secret pull-secret -n openshift-config -o yaml > pull-secret.yaml
```

2.  Create a config file for your destination repo

```
$ cat config.json
{
  "auths": {
    "mirror.syangsao.net:8443": {
      "auth": "b2NwNCtyb2JvdDpXT05JSkVLUlU2T0E1OTc0TkI3NzVVMklCQ0FET0ZFVVJRVVE3SDZDSzRCUlJIV0tMVTRORVNFUUtRVTlaTlIw",
      "email": ""
    }
  }
}
```

3.  Update the current configuration by overriding the pull-secret

```
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=config.json
```

4.  Update the current configuration by overriding the pull-secret

```
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=config.json
```

5.  Verify the pull secret has been added

```
$ oc get secret pull-secret -n openshift-config -o yaml
apiVersion: v1
data:
  .dockerconfigjson: ewogICJhdXRocyI6IHsKICAgICJtaXJyb3Iuc3lhbmdzYW8ubmV0Ojg0NDMiOiB7CiAgICAgICJhdXRoIjogImIyTndOQ3R5YjJKdmREcFhUMDVKU2tWTFVsVTJUMEUxT1RjMFRrSTNOelZWTWtsQ1EwRkVUMFpGVlZKUlZWRTNTRFpEU3pSQ1VsSklWMHRNVlRST1JWTkZVVXRSVlRsYVRsSXciLAogICAgICAiZW1haWwiOiAiIgogICAgfQogIH0KfQo=
kind: Secret
metadata:
  creationTimestamp: "2025-07-15T23:14:12Z"
  name: pull-secret
  namespace: openshift-config
  resourceVersion: "6568877"
  uid: 8c257d3b-cb55-4e69-89cd-97ce9486805d
type: kubernetes.io/dockerconfigjson
```

6.  Verify `.dockerconfigjson` secret data is valid for the local Quay mirror registry

```
$ echo "ewogICJhdXRocyI6IHsKICAgICJtaXJyb3Iuc3lhbmdzYW8ubmV0Ojg0NDMiOiB7CiAgICAgICJhdXRoIjogImIyTndOQ3R5YjJKdmREcFhUMDVKU2tWTFVsVTJUMEUxT1RjMFRrSTNOelZWTWtsQ1EwRkVUMFpGVlZKUlZWRTNTRFpEU3pSQ1VsSklWMHRNVlRST1JWTkZVVXRSVlRsYVRsSXciLAogICAgICAiZW1haWwiOiAiIgogICAgfQogIH0KfQo=" |base64 -d
{
  "auths": {
    "mirror.syangsao.net:8443": {
      "auth": "b2NwNCtyb2JvdDpXT05JSkVLUlU2T0E1OTc0TkI3NzVVMklCQ0FET0ZFVVJRVVE3SDZDSzRCUlJIV0tMVTRORVNFUUtRVTlaTlIw",
      "email": ""
    }
  }
}
```

7.  Download the certificate from the local Quay repo and create a configMap with the certificate

```
$ oc create configmap registry-config --from-file=mirror.syangsao.net..8443=mirror-syangsao-net.pem -n openshift-config
configmap/registry-config created

$ oc get cm registry-config -o yaml
apiVersion: v1
data:
  mirror.syangsao.net..8443: "-----BEGIN CERTIFICATE-----\r\nMIIECjCCA5CgAwIBAgIQLHdgnL6aZPG5ps/Jf5vgaDAKBggqhkjOPQQDAzBLMQswCQYDVQQGEwJBVDEQMA4GA1UEChMHWmVyb1NTTDEqMCgGA1UEAxMhWmVyb1NTTCBFQ0MgRG9tYWluIFNlY3VyZSBTaXRlIENBMB4XDTI1MDYyNDAwMDAwMFoXDTI1MDkyMjIzNTk1OVowHjEcMBoGA1UEAxMTbWlycm9yLnN5YW5nc2FvLm5ldDBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABEScWYpjITTUYTGm4KF0wGE82xX3/3jzLljBkTLHtX4kKSv7Px1NrEUiSzus1J8QsnzawdPuUBrrOtiCQbJHoj2jggKBMIICfTAfBgNVHSMEGDAWgBQPa+ZLzjlHrvZ+kB558DCRkshfozAdBgNVHQ4EFgQUn05q8O48TJkeeKbrAP+rYmaFaOMwDgYDVR0PAQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMEkGA1UdIARCMEAwNAYLKwYBBAGyMQECAk4wJTAjBggrBgEFBQcCARYXaHR0cHM6Ly9zZWN0aWdvLmNvbS9DUFMwCAYGZ4EMAQIBMIGIBggrBgEFBQcBAQR8MHowSwYIKwYBBQUHMAKGP2h0dHA6Ly96ZXJvc3NsLmNydC5zZWN0aWdvLmNvbS9aZXJvU1NMRUNDRG9tYWluU2VjdXJlU2l0ZUNBLmNydDArBggrBgEFBQcwAYYfaHR0cDovL3plcm9zc2wub2NzcC5zZWN0aWdvLmNvbTCCAQYGCisGAQQB1nkCBAIEgfcEgfQA8gB3AN3cyjSV1+EWBeeVMvrHn/g9HFDf2wA6FBJ2Ciysu8gqAAABl6HR33YAAAQDAEgwRgIhAK/WG5Khs4ws8+vVDM82cxzrxti5DIXDm2at4gwg9iCfAiEAmIlfds84Cmia57WR3KxON12lJ7EXfOLU+NwtdEBdT50AdwAN4fIwK9MNwUBiEgnqVS78R3R8sdfpMO8OQh60fk6qNAAAAZeh0d9DAAAEAwBIMEYCIQDHM74BMlsaFOf7jgK9nCnfbJY+Gfnp3btc37zbg0yiugIhAPVMUEjHYw4TNjravHu91kIeRbEiXoPFsN8OcHlPOJ6UMB4GA1UdEQQXMBWCE21pcnJvci5zeWFuZ3Nhby5uZXQwCgYIKoZIzj0EAwMDaAAwZQIxALGy3YfFE6WuqVN1jaFFBj+9zhw4a0ZEfXAH94ptwHYALDV4kJbJlqWADu5/Wx9f0AIwJlNoyraAPy6ggvvcqRdAhtEaW90zM1LW/yBoKmq2RqU4VT8Qybxv7L+Tq5TSBQVF\r\n-----END
    CERTIFICATE-----\r\n"
kind: ConfigMap
metadata:
  creationTimestamp: "2025-07-29T19:00:31Z"
  name: registry-config
  namespace: openshift-config
  resourceVersion: "43942"
  uid: 5ed426ec-7a38-4106-a051-552666423e86
```

8.  Patch the image cluster configuration with the local Quay mirror registry info

```
$ oc patch image.config.openshift.io/cluster --patch '{"spec":{"additionalTrustedCA":{"name":"registry-config"}}}' --type=merge

$ oc get image.config.openshift.io/cluster -o yaml
apiVersion: config.openshift.io/v1
kind: Image
metadata:
  annotations:
    include.release.openshift.io/ibm-cloud-managed: "true"
    include.release.openshift.io/self-managed-high-availability: "true"
    release.openshift.io/create-only: "true"
  creationTimestamp: "2025-07-29T17:34:40Z"
  generation: 2
  name: cluster
  ownerReferences:
  - apiVersion: config.openshift.io/v1
    kind: ClusterVersion
    name: version
    uid: b06fc08d-fa54-4aba-a8d1-94e5067e18d2
  resourceVersion: "49287"
  uid: 6e32b798-1372-4fec-ac72-c61b97863183
spec:
  additionalTrustedCA:
    name: registry-config
```

9.  Disable all the operators in the `UI - Administration -> Cluster Settings -> Configuration -> OperatorHub -> Cluster -> Sources`

https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html-single/disconnected_environments/index#olm-restricted-networks-operatorhub_olm-restricted-networks

```
oc patch OperatorHub cluster --type json \
    -p '[{"op": "add", "path": "/spec/disableAllDefaultSources", "value": true}]'
```

10.  Apply the IDMS and ITMS files ... note that there are 2 `signature-configmap` files, only apply 1 of them.  The `updateService` file should only be applied to an OpenShift cluster that is hosting the `OpenShift Update Service`

```
$ pwd
/grogu/syangsao/oc-mirror/mirror1/working-dir/cluster-resources

[syangsao@grogu cluster-resources]$ ll
total 40
-rwxrwxrwx+ 1 syangsao syangsao  428 Jul 26 08:09 cc-redhat-operator-index-v4-19.yaml
-rwxrwxrwx+ 1 syangsao syangsao  430 Jul 26 08:09 cs-redhat-operator-index-v4-19.yaml
-rwxrwxrwx+ 1 syangsao syangsao 1071 Jul 26 08:09 idms-oc-mirror.yaml
-rwxrwxrwx+ 1 syangsao syangsao  830 Jul 26 08:09 itms-oc-mirror.yaml
-rwxrwxrwx+ 1 syangsao syangsao 5359 Jul 26 08:09 signature-configmap.json
-rwxrwxrwx+ 1 syangsao syangsao 5344 Jul 26 08:09 signature-configmap.yaml
-rwxrwxrwx+ 1 syangsao syangsao  463 Jul 26 08:09 updateService.yaml

$ oc apply -f /grogu/syangsao/oc-mirror/mirror1/working-dir/cluster-resources
clustercatalog.olm.operatorframework.io/cc-redhat-operator-index-v4-19 created
catalogsource.operators.coreos.com/cs-redhat-operator-index-v4-19 created
imagedigestmirrorset.config.openshift.io/idms-release-0 created
imagedigestmirrorset.config.openshift.io/idms-operator-0 created
imagetagmirrorset.config.openshift.io/itms-generic-0 created
imagetagmirrorset.config.openshift.io/itms-release-0 created
configmap/mirrored-release-signatures created
configmap/mirrored-release-signatures unchanged
error: resource mapping not found for name: "update-service-oc-mirror" namespace: "" from "cluster-resources/updateService.yaml": no matches for kind "UpdateService" in version "updateservice.operator.openshift.io/v1"
ensure CRDs are installed first
```

11.  Verification that the image sets for both the operator and releases are installed, along with the catalogsource pointed to the local Quay repo

```
$ oc get imagedigestmirrorset
NAME              AGE
idms-operator-0   76s
idms-release-0    76s

$ oc get imagedigestmirrorset -o jsonpath='{.items[?(@.metadata.annotations.createdBy=="oc-mirror v2")].metadata.name}'
idms-operator-0 
idms-release-0

$ oc get imagetagmirrorset
NAME             AGE
itms-generic-0   90s
itms-release-0   90s

$ oc get imagetagmirrorset -o jsonpath='{.items[?(@.metadata.annotations.createdBy=="oc-mirror v2")].metadata.name}'
itms-generic-0 
itms-release-0

$ oc get catalogsource -n openshift-marketplace
NAME                             DISPLAY   TYPE   PUBLISHER   AGE
cs-redhat-operator-index-v4-19             grpc               106s

$ oc get catalogsource -o jsonpath='{.items[?(@.metadata.annotations.createdBy=="oc-mirror v2")].metadata.name}'

$ oc get clustercatalog
NAME                             LASTUNPACKED   SERVING   AGE
cc-redhat-operator-index-v4-19                            118s
openshift-certified-operators    53s            True      133m
openshift-community-operators    40s            True      133m
openshift-redhat-marketplace     25s            True      133m
openshift-redhat-operators       73s            True      133m

$ oc get clustercatalog -o jsonpath='{.items[?(@.metadata.annotations.createdBy=="oc-mirror v2")].metadata.name}'
```

12.  Test the installation of the Operator and verify the release images are being used by scaling up a node
