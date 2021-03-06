+++
date = "2016-07-03T04:02:20Z"
title = "Preflight Checks"
description = "A guide to implementing the Preflight Checks feature to analyze customer systems to determine if the environment meets the minimum requirements for installation or update."
weight = "213"
categories = [ "Packaging" ]

[menu.main]
Name       = "Preflight Checks"
identifier = "preflight-checks"
parent     = "/packaging-an-application"
url        = "/docs/packaging-an-application/preflight-checks"
+++

A preflight check is a test that is run before installing and running an application.  The
test will analyze the system to determine if the environment meets the minimum requirements.  

By default, Replicated automatically adds preflight checks for:

| **Category** | **Check** |
|--------------|-----------|
| OS | Linux |
| Distribution | Supported Linux distributions |
| Linux Kernel | 3.10 or greater |
| Memory | 1 GB |
| Docker Version | {{< docker_version_minimum >}} - {{< docker_version_default >}} |
| Disk Space | /tmp 1 GB <br /> /var/lib/replicated 250 MB <br /> /var/lib/docker/aufs 1 GB (aufs storage driver root directory) |
| TCP Ports (Replicated services) | 9870-9880 on docker0 |
| Outbound internet access (if required) | Replicated APIs, external registries |

Additionally, it's recommended to specify additional system requirements in the `host_requirements` section of the
application YAML. These host requirements will apply to single node installs, as well as each node on distributed
installs.

```yaml
name: My Counter App
host_requirements:
  docker_version: "1.10.3"
  cpu_cores: 2
  cpu_mhz: 2400
  memory: 8GB
  disk_space: 8GB
```

It is possible to override all properties (except docker version) of the root `host_requirements` on a per-component basis. On distributed
installs, the component host requirements will only apply to nodes tagged for that component.

```yaml
components:
- name: DB
  tags: ["db"]
  host_requirements:
    cpu_cores: 2
    cpu_mhz: 2400
    memory: 8GB
    disk_space: 30GB
```

Note that component host requirements are not additive, thus when multiple components are allocated to a single host, each requirement
will be evaluated individually.

It is also possible to define minimum disk space requirements for volumes on the host machine via the host_volumes property of the
component configuration.

```yaml
components:
- name: DB
  tags: ["db"]
  host_volumes:
  - host_path: /data
    min_disk_space: 30GB
  containers:
  - image_name: redis
```

Replicated enforces these requirements and will not allow the customer to start the application without either meeting these requirements or
dismissing the warnings. Upon dismissing preflight warnings, an entry will be recorded in the on-premise audit log.

![Preflight Checks Screenshot](/static/preflight-checks.png)

{{% page_notes %}}
- `min_disk_space` does not guarantee free space, it refers to the disk size mounted at the specified location.
- The requested `docker_version` must be one of the versions Replicated supports.
{{% /page_notes %}}
