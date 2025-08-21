For this 3rd step, we'll be installing `OSUS` in a disconnected environment, following this procedure

https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html-single/updating_clusters/index#update-service-overview_updating-restricted-network-cluster-osus

```
The OpenShift Update Service (OSUS) provides update recommendations to OpenShift Container Platform clusters. Red Hat publicly hosts the OpenShift Update Service, and clusters in a connected environment can connect to the service through public APIs to retrieve update recommendations.  However, clusters in a disconnected environment cannot access these public APIs to retrieve update information. To have a similar update experience in a disconnected environment, you can install and configure the OpenShift Update Service so that it is available within the disconnected environment. 
```

1.  Configure a ConfigMap

```
$ cat registry-config.yaml 
apiVersion: v1
data:
  updateservice-registry: "-----BEGIN CERTIFICATE-----\r\nMIIECjCCA5CgAwIBAgIQLHdgnL6aZPG5ps/Jf5vgaDAKBggqhkjOPQQDAzBLMQswCQYDVQQGEwJBVDEQMA4GA1UEChMHWmVyb1NTTDEqMCgGA1UEAxMhWmVyb1NTTCBFQ0MgRG9tYWluIFNlY3VyZSBTaXRlIENBMB4XDTI1MDYyNDAwMDAwMFoXDTI1MDkyMjIzNTk1OVowHjEcMBoGA1UEAxMTbWlycm9yLnN5YW5nc2FvLm5ldDBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABEScWYpjITTUYTGm4KF0wGE82xX3/3jzLljBkTLHtX4kKSv7Px1NrEUiSzus1J8QsnzawdPuUBrrOtiCQbJHoj2jggKBMIICfTAfBgNVHSMEGDAWgBQPa+ZLzjlHrvZ+kB558DCRkshfozAdBgNVHQ4EFgQUn05q8O48TJkeeKbrAP+rYmaFaOMwDgYDVR0PAQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMEkGA1UdIARCMEAwNAYLKwYBBAGyMQECAk4wJTAjBggrBgEFBQcCARYXaHR0cHM6Ly9zZWN0aWdvLmNvbS9DUFMwCAYGZ4EMAQIBMIGIBggrBgEFBQcBAQR8MHowSwYIKwYBBQUHMAKGP2h0dHA6Ly96ZXJvc3NsLmNydC5zZWN0aWdvLmNvbS9aZXJvU1NMRUNDRG9tYWluU2VjdXJlU2l0ZUNBLmNydDArBggrBgEFBQcwAYYfaHR0cDovL3plcm9zc2wub2NzcC5zZWN0aWdvLmNvbTCCAQYGCisGAQQB1nkCBAIEgfcEgfQA8gB3AN3cyjSV1+EWBeeVMvrHn/g9HFDf2wA6FBJ2Ciysu8gqAAABl6HR33YAAAQDAEgwRgIhAK/WG5Khs4ws8+vVDM82cxzrxti5DIXDm2at4gwg9iCfAiEAmIlfds84Cmia57WR3KxON12lJ7EXfOLU+NwtdEBdT50AdwAN4fIwK9MNwUBiEgnqVS78R3R8sdfpMO8OQh60fk6qNAAAAZeh0d9DAAAEAwBIMEYCIQDHM74BMlsaFOf7jgK9nCnfbJY+Gfnp3btc37zbg0yiugIhAPVMUEjHYw4TNjravHu91kIeRbEiXoPFsN8OcHlPOJ6UMB4GA1UdEQQXMBWCE21pcnJvci5zeWFuZ3Nhby5uZXQwCgYIKoZIzj0EAwMDaAAwZQIxALGy3YfFE6WuqVN1jaFFBj+9zhw4a0ZEfXAH94ptwHYALDV4kJbJlqWADu5/Wx9f0AIwJlNoyraAPy6ggvvcqRdAhtEaW90zM1LW/yBoKmq2RqU4VT8Qybxv7L+Tq5TSBQVF\r\n-----END
        CERTIFICATE-----\r\n"
  mirror.syangsao.net..8443: "-----BEGIN CERTIFICATE-----\r\nMIIECjCCA5CgAwIBAgIQLHdgnL6aZPG5ps/Jf5vgaDAKBggqhkjOPQQDAzBLMQswCQYDVQQGEwJBVDEQMA4GA1UEChMHWmVyb1NTTDEqMCgGA1UEAxMhWmVyb1NTTCBFQ0MgRG9tYWluIFNlY3VyZSBTaXRlIENBMB4XDTI1MDYyNDAwMDAwMFoXDTI1MDkyMjIzNTk1OVowHjEcMBoGA1UEAxMTbWlycm9yLnN5YW5nc2FvLm5ldDBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABEScWYpjITTUYTGm4KF0wGE82xX3/3jzLljBkTLHtX4kKSv7Px1NrEUiSzus1J8QsnzawdPuUBrrOtiCQbJHoj2jggKBMIICfTAfBgNVHSMEGDAWgBQPa+ZLzjlHrvZ+kB558DCRkshfozAdBgNVHQ4EFgQUn05q8O48TJkeeKbrAP+rYmaFaOMwDgYDVR0PAQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMEkGA1UdIARCMEAwNAYLKwYBBAGyMQECAk4wJTAjBggrBgEFBQcCARYXaHR0cHM6Ly9zZWN0aWdvLmNvbS9DUFMwCAYGZ4EMAQIBMIGIBggrBgEFBQcBAQR8MHowSwYIKwYBBQUHMAKGP2h0dHA6Ly96ZXJvc3NsLmNydC5zZWN0aWdvLmNvbS9aZXJvU1NMRUNDRG9tYWluU2VjdXJlU2l0ZUNBLmNydDArBggrBgEFBQcwAYYfaHR0cDovL3plcm9zc2wub2NzcC5zZWN0aWdvLmNvbTCCAQYGCisGAQQB1nkCBAIEgfcEgfQA8gB3AN3cyjSV1+EWBeeVMvrHn/g9HFDf2wA6FBJ2Ciysu8gqAAABl6HR33YAAAQDAEgwRgIhAK/WG5Khs4ws8+vVDM82cxzrxti5DIXDm2at4gwg9iCfAiEAmIlfds84Cmia57WR3KxON12lJ7EXfOLU+NwtdEBdT50AdwAN4fIwK9MNwUBiEgnqVS78R3R8sdfpMO8OQh60fk6qNAAAAZeh0d9DAAAEAwBIMEYCIQDHM74BMlsaFOf7jgK9nCnfbJY+Gfnp3btc37zbg0yiugIhAPVMUEjHYw4TNjravHu91kIeRbEiXoPFsN8OcHlPOJ6UMB4GA1UdEQQXMBWCE21pcnJvci5zeWFuZ3Nhby5uZXQwCgYIKoZIzj0EAwMDaAAwZQIxALGy3YfFE6WuqVN1jaFFBj+9zhw4a0ZEfXAH94ptwHYALDV4kJbJlqWADu5/Wx9f0AIwJlNoyraAPy6ggvvcqRdAhtEaW90zM1LW/yBoKmq2RqU4VT8Qybxv7L+Tq5TSBQVF\r\n-----END
    CERTIFICATE-----\r\n"
kind: ConfigMap
metadata:
  creationTimestamp: "2025-07-29T19:00:31Z"
  name: registry-config
  namespace: openshift-config
  resourceVersion: "755441"
  uid: 5ed426ec-7a38-4106-a051-552666423e86

$ oc create -f registry-config.yaml -n openshift-config
```

