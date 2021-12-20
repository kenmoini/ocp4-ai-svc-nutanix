# OpenShift 4 Assisted Installer on Nutanix

This set of resources handles an idempotent way to deploy OpenShift via an Assisted Installer service onto a Nutanix cluster.

## Operations

What this Ansible content will do is the following:

### bootstrap.yaml

- Do preflight for binaries, create asset generation directory, set HTTP Headers & Authentication
- Do preflight checks for supported OpenShift versions on the Assisted Installer Service
- Do preflight checks for the Nutanix environment
- Query the AI Svc, check for existing cluster with the same name
- Set needed facts, or create a new cluster with new SSH keys
- Configure cluster on the AI Svc with deployment specs and ISO Params
- Copy the ISO to the Prism Image Configuration service
- Create VMs via Nutanix's Prism APIs
- Wait for the hosts to report into the AI Svc
- Set Host Names and Roles on the AI Svc
- Set Network VIPs on the AI Svc
- Wait for the hosts to be ready
- [Optional/Dependant/Non-Default] Set Cilium Networking Manifests on the AI Svc
- [Optional/Dependant/Non-Default] Set Calico Networking Manifests on the AI Svc
- Start the cluster installation on the AI Svc
- Wait for the cluster to be fully installed
- Pull cluster credentials from the AI Svc
- Perform cluster role-based post-configuration, roles as defined via the `extra_roles` variable, by default deploying the Nutanix CSI, a Red Matrix login, and a Mario Game.

## Usage

### Clone the Repo

```bash
git clone https://github.com/kenmoini/ocp4-ai-svc-nutanix.git
cd ocp4-ai-svc-nutanix/
```

### Installing Ansible Collections

In order to run this Playbook you'll need to have the needed Ansible Collections already installed - you can do so easily by running the following command:

```bash
ansible-galaxy collection install -r requirements.yml
```

### Modify the Variables files

- Copy `example_vars/cluster-config.yaml` to the working directory, ideally with a prefix of the cluster name - modify as needed
- Modify the other files in `example_vars/` and copy to `vars/` as you see fit, in case you need to add the new cluster to an ACM Hub for instance - at a minimum, you need to copy/modify the `example_vars/assisted-service.yaml` and `example_vars/nutanix-config.yaml` files

```bash
# OCP Cluster Configuration Information
cp example_vars/cluster-config.yaml my-ocp-cluster.cluster-config.yaml
vi my-ocp-cluster.cluster-config.yaml

# Other variable files
cp example_vars/assisted-service.yaml vars/assisted-service.yaml
cp example_vars/nutanix-config.yaml vars/nutanix-config.yaml

```

### Using the Red Hat Console/Cloud hosted Installer Service

Instead of hosting the Assisted Installer Service yourself, you can use the AI Service hosted online by Red Hat: https://console.redhat.com/openshift/assisted-installer/clusters

To do so with this automation:

1. Get an Offline Token: https://access.redhat.com/management/api
2. Modify the `vars/assisted-service.yaml` variable file and define the following:

```yaml
assisted_service_fqdn: api.openshift.com
assisted_service_port: 443
assisted_service_transport: https
assisted_service_authentication: bearer-token
assisted_service_authentication_api_bearer_token: yourOfflineToken
```

*The `bootstrap.yaml` Playbook will swap out the Offline Token for an ephemerial API token.*

3. Make sure the hardware definition of your VMs meets the minimum required by the hosted service

## Running the Playbook

With the needed variables altered, you can run the Playbook with the following command:

```bash
ansible-playbook -e "@my-ocp-cluster.cluster-config.yaml" bootstrap.yaml
```

## Destroying the Cluster

If you are done with the cluster or some error occured you can quickly delete it from the Nutanix environment, the Assisted Installer Service, and the local assets:

```bash
ansible-playbook -e "@my-ocp-cluster.cluster-config.yaml" destroy.yaml
```

## Background Information & Sources

- assisted-service Source: https://github.com/openshift/assisted-service
- assisted-service OnPrem Deployment: https://github.com/sonofspike/assisted-service-onprem
- Podman & Systemd AI Svc Deployment: https://github.com/kenmoini/homelab/blob/main/ansible-collections/deploy-caas-ocp-assisted-installer.yml
- Red Hat Console hosted Assisted Installer Service: https://console.redhat.com/openshift/assisted-installer/clusters

- Last tested with the [Red Hat hosted Assisted Installer](https://console.redhat.com/openshift/assisted-installer/clusters) on 12/19/21
- Last tested offline on 8/10/2021 with:
  - quay.io/ocpmetal/ocp-metal-ui:stable-candidate.10.08.2021-08.28
  - quay.io/ocpmetal/assisted-service:stable-candidate.10.08.2021-08.28
  - quay.io/ocpmetal/postgresql-12-centos7
  - quay.io/coreos/coreos-installer:v0.10.0
- Usable with self-hosted Assisted Installer Service or Red Hat Cloud/Console hosted AI Service