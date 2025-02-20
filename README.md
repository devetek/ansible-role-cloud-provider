[WIP] Ansible role: Cloud Provider Creators (GCP / AWS / Azure)
=========

This Ansible role is designed to automate the provisioning, configuration, and management of cloud resources across various cloud providers. It simplifies and streamlines tasks such as creating, updating, and deleting resources like virtual machines, storage, networking, and more.

Key Features:
- **Provider Agnostic**: Supports multiple cloud providers (e.g., AWS, Azure, Google Cloud, etc.), allowing users to manage resources across different platforms using a unified approach.

- **Declarative Resource Management**: Define the desired state of cloud resources (e.g., VM configurations, security groups, network interfaces) and let Ansible ensure that resources are maintained accordingly.

- **Automation of Common Tasks**: Automates routine cloud resource operations such as creating instances, configuring networking, managing access controls, and setting up storage volumes.

- **Scalability and Flexibility**: Easily adaptable to different use cases, from small environments to large-scale cloud infrastructure, by simply adjusting variables or extending the role.
Idempotent: Ensures that applying the role multiple times doesn’t result in unwanted changes, maintaining consistency across cloud resources.

This role integrates seamlessly into CI/CD pipelines, enabling automated infrastructure management, efficient resource scaling, and consistent environment setup, improving operational efficiency and reducing manual errors.


Requirements
------------

This role requires the google.cloud Ansible collection to interact with Google Cloud services. Install the collection using the following command:

```sh
ansible-galaxy collection install google.cloud
```

Role Variables
--------------

List of variables in ansible-role-cloud-provider:

```sh
---
cloud_provider: "gcp" # gcp | aws | proxmox
cloud_provider_auth: {} # depends to cloud provider
cloud_provider_resource_type: "vm" # resource type creation
cloud_provider_resource_detail: {} # dictionary resource attribute detail
```

Example Playbook
----------------

```sh
---

- name: Create Compute Engine
  hosts: localhost
  gather_facts: false
  
  vars:
    cloud_provider: "gcp"
    cloud_provider_resource_type: "vm"
    cloud_provider_auth:
      project_id: "terpusat"
      auth_kind: "serviceaccount"
      service_acount_token: "/home/devetek/creator/credentials/gcp/sa-development.json"
    cloud_provider_resource_detail:
      # network parameters
      network_name: "network-dpanel-example"
      auto_create_subnetworks: false
      # subnetwork parameters
      subnetwork_name: "subnetwork-dpanel-example"
      region: "asia-southeast2"
      ip_cidr_range: 172.16.0.0/16
      # disk parameters
      disk_name: "disk-dpanel-example"
      source_image: "projects/ubuntu-os-cloud/global/images/family/ubuntu-2204-lts"
      zone: "asia-southeast2-a"
      size_gb: 40
      # external IP (NAT) parameters
      address_name: "address-dpanel-example"
      # compute engine parameters
      vm_name: "vm-dpanel-example"
      machine_type: "n1-standard-1"
      environment: "development"


  roles:
    - role: dpanel.cloud-provider
```

License
-------

[MIT]

Author Information
------------------

[Nedya Prakasa]. Role created for [dPanel].

[dPanel]: https://cloud.terpusat.com/
[Nedya Prakasa]: https://github.com/prakasa1904
[mit]: https://opensource.org/licenses/MIT
[devetek]: https://github.com/devetek
[alicloud]: https://github.com/alibaba/alibaba.alicloud
[amazon.aws]: https://github.com/ansible-collections/amazon.aws
[google.cloud]: https://github.com/ansible-collections/google.cloud
[microsoft.azure]: https://github.com/ansible-collections/azure