2.  Configure additional trusts [2] and update the spec section

```
$ oc edit image.config.openshift.io cluster
[...]
spec:
  additionalTrustedCA:
    name: registry-config
```

3.  Configure OpenShift Update Service [3]

```
$ NAMESPACE=openshift-update-service
$ NAME=service
$ RELEASE_IMAGES=mirror.syangsao.net:8443/ocp4/openshift/release-images
$ GRAPH_DATA_IMAGE=mirror.syangsao.net:8443/ocp4/openshift/graph-image:latest

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
updateservice.updateservice.operator.openshift.io/service created
```

If this issue occurs, you will need 

```
$ oc get pods -n openshift-update-service
NAME                                     READY   STATUS    RESTARTS   AGE
graph-data-tag-digest                    1/1     Running   0          3m54s
service-76d8f6586f-chgmr                 0/2     Running   0          3m53s
service-8464c79b64-5bchq                 0/2     Running   0          3m51s
service-8464c79b64-x6xzq                 0/2     Running   0          3m50s
updateservice-operator-7b7dcbb48-h6xvl   1/1     Running   0          18h

$ oc logs service-8464c79b64-5bchq
Defaulted container "graph-builder" out of: graph-builder, policy-engine, graph-data (init)
[2025-07-31T19:37:37Z INFO  graph_builder] application settings:
    AppSettings {
        address: ::,
        credentials_path: None,
        mandatory_client_parameters: {},
        manifestref_key: "io.openshift.upgrades.graph.release.manifestref",
[...]
[2025-07-31T19:37:37Z DEBUG cincinnati::plugins::internal::graph_builder::release_scrape_dockerv2::registry] checking namespaced credentials for mirror.syangsao.net:8443/ocp4/openshift/release-images
[2025-07-31T19:37:37Z DEBUG cincinnati::plugins::internal::graph_builder::release_scrape_dockerv2::registry] checking namespaced credentials for mirror.syangsao.net:8443/ocp4/openshift
[2025-07-31T19:37:37Z DEBUG cincinnati::plugins::internal::graph_builder::release_scrape_dockerv2::registry] checking namespaced credentials for mirror.syangsao.net:8443/ocp4
[2025-07-31T19:37:37Z DEBUG cincinnati::plugins::internal::graph_builder::release_scrape_dockerv2::registry] getting credentials for mirror.syangsao.net:8443
[2025-07-31T19:37:37Z INFO  graph_builder::graph] graph update triggered
[2025-07-31T19:37:37Z TRACE cincinnati::plugins] Running next plugin 'release-scrape-dockerv2'
[2025-07-31T19:37:37Z DEBUG cincinnati::plugins::internal::graph_builder::commons] Adding /etc/pki/ca-trust/extracted/pem/..2025_07_31_19_37_22.2019677119/tls-ca-bundle.pem to certificates
[2025-07-31T19:37:37Z DEBUG cincinnati::plugins::internal::graph_builder::commons] unable to process certificate ca-bundle.trust.crt: builder error: error:0909006C:PEM routines:get_name:no start line:crypto/pem/pem_lib.c:745:Expecting: CERTIFICATE
[2025-07-31T19:37:37Z ERROR graph_builder::graph] failed to fetch all release metadata from mirror.syangsao.net:8443/ocp4/openshift/release-images
[2025-07-31T19:37:37Z ERROR graph_builder::graph] http transport error: error sending request for url (https://mirror.syangsao.net:8443/v2/): error trying to connect: error:1416F086:SSL routines:tls_process_server_certificate:certificate verify failed:ssl/statem/statem_clnt.c:1915: (unable to get local issuer certificate)
[2025-07-31T19:37:37Z ERROR graph_builder::graph] error sending request for url (https://mirror.syangsao.net:8443/v2/): error trying to connect: error:1416F086:SSL routines:tls_process_server_certificate:certificate verify failed:ssl/statem/statem_clnt.c:1915: (unable to get local issuer certificate)
[2025-07-31T19:37:37Z ERROR graph_builder::graph] error trying to connect: error:1416F086:SSL routines:tls_process_server_certificate:certificate verify failed:ssl/statem/statem_clnt.c:1915: (unable to get local issuer certificate)
[2025-07-31T19:37:37Z ERROR graph_builder::graph] error:1416F086:SSL routines:tls_process_server_certificate:certificate verify failed:ssl/statem/statem_clnt.c:1915: (unable to get local issuer certificate)
[2025-07-31T19:37:37Z ERROR graph_builder::graph] error:1416F086:SSL routines:tls_process_server_certificate:certificate verify failed:ssl/statem/statem_clnt.c:1915:
```

