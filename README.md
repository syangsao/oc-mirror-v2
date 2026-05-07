# OpenShift 4.19 Disconnected Setup with oc-mirror v2

This repository documents the process of setting up an OpenShift v4.19 cluster in **disconnected mode** using `oc-mirror v2`. The local registry used for this configuration is a **Quay mirror-registry** running on RHEL, installed following [Red Hat's guide](https://www.redhat.com/en/blog/introducing-mirror-registry-for-red-hat-openshift).

## Prerequisites

- `oc-mirror` CLI tool installed (follow [Red Hat's documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html-single/disconnected_environments/index))
- A local Quay mirror-registry configured with credentials
- A connected OpenShift 4.19 cluster (for Steps 2–4)
- Root access to the mirror-registry host for certificate management

## Table of Contents

| Step | Description | File |
|------|-------------|------|
| 1 | Mirror release and operator images from Red Hat to a local registry | [steps/01-mirror-images.md](steps/01-mirror-images.md) |
| 2 | Convert a connected OpenShift cluster to disconnected mode | [steps/02-convert-to-disconnected.md](steps/02-convert-to-disconnected.md) |
| 3 | Install OpenShift Update Service (OSUS) in a disconnected environment | [steps/03-install-osus.md](steps/03-install-osus.md) |
| 4 | Upgrade the cluster using the local mirror-registry | [steps/04-upgrade-cluster.md](steps/04-upgrade-cluster.md) |
| 5 | Delete unwanted images from the local registry | [steps/05-cleanup-registry.md](steps/05-cleanup-registry.md) |
| 6 | **TODO** Build an `install-config.yaml` to boot from the local registry | _coming soon_ |

## Placeholders

Throughout this guide, the following placeholders are used. Replace them with your actual values:

| Placeholder | Description |
|-------------|-------------|
| `<CACHE_DIR>` | Path to the oc-mirror cache directory (e.g., `/var/lib/oc-mirror`) |
| `<MIRROR_REGISTRY>` | Your local Quay registry address (e.g., `mirror.example.net:8443`) |
| `<OCMIRROR_PREFIX>` | Repository prefix on the mirror (e.g., `ocp4`) |
| `<OSUS_ROUTE>` | OSUS route URL (e.g., `https://osus.apps.example.net/api/upgrades_info/v1/graph`) |
| `<REGISTRY_CERT>` | Path to your registry's PEM certificate file |

## License

This work is licensed under the [MIT License](LICENSE.md).

## Contributing

Found an issue or have an improvement? Feel free to open a Pull Request or file an Issue.
