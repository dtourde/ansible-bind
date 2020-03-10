# Ansible Role bind

## Description

This role is used to manage bind configuration on my server.
I run bind inside a container I build from debian image.

It's a very basic image as it only install bind and prepare the mounted /data to match /etc/bind & such.

I'll push this container repo ASAP.

However, this role is meant to be used with non-containerized environment.


## Private variables

The first task include a file named "secret_vars.yml"

This file includes these variables, you must set your own file in order proper fonction of the role.

```yaml
public_ip_addr: "{{ipify_public_ip}}" # Yes, it's a shortcut for A record in my case
ip_router: 10.xxx.xxx.xxx # eg. used as forwarder

domains_acl:
  - name: gandi-net
    ipv4:
      - 217.70.176.0/20
  - name: example-org
    ipv4:
      - 1.2.3.4
```

## Usage

You need to prepare your own vars file for bind, named as your wish, e.g. the name of the server.
There is a template of this file in `vars/example_vars.yml`

You also need to prepare you own `secret_vars.yml` (can be vaulted too).  
  
Place them in `roles/bind/vars/`, they will be loaded with:

```yaml
- name: Include vars files, see README.md for full list
  include_vars: "{{item}}"
  tags: always
  loop:
    "vars/{{e_bind_vars_file}}.yml"
    "vars/secret_vars.yml"
```

If using docker_compose with dto/bind9 image, extra var `e_use_docker_compose` should be set to _True_.
If set to _False_, bind9 will be installed via package manager (Debian only for now)

## Playbook file example

You might want to set your extra vars inside your `deploy_bind.yml`:

`deploy_bind.yml`:
```yaml
---
- name: Deploy bind on server 1 within docker
  hosts: server1
  gather_facts: yes
  roles:
    - { role: bind, e_bind_vars_file: server1_vars.yml, e_use_docker_compose: True }

- name: Deploy bind on server 2
  hosts: server2
  gather_facts: yes
  roles:
    - { role: bind, e_bind_vars_file: example_vars.yml, e_use_docker_compose: False }
```