You might need to make sure the certificate on the registry is a full-chain certificate

https://access.redhat.com/solutions/7040684

4.  Run the following command on the `OSUS` cluster to verify that the graph is readily available.

```
[syangsao@grogu ocp416-leia]$ NAMESPACE=openshift-update-service
[syangsao@grogu ocp416-leia]$ NAME=service
[syangsao@grogu ocp416-leia]$ POLICY_ENGINE_GRAPH_URI="$(oc -n "${NAMESPACE}" get -o jsonpath='{.status.policyEngineURI}/api/upgrades_info/v1/graph{"\n"}' updateservice "${NAME}")"
[syangsao@grogu ocp416-leia]$ PATCH="{\"spec\":{\"upstream\":\"${POLICY_ENGINE_GRAPH_URI}\"}}"
[syangsao@grogu ocp416-leia]$ $(oc -n "${NAMESPACE}" get -o jsonpath='{.status.policyEngineURI}/api/upgrades_info/v1/graph{"\n"}' updateservice "${NAME}")
```

There should be an output similar to this that you will need to update on the disconnected cluster.

```
https://service-route-openshift-update-service.apps.leia.syangsao.net/api/upgrades_info/v1/graph
```

Also, make sure the OSUS target's `*.apps` address is also using a valid certificate, which may need to be updated as well for the disconnected cluster to successfully connect to it..

