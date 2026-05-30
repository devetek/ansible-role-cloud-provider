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
cloud_provider: "gcp" # gcp | idcloudhost
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

### IDCloudHost VM example

```yaml
---
- name: Create an IDCloudHost VM
  hosts: localhost
  gather_facts: false

  vars:
    with_output: true
    cloud_provider: "idcloudhost"
    cloud_provider_resource_type:
      - "vm"
    cloud_provider_auth:
      token: "{{ idcloudhost_api_token }}"
      # Optional. If omitted, IDCloudHost default location is used.
      region: "jkt01"
    cloud_provider_resource_detail:
      vm:
        name: "dpanel-idcloudhost-01"
        region: "jkt01"
        os_name: "ubuntu"
        os_version: "22.04-lts"
        disks: 20
        vcpu: 2
        ram: 2048
        username: "dpanel"
        password: "{{ idcloudhost_vm_password }}"
        billing_account_id: 12345
        network_uuid: "50341410-9e88-4af2-a21e-b8c898e33e52"
        reserve_public_ip: true
        # Optional. HTTP read timeout for the initial create request.
        create_timeout: 120
        # Optional. The role waits up to 30 minutes by default for async
        # IDCloudHost VM creation to become visible in the VM list.
        wait_timeout: 1800
        wait_delay: 10
        # Optional. If IDCloudHost returns a retryable rejected create response,
        # the role reconciles against the VM list before failing.
        reconcile_timeout: 1800
        reconcile_delay: 10
        cloud_init:
          package_update: true

  roles:
    - role: dpanel.cloud-provider
```

The role writes VM creation output to `output_idcloudhost_vm` and `output_vm`
when `with_output: true`. Each output item is keyed by VM name and includes the
provider response, including the IDCloudHost `uuid` that dPanel can store as the
provider instance reference. When `reserve_public_ip` is enabled and IDCloudHost
returns an assigned public address from `/network/ip_addresses`, the role enriches
the VM output with `public_ip` and `public_ipv4` so dPanel can run setup against
the reachable address.

For IDCloudHost VM creation, the role treats a successful create request as
asynchronous. It polls the IDCloudHost VM list until the new VM is visible by
UUID or name, using `vm.wait_timeout`/`cloud_provider_auth.vm_wait_timeout`
with a default of `1800` seconds and `vm.wait_delay`/`cloud_provider_auth.vm_wait_delay`
with a default of `10` seconds. If the VM is still not visible when the timeout
expires, the role fails the task.

The initial create request uses `vm.create_timeout`/`cloud_provider_auth.vm_create_timeout`
with a default of `120` seconds. If IDCloudHost returns a retryable rejected
create response (`400`, `409`, `422`, `500`) or Ansible reports `status=-1`
after a transport timeout, the role also reconciles against the VM list by UUID
or name for up to `vm.reconcile_timeout`/`cloud_provider_auth.vm_reconcile_timeout`
seconds before failing. If the VM is found, the role continues with the provider
VM output instead of treating the create response as fatal. Authentication and
authorization failures are not retried.

License
-------

GNU General Public License v3.0 or later

Author Information
------------------

[Nedya Prakasa]. Role for [dPanel].

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
