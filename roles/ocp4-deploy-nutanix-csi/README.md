ocp4-deploy-nutanix-csi
=========

This Ansible Role will deploy the Nutanix CSI Operator, an Operand Instance, the StorageClasses and MachineConfigs required to use the underlying Nutanix storage layer in an OpenShift 4 cluster.

Tested with OpenShift 4.8 and 4.9.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

```yaml
---
operator_namespace: ntnx-system
secret_name: ntnx-secret

### Nutanix Volumes StorageClass, iSCSI based, no RWX
nutanix_csi_deploy_volumes_storageclass: true
nutanix_csi_prism_hostname: "{{ nutanix_prism_hostname }}"
nutanix_csi_prism_port: "{{ nutanix_prism_port }}"
nutanix_csi_prism_username: "{{ nutanix_prism_username }}"
nutanix_csi_prism_password: "{{ nutanix_prism_password }}"
nutanix_csi_volumes_storage_container_name: "{{ nutanix_prism_vm_storage_container_name }}"
nutanix_csi_volumes_dataservices_ip: 1.2.3.4
nutanix_csi_volumes_filesystem_type: ext4
nutanix_csi_volumes_flashmode: false

## nutanix_csi_deploy_iscsid_machineconfig is required to enable the iSCSId SystemD Service to bind the PVCs to the Pods
nutanix_csi_deploy_iscsid_machineconfig: true

### Nutanix Files StorageClass, NFS Based, RWO and RWX supported
#### Assumes the NFS Store has All Users Squashed to nobody:nobody (65534:65534)
nutanix_csi_deploy_files_storageclass: false
nutanix_csi_files_nfs_server: 5.6.7.8
nutanix_csi_files_nfs_path: somePath
```

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
