# Understanding with Google Cloud Platform

## Google Computer Engining

* Manage VS Unmanaged

The difference, instance-group managed can use autoscaling, instance-group unmanaged just like group.

> This page describes how to create a group of unmanaged instances, known as a unmanaged instance group.
> Unmanaged instance groups are collections of instances that are not necessarily identical and do not share a common instance template. You can use unmanaged instance groups to accommodate your pre-existing configurations for load balancing tasks. However, you should always use managed instance groups unless your applications require you to group instances together that are not identical.

* Routers vs Routes

Routers for VPN
Routes as Routes for internet Routes

* Deployment Template

The schema file is like the one in azure, parameter file. it's available for json version as well.
more details you can reference this [link](https://cloud.google.com/deployment-manager/docs/configuration/templates/using-schemas)

Schema

```yaml
info:
  title: VM Template
  author: Jane
  description: Creates a new network and instance
  version: 1.0

imports:
- path: vm-instance.jinja # Must be a relative path

required:
- IPv4Range

properties:
  IPv4Range:
    type: string
    description: Range of the network

  description:
    type: string
    default: "My super great network"
    description: Description of network
```

jinja

```yaml
resources:
- name: vm-1
  type: vm-instance.jinja

- name: a-new-network
  type: compute.v1.network
  properties:
    IPv4Range: {{ properties['IPv4Range'] }}
    description: {{ properties['description'] }}
```

Structure of a schema

```yaml
info:
  title: MongoDB Template
  author: Jane
  description: Creates a MongoDB cluster
  version: 1.0

imports:
  - path: helper.py
    name: mongodb_helper.py

required:
  - name

properties:
  name:
    type: string
    description: Name of your Mongo Cluster

  size:
    type: integer
    default: 2
    description: Number of Mongo Slaves

  zone:
    type: string
    default: us-central1-a
    description: Zone to run
    metadata: gce-zone
```