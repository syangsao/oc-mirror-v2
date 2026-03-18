For this 3rd step, we'll be installing `OSUS` in a disconnected environment, following this procedure

https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html-single/updating_clusters/index#update-service-overview_updating-restricted-network-cluster-osus

```
The OpenShift Update Service (OSUS) provides update recommendations to OpenShift Container Platform clusters. Red Hat publicly hosts the OpenShift Update Service, and clusters in a connected environment can connect to the service through public APIs to retrieve update recommendations.  However, clusters in a disconnected environment cannot access these public APIs to retrieve update information. To have a similar update experience in a disconnected environment, you can install and configure the OpenShift Update Service so that it is available within the disconnected environment. 
```

1.  Configure a ConfigMap using the local registry's `FULL CHAIN` certificate

```
[syangsao@grogu files]$ cat mirror-syangsao-net-chain-1.pem
-----BEGIN CERTIFICATE-----
MIID+zCCA4KgAwIBAgIQQjmlAzHG0/Ip/FPTiTziKDAKBggqhkjOPQQDAzBLMQsw
CQYDVQQGEwJBVDEQMA4GA1UEChMHWmVyb1NTTDEqMCgGA1UEAxMhWmVyb1NTTCBF
Q0MgRG9tYWluIFNlY3VyZSBTaXRlIENBMB4XDTI2MDEyMDAwMDAwMFoXDTI2MDQy
MDIzNTk1OVowHjEcMBoGA1UEAxMTbWlycm9yLnN5YW5nc2FvLm5ldDBZMBMGByqG
SM49AgEGCCqGSM49AwEHA0IABEScWYpjITTUYTGm4KF0wGE82xX3/3jzLljBkTLH
tX4kKSv7Px1NrEUiSzus1J8QsnzawdPuUBrrOtiCQbJHoj2jggJzMIICbzAfBgNV
HSMEGDAWgBQPa+ZLzjlHrvZ+kB558DCRkshfozAdBgNVHQ4EFgQUn05q8O48TJke
eKbrAP+rYmaFaOMwDgYDVR0PAQH/BAQDAgeAMAwGA1UdEwEB/wQCMAAwEwYDVR0l
BAwwCgYIKwYBBQUHAwEwSQYDVR0gBEIwQDA0BgsrBgEEAbIxAQICTjAlMCMGCCsG
AQUFBwIBFhdodHRwczovL3NlY3RpZ28uY29tL0NQUzAIBgZngQwBAgEwgYgGCCsG
AQUFBwEBBHwwejBLBggrBgEFBQcwAoY/aHR0cDovL3plcm9zc2wuY3J0LnNlY3Rp
Z28uY29tL1plcm9TU0xFQ0NEb21haW5TZWN1cmVTaXRlQ0EuY3J0MCsGCCsGAQUF
BzABhh9odHRwOi8vemVyb3NzbC5vY3NwLnNlY3RpZ28uY29tMIIBAgYKKwYBBAHW
eQIEAgSB8wSB8ADuAHUADleUvPOuqT4zGyyZB7P3kN+bwj1xMiXdIaklrGHFTiEA
AAGb3BiOMAAABAMARjBEAiB2Sour9yVN5KzyYdfzOYhpj3OPNfcsHHoS2KH2NfJ2
RQIgTSB/DABbaxY4qxcrwpH/MB7RTAQWgF54BIHDxtkpoPgAdQAWgy2r8KklDw/w
OqVF/8i/yCPQh0v2BCkn+OcfMxP1+gAAAZvcGI46AAAEAwBGMEQCICOAu1jDZRme
RDeF4rVQznFPk3+fTPffo+Vz/KH3bFNMAiBDovSIsNd9Qq5HlG+HZ15VKSZaOB8b
ZFGZWcbr+iBvOzAeBgNVHREEFzAVghNtaXJyb3Iuc3lhbmdzYW8ubmV0MAoGCCqG
SM49BAMDA2cAMGQCMBbn7hDUafXFUKjNjGvfwUu+vVurzICx0mYglIuvMkv+VAkk
cPKuUts7LhzxVxf80QIwCd3I2u27tQ/ryaK3wSxZNOmtc/l3EjwkFpxi72VLcUhB
RnpjH5iGPn2urYwkKeyz
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIDhTCCAwygAwIBAgIQI7dt48G7KxpRlh4I6rdk6DAKBggqhkjOPQQDAzCBiDEL
MAkGA1UEBhMCVVMxEzARBgNVBAgTCk5ldyBKZXJzZXkxFDASBgNVBAcTC0plcnNl
eSBDaXR5MR4wHAYDVQQKExVUaGUgVVNFUlRSVVNUIE5ldHdvcmsxLjAsBgNVBAMT
JVVTRVJUcnVzdCBFQ0MgQ2VydGlmaWNhdGlvbiBBdXRob3JpdHkwHhcNMjAwMTMw
MDAwMDAwWhcNMzAwMTI5MjM1OTU5WjBLMQswCQYDVQQGEwJBVDEQMA4GA1UEChMH
WmVyb1NTTDEqMCgGA1UEAxMhWmVyb1NTTCBFQ0MgRG9tYWluIFNlY3VyZSBTaXRl
IENBMHYwEAYHKoZIzj0CAQYFK4EEACIDYgAENkFhFytTJe2qypTk1tpIV+9QuoRk
gte7BRvWHwYk9qUznYzn8QtVaGOCMBBfjWXsqqivl8q1hs4wAYl03uNOXgFu7iZ7
zFP6I6T3RB0+TR5fZqathfby47yOCZiAJI4go4IBdTCCAXEwHwYDVR0jBBgwFoAU
OuEJhtTPGcKWdnRJdtzgNcZjY5owHQYDVR0OBBYEFA9r5kvOOUeu9n6QHnnwMJGS
yF+jMA4GA1UdDwEB/wQEAwIBhjASBgNVHRMBAf8ECDAGAQH/AgEAMB0GA1UdJQQW
MBQGCCsGAQUFBwMBBggrBgEFBQcDAjAiBgNVHSAEGzAZMA0GCysGAQQBsjEBAgJO
MAgGBmeBDAECATBQBgNVHR8ESTBHMEWgQ6BBhj9odHRwOi8vY3JsLnVzZXJ0cnVz
dC5jb20vVVNFUlRydXN0RUNDQ2VydGlmaWNhdGlvbkF1dGhvcml0eS5jcmwwdgYI
KwYBBQUHAQEEajBoMD8GCCsGAQUFBzAChjNodHRwOi8vY3J0LnVzZXJ0cnVzdC5j
b20vVVNFUlRydXN0RUNDQWRkVHJ1c3RDQS5jcnQwJQYIKwYBBQUHMAGGGWh0dHA6
Ly9vY3NwLnVzZXJ0cnVzdC5jb20wCgYIKoZIzj0EAwMDZwAwZAIwJHBUDwHJQN3I
VNltVMrICMqYQ3TYP/TXqV9t8mG5cAomG2MwqIsxnL937Gewf6WIAjAlrauksO6N
UuDdDXyd330druJcZJx0+H5j5cFOYBaGsKdeGW7sCMaR2PsDFKGllas=
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIICjzCCAhWgAwIBAgIQXIuZxVqUxdJxVt7NiYDMJjAKBggqhkjOPQQDAzCBiDEL
MAkGA1UEBhMCVVMxEzARBgNVBAgTCk5ldyBKZXJzZXkxFDASBgNVBAcTC0plcnNl
eSBDaXR5MR4wHAYDVQQKExVUaGUgVVNFUlRSVVNUIE5ldHdvcmsxLjAsBgNVBAMT
JVVTRVJUcnVzdCBFQ0MgQ2VydGlmaWNhdGlvbiBBdXRob3JpdHkwHhcNMTAwMjAx
MDAwMDAwWhcNMzgwMTE4MjM1OTU5WjCBiDELMAkGA1UEBhMCVVMxEzARBgNVBAgT
Ck5ldyBKZXJzZXkxFDASBgNVBAcTC0plcnNleSBDaXR5MR4wHAYDVQQKExVUaGUg
VVNFUlRSVVNUIE5ldHdvcmsxLjAsBgNVBAMTJVVTRVJUcnVzdCBFQ0MgQ2VydGlm
aWNhdGlvbiBBdXRob3JpdHkwdjAQBgcqhkjOPQIBBgUrgQQAIgNiAAQarFRaqflo
I+d61SRvU8Za2EurxtW20eZzca7dnNYMYf3boIkDuAUU7FfO7l0/4iGzzvfUinng
o4N+LZfQYcTxmdwlkWOrfzCjtHDix6EznPO/LlxTsV+zfTJ/ijTjeXmjQjBAMB0G
A1UdDgQWBBQ64QmG1M8ZwpZ2dEl23OA1xmNjmjAOBgNVHQ8BAf8EBAMCAQYwDwYD
VR0TAQH/BAUwAwEB/zAKBggqhkjOPQQDAwNoADBlAjA2Z6EWCNzklwBBHU6+4WMB
zzuqQhFkoJ2UOQIReVx7Hfpkue4WQrO/isIJxOzksU0CMQDpKmFHjFJKS04YcPbW
RNZu9YO6bVi9JNlWSOrvxKJGgYhqOkbRqZtNyWHa0V1Xahg=
-----END CERTIFICATE-----

$ oc create configmap registry-config --from-file=mirror.syangsao.net..8443=mirror-syangsao-net.pem -n openshift-config
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
$ NAMESPACE=openshift-update-service
$ NAME=service
$ POLICY_ENGINE_GRAPH_URI="$(oc -n "${NAMESPACE}" get -o jsonpath='{.status.policyEngineURI}/api/upgrades_info/v1/graph{"\n"}' updateservice "${NAME}")"
$ PATCH="{\"spec\":{\"upstream\":\"${POLICY_ENGINE_GRAPH_URI}\"}}"
$ $(oc -n "${NAMESPACE}" get -o jsonpath='{.status.policyEngineURI}/api/upgrades_info/v1/graph{"\n"}' updateservice "${NAME}")
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
