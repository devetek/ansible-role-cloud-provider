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


Prerequisites
------------

This role requires the `google.cloud` Ansible collection to interact with Google Cloud services. Unfortunately, the official `google.cloud` collection has not been updated to support the latest Google Cloud Platform API. Therefore, it is necessary to route the target dependency from a custom Git URL. Install the collection using the following steps:

### 1. Install `google.cloud` with CLI or file `requirements.yml`

- CLI:
```sh
ansible-galaxy collection install git+https://github.com/prakasa1904/google.cloud.git,master
```

- requirements.yml:
```sh
collections:
  - name: https://github.com/prakasa1904/google.cloud.git
    type: git
    version: master
```
Execute command: `ansible-galaxy install -r requirements.yml`

### 2. Verify Installation

After installation, you can verify that the collection is correctly installed by running:

```sh
ansible-galaxy collection list google.cloud
```

This command will display the installed version and confirm that it's sourced from the custom repository.

Role Variables
--------------

List of variables in ansible-role-cloud-provider:

```sh
---
cloud_provider: "gcp" # gcp | aws | proxmox
cloud_provider_auth: {} # depend on provider
cloud_provider_resource_type: # array of resource type
  - "global-forwarding-rule"
cloud_provider_resource_detail: {} # dictionary resource attribute detail
state: "present" # present | absent
```

Example Playbook
----------------

```sh
---
# includes all dependencies
- name: Manage a Global Address
  ansible.builtin.import_playbook: global-address.yaml
  vars:
    with_output: true
    include_state: "present"

- name: Manage a HTTPS Proxy (Backend URL Map Included)
  ansible.builtin.import_playbook: target-https-proxy.yaml
  vars:
    with_output: true
    include_state: "present"
# includes all dependencies

- name: Manage Global Forwarding Rule
  hosts: localhost
  gather_facts: false
  
  vars:
    with_output: true
    cloud_provider: "gcp"
    cloud_provider_resource_type:
      - "global-forwarding-rule"
    cloud_provider_auth:
      project_id: "terpusat"
      auth_kind: "serviceaccount"
      service_acount_token: "../../credential.json"
    cloud_provider_resource_detail:
      global_forwarding_rule:
        name: "global-forwarding-rule"
        description: "Global Forwarding Rule"
        ip_address: "{{ output_global_address['external-global-address'].address }}"
        port_range: "443"
        target: "{{ output_target_https_proxy['https-proxy'].selfLink }}"


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
[hetzner.hcloud]: https://github.com/ansible-collections/hetzner.hcloud
[community.digitalocean]: https://github.com/ansible-collections/community.digitalocean