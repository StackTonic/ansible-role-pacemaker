# Pacemaker role for Ansible

## Requirements

This role has been written for and tested in Scientific Linux 7. It might also work in other
distros, please share your experience.

## Role variables

Boolean values in properties (parsed by Pacemaker itself) don't have to be quoted. However,
resource agents may expect Boolean-like arguments as integers, strings, etc. Such values **must**
be quoted.

## Required

#### `pacemaker_password`

The plaintext password for the cluster user. It will be hashed with per-host salt to maintain
idempotency.

## Optional

#### `pacemaker_cluster_name`

Name of the cluster.

Default: `hacluster`.

#### `pacemaker_hosts`

The list of cluster members.

Default: `ansible_play_batch`

#### `pacemaker_user`

The system user to authenticate PCS nodes with. PCS will authenticate all nodes with each other.

Default: hacluster

#### `pacemaker_properties`

Dictionary with cluster properties.

#### `pacemaker_resource_defaults`

Dictionary of resource defaults.

#### `pacemaker_simple_resources`

A dictionary of simple (primitive) resources. Keys are resource IDs. Values are dictionaries.

These dictionaries may have the following members.

* `resource`: a dictionary defining the resource agent (`class`, `provider`, and `type`). This
 member is required.

* `options`: a dictionary of options specific to the resource agent. Remember to quote values in the
 format that the RA expects.

* `meta`: a dictionary of resource [meta attributes](https://clusterlabs.org/pacemaker/doc/en-US/Pa
cemaker/1.1/html/Pacemaker_Explained/s-resource-options.html#_resource_meta_attributes)
(RA-nonspecific).

* `op`: a list of dictionaries defining resource operations. The members `name` and `interval` are
required.

#### `pacemaker_advanced_resources`

A dictionary of advanced (group, clone, or master) resources. Dictionary keys are resource IDs.
Members may include the following.

* `type`: required, one of: `group`, `clone`, or `master`.

* `meta`: optional, a dictionary of advanced resource meta attributes.

* `resources`: a dictionary of primitive resources, identical to `pacemaker_simple_resources`
described above.

#### `pacemaker_constraints`

A list of dictionaries defining resource constraints. The following members are required, the rest
being optional.

* `type`: one of: `location`, `colocation`, or `order`.

* `score`: constraint score (signed integer or +/-INFINITY).

## Example playbooks

### Active-active chrooted BIND DNS server

    ---
    - name: Configure DNS cluster
      hosts: dns-servers
      tasks:
        - name: Setup cluster
          include_role:
            name: devgateway.pacemaker
          vars:
            pacemaker_password: hunter2
            pacemaker_cluster_name: named
            pacemaker_properties:
              stonith-enabled: false
            pacemaker_simple_resources:
              dns-ip:
                resource:
                  class: ocf
                  provider: heartbeat
                  type: IPaddr2
                options:
                  ip: 10.0.0.1
                  cidr_netmask: 8
                op:
                  - name: monitor
                    interval: 5s
            pacemaker_advanced_resources:
              dns-clone:
                type: clone
                resources:
                  named:
                    resource:
                      class: service
                      type: named-chroot
                    op:
                      - name: monitor
                        interval: 5s
            pacemaker_constraints:
              - order dns-ip then dns-clone

### Active-active Squid proxy

    ---
    - name: Configure Squid cluster
      hosts: proxy-servers
      tasks:
        - name: Setup cluster
          include_role:
            name: devgateway.pacemaker
          vars:
            pacemaker_password: hunter2
            pacemaker_cluster_name: squid
            pacemaker_properties:
              stonith-enabled: false
            pacemaker_simple_resources:
              squid-ip:
                resource:
                  class: ocf
                  provider: heartbeat
                  type: IPaddr2
                options:
                  ip: 192.168.0.200
                  cidr_netmask: 24
                op:
                  - name: monitor
                    interval: 5s
            pacemaker_advanced_resources:
              squid:
                type: clone
                resources:
                  squid-service:
                    resource:
                      class: service
                      type: squid
                    op:
                      - name: monitor
                        interval: 5s
            pacemaker_constraints:
              - order squid-ip then squid

## See also

- [The official Pacemaker documentation](http://clusterlabs.org/doc/)

## Copyright

Copyright 2015-2018, Development Gateway. Licensed under GPL v3+.
