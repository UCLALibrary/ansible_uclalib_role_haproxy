# UCLALib Ansible Role: HAProxy

Deploys HAProxy on RHEL servers

## Dependencies

This role requires a HAProxy RPM be provided from an external source as it is not available via the standard RHEL yum repositories.

Once obtained (e.g. built from source) it should be placed in this role's `files` directory.

## Deployment

You can use this role to deploy HAProxy with a single frontend and as many backends as necessary

## Variables

haproxy_rpm - name of the externally supplied haproxy RPM file that will be used to install

haproxy_cfg_path - path to the haproxy configuration directory (e.g. `/etc/haproxy`)

haproxy_bin_path - path to the haproxy binary (e.g. `/usr/sbin/haproxy`)

frontend - this is the list variable for the haproxy frontend listener
- ident - identifier name for the haproxy frontend
-  acl - optional acl parameters
-  acl_cond - if you define an acl above, use this to define the resulting behavior
-  default_be - define a default backend

#### Sample frontend definition:
```
frontend:
  ident: 'lb_solr'
  acl: 'http_methods method POST DELETE PUT'
  acl_cond: 'use_backend master_solr if http_methods'
  default_be: 'slave_solr'
```

backends - this is the list/dictionary variable for the haproxy backends
- ident - identifier name for the haproxy backend
-  http_check - define the http check command to determine health of backend
-  hosts - hostname of the backend server

#### Sample backend definition
```
backends:
  - ident: master_solr
    http_check: 'HEAD /solr/admin/ping HTTP/1.0\r\n'
    hosts: [ t-w-solr-01 ]
  - ident: slave_solr
    http_check: 'HEAD /solr/admin/ping HTTP/1.0\r\n'
    hosts: [ t-w-solr-02, t-w-solr-03 ]
```

The variable definitions should be placed in the playbook under the `vars` statement.
Example:

```
---

- name: uclalib_haproxy.yml
  sudo: true
  hosts: test
  # Define the intended haproxy config parameters in the vars section below
  vars:
    frontend:
      ident: 'lb_solr'
      acl: 'http_methods method POST DELETE PUT'
      acl_cond: 'use_backend master_solr if http_methods'
      default_be: 'slave_solr'
    backends:
      - ident: master_solr
        http_check: 'HEAD /solr/admin/ping HTTP/1.0\r\n'
        hosts: [ t-w-solr-01 ]
      - ident: slave_solr
        http_check: 'HEAD /solr/admin/ping HTTP/1.0\r\n'
        hosts: [ t-w-solr-02 ]

  roles:
    - { role: uclalib_role_haproxy }
```
