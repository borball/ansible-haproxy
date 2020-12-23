Ansible Role: HAProxy
=========

An ansible role to install HAProxy and configure the pools based on pre-defined templates. It can support deploying and configuring multiple HAProxy instances on the same nodes.

Requirements
------------

This role  more focus on HAProxy configuration than installation, users may have to ensure the HAProxy packages exist in the repo.

Role Variables
--------------

Following are the default settings which can be customized based on the needs. 

```yaml
haproxy:
  software: /usr/sbin/haproxy
  conf: /etc/haproxy
  sysconf: /etc/sysconfig
  group: haproxy
  user: haproxy
  sock_location: /var/lib/haproxy
  global:
    maxconn: 32768
  defaults:
    retries: 2
    maxconn: 8192
    timeout:
      connect: 3000
      client: 10000
      server: 10000
```

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Prepare a yml file lbs.yml to define all the HAProxy instances, following is an example:

```yaml
lbs:
  lb_service1:
    hosts:
      - localhost
      - CA-00001205
    service: lb_service1
    frontend:
      - 192.168.1.200:8080
      - 172.10.1.200:8080
    backend:
      pool:
        pool1: 172.10.1.1:8080
        pool2: 172.10.1.2:8080
        pool3: 172.10.1.3:8080
    health:
      - option httpchk GET /health
      - http-check expect status 200
    stat:
      enabled: yes
      listen: 172.10.1.200:28080

  lb_service2:
    hosts:
      - CA-00001205
      - localhost
    service: lb_service2
    frontend:
      - 192.168.1.200:8082
    backend:
      pool:
        pool1: 172.10.1.1:8082
        pool2: 172.10.1.2:8082
        pool3: 172.10.1.3:8082

  lb_service3:
    service: lb_service3
    frontend:
      - 172.10.1.200:8084
      - 172.10.1.201:8084
    backend:
      pool:
        pool1: 172.10.1.1:8084
        pool2: 172.10.1.2:8084
        pool3: 172.10.1.4:8084
```

You can define all HAProxy instances based on your needs, including the name, frontend, backend, health check, and hosts list etc.

Download the role either with ansible galaxy or add the role as a git submodule in your own ansible project.

Following is an example how to use the lbs.yml and role. 

```yaml
- hosts: servers
  vars_files:
    - lbs.yml
  roles:
    - role: borball.haproxy
```

License
-------

Apache-2.0

Author Information
------------------

