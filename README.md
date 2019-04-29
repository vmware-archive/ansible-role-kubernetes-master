# ansible-role-kubernetes_master

Ansible playbook to automate installing and maintaining kubernetes master nodes.

## Requirements

This role currently requires a working VMware Photon OS server with enabled
Docker environment.

## Role Variables

The following variables are available within the role, with defaults noted:

```yaml
# Container memory limit. Use 512MB, type string, or 0 for unlimited
memory_limit: 0MB

# The name of the kubernetes package to install.
kubernetes_package: kubernetes

# The name of the flannel package to install.
flannel_package: flannel
```

## Example playbook

```yaml
---
- hosts: kubernetes_masters
  sudo: True
  roles:
    - kubernetes_master
  vars:
    - ... forthcoming
```

# License and Copyright
 
Copyright 2015 VMware, Inc.  All rights reserved.

SPDX-License-Identifier: Apache-2.0 OR GPL-3.0-only

[This code is Dual Licensed Apache-2.0 or GPLv3](LICENSE)

