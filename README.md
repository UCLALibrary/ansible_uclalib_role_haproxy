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

### Frontend Variables

frontend_ident - identifier name for the haproxy frontend

frontend_acls - list variable defining acl parameters for the frontend

frontend_acl_conds - list variable defining acl conditions for the frontend acls

frontend_default_be - define a default backend

#### Sample frontend definition:
```
frontend:
  ident: 'lb_solr'
  acl: 'http_methods method POST DELETE PUT'
  acl_cond: 'use_backend master_solr if http_methods'
  default_be: 'slave_solr'
```

### Backend Variables

backends - this is the list variable for the haproxy backends
- ident - identifier name for the backend
- http_check - define the http check command to determine health of backend
- hosts - list of hostnames associated with the backend

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
    frontend_ident: 'lb_solr'
    frontend_acls:
      - acl: 'query_network_allowed src 0.0.0.0/0'
      - acl: 'update_network_allowed src 164.67.0.0/16'
      - acl: 'allow_query_path path_dir /select /terms'
      - acl: 'restricted_path path_beg /solr'
      - acl: 'http_methods method POST DELETE PUT'
    frontend_aclconds:
      - aclcond: 'http-request allow if allow_query_path query_network_allowed'
      - aclcond: 'http-request deny if restricted_path !update_network_allowed'
      - aclcond: 'use_backend master_solr if http_methods'
    frontend_defaultbe: 'slave_solr'
    backends:
      - ident: master_solr
        http_check: 'HEAD /solr/www/admin/ping HTTP/1.0\r\n'
        hosts: [ p-w-solrmaster01 ]
      - ident: slave_solr
        http_check: 'HEAD /solr/www/admin/ping HTTP/1.0\r\n'
        hosts: [ p-w-solrslave01, p-w-solrslave02 ]
  roles:
    - { role: uclalib_role_haproxy }
```