```
$ curl -kvv https://service-route-openshift-update-service.apps.leia.syangsao.net/api/upgrades_info/v1/graph
...
* Server certificate:
*  subject: CN=*.apps.leia.syangsao.net
*  start date: Jul 31 00:00:00 2025 GMT
*  expire date: Oct 29 23:59:59 2025 GMT
*  issuer: C=AT; O=ZeroSSL; CN=ZeroSSL ECC Domain Secure Site CA
*  SSL certificate verify ok.
* TLSv1.2 (OUT), TLS header, Unknown (23):
> GET /api/upgrades_info/v1/graph HTTP/1.1
> Host: service-route-openshift-update-service.apps.leia.syangsao.net
> User-Agent: curl/7.76.1
> Accept: */*
[...]
```

5.  Verified that OSUS link is working on the client after changing the `Upstream` address for the graph.  

The `oc adm upgrade` while verifying it with some of the new command options

https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html/updating_clusters/performing-a-cluster-update#update-upgrading-oc-adm-upgrade-recommend_updating-cluster-cli

```
$ oc adm upgrade
[syangsao@grogu ~]$ oc adm upgrade
Cluster version is 4.19.1

Upstream: https://service-route-openshift-update-service.apps.leia.syangsao.net/api/upgrades_info/v1/graph
Channel: stable-4.19 (available channels: candidate-4.19, candidate-4.20, fast-4.19, stable-4.19)

Recommended updates:

  VERSION     IMAGE
  4.19.5      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:bc79be35e8b8a3719a3e16c91b64e5945c6c4ff1a9c9d0816339f14e2b004385
  4.19.4      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:8153a8c010b292c0c4ca7d8b4ca13ebeb634d449982c66568764511c736281b8
  4.19.3      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:0b44c4b526b4743e744cb989c6fc768fdfd9ac9abffc8f43a014bb90b7bf522d
  4.19.2      mirror.syangsao.net:8443/ocp4/openshift/release-images@sha256:1293f5ccad2a2776241344faecaf7320f60ee91882df4e24b309f3a7cefc04be

$ export OC_ENABLE_CMD_UPGRADE_RECOMMEND=true
$ oc adm upgrade recommend

Upstream update service: https://service-route-openshift-update-service.apps.leia.syangsao.net/api/upgrades_info/v1/graph
Channel: stable-4.19 (available channels: candidate-4.19, candidate-4.20, fast-4.19, stable-4.19)

Updates to 4.19:
  VERSION     ISSUES
  4.19.5      no known issues relevant to this cluster
  4.19.4      no known issues relevant to this cluster
And 2 older 4.19 updates you can see with '--show-outdated-releases' or '--version VERSION'.

$ export OC_ENABLE_CMD_UPGRADE_STATUS=true
$ oc adm upgrade status
```