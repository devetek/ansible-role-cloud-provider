## Purpose

This repository contains reusable Ansible tasks for managing cloud resources across multiple cloud providers.

The repository is designed to closely follow each cloud provider's official API and product terminology. Each provider implementation should remain independent and reflect the provider’s native structure.

Downstream projects such as **dPanel** and **dNocs** consume this repository and build abstraction layers on top of it (for example, mapping different provider products into a unified `vm` concept).

---

# Repository Structure

Tasks are organized using the following hierarchy:

```
tasks/
└── <provider>/
    └── <product>/
        └── <state>.yml
```

Example:

```
tasks/
├── aws/
│   ├── ec2/
│   │   ├── present.yml
│   │   └── absent.yml
│   └── vpc/
│
├── tencent/
│   ├── cvm/
│   │   ├── present.yml
│   │   └── absent.yml
│   └── vpc/
│
├── alibabacloud/
│   ├── ecs/
│   │   ├── present.yml
│   │   └── absent.yml
│   └── vpc/
│
└── idcloudhost/
    └── vm/
        ├── present.yml
        └── absent.yml
```

---

# Design Principles

- Follow official cloud provider terminology whenever possible.
- Keep provider implementations isolated from each other.
- Organize tasks strictly by provider and product.
- Use Ansible state-based design (`present.yml`, `absent.yml`).
- Keep tasks small, reusable, and idempotent.
- Avoid duplication across providers.

---

# Adding a New Cloud Provider

When adding a new provider:

1. Create a provider directory:

```
tasks/<provider>/
```

2. Add provider-specific product directories using official naming.

Example:

```
tasks/tencent/cvm/
tasks/tencent/vpc/
tasks/tencent/clb/

tasks/aws/ec2/
tasks/aws/vpc/
```

Do not group multiple products into a single directory.

---

# Product Naming

Always use the **official product name from the cloud provider**.

Examples:

| Provider | Product |
|----------|---------|
| AWS | ec2 |
| Tencent Cloud | cvm |
| Alibaba Cloud | ecs |
| Google Cloud | compute-engine |
| IDCloudHost | vm |

Do NOT normalize or rename provider products into a shared abstraction.

❌ Incorrect:

```
tasks/aws/vm/
tasks/tencent/vm/
```

✅ Correct:

```
tasks/aws/ec2/
tasks/tencent/cvm/
```

---

# Task Naming Convention

This repository follows Ansible state-based conventions.

Preferred filenames:

```
present.yml
absent.yml
```

Avoid action-based filenames:

```
create.yml
delete.yml
update.yml
remove.yml
```

The playbook determines the desired state and includes the correct task file.

---

# Variables

Use provider-specific variables to avoid conflicts.

Examples:

```
aws_region
aws_access_key
aws_secret_key

tencent_secret_id
tencent_secret_key

gcp_project

idcloudhost_api_key
```

Avoid generic variable names shared across providers.

---

# Writing Tasks

Tasks must:

- Be idempotent
- Prefer native Ansible modules over shell/command
- Support check mode where possible
- Return useful registered outputs
- Fail with clear error messages
- Use descriptive task names

Example:

```yaml
- name: Ensure EC2 instance is present
```

Avoid:

```yaml
- name: Call API
```

---

# Reusability

Avoid duplicating logic across providers.

If multiple providers share similar logic, extract it into reusable task includes while keeping provider-specific API logic separate.

---

# dPanel / dNocs Abstraction Rule

For **dPanel / dNocs abstraction**, this repository will always use the following structure:

```
tasks/<provider>/vm/(present|absent).yml
```

Example:

```
tasks/aws/vm/present.yml
tasks/aws/vm/absent.yml

tasks/tencent/vm/present.yml
tasks/tencent/vm/absent.yml

tasks/alibabacloud/vm/present.yml
tasks/alibabacloud/vm/absent.yml

tasks/idcloudhost/vm/present.yml
tasks/idcloudhost/vm/absent.yml
```

This `vm` abstraction is used exclusively by dPanel / dNocs, while the underlying provider-specific product implementations (such as `ec2`, `cvm`, `ecs`) remain the actual source of truth.

---

# Backward Compatibility

When contributing:

- Do not rename existing providers or products
- Do not restructure existing task paths
- Ensure new changes do not break existing consumers
- Maintain provider-native structure

---

# Contributor Guidelines

Before submitting a PR:

- Follow provider-native product naming
- Use `present.yml` and `absent.yml`
- Keep tasks idempotent
- Avoid introducing abstraction layers in provider product directories
- Ensure compatibility with existing consumers

If uncertain, always prefer matching the cloud provider’s official terminology over introducing a new convention.