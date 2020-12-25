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
  option: 'OPTIONS=\"-dS -dG -dV\"'
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

None.

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

Each HAProxy instance defined in lbs.yml will have a separated set of configurations including: 

- /etc/haproxy/{instance_name}/haproxy.cfg
- /etc/sysconfig/{instance_name}
- /lib/systemd/system/{instance_name}.service

Use systemctl status {instance_name}.service to check its status, for example:

```shell
borball@ubuntu:~/github/ansible-test$ systemctl status lb_service3
[0m lb_service3.service - HAProxy lb_service3
     Loaded: loaded (/lib/systemd/system/lb_service3.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2020-12-24 16:12:37 EST; 19h ago
    Process: 3911852 ExecStartPre=/usr/sbin/haproxy -f $CONFIG -c -q (code=exited, status=0/SUCCESS)
   Main PID: 3911853 (haproxy)
      Tasks: 2 (limit: 4620)
     Memory: 5.9M
     CGroup: /system.slice/lb_service3.service
             3911853 /usr/sbin/haproxy -Ws -f /etc/haproxy/lb_service3/haproxy.cfg -p /run/lb_service3 -dS -dG -dV
             3911860 /usr/sbin/haproxy -Ws -f /etc/haproxy/lb_service3/haproxy.cfg -p /run/lb_service3 -dS -dG -dV

Dec 24 16:12:37 ubuntu systemd[1]: Starting HAProxy { instance.service }}...
Dec 24 16:12:37 ubuntu haproxy[3911853]: [NOTICE] 358/211237 (3911853) : New worker #1 (3911860) forked
Dec 24 16:12:37 ubuntu systemd[1]: Started HAProxy { instance.service }}.
Dec 24 16:12:47 ubuntu haproxy[3911860]: [WARNING] 358/211247 (3911860) : Server backend-lb_service3/pool1 is DOWN, reason: Layer4 timeout, check duration: 1>Dec 24 16:12:51 ubuntu haproxy[3911860]: [WARNING] 358/211251 (3911860) : Server backend-lb_service3/pool2 is DOWN, reason: Layer4 timeout, check duration: 1>Dec 24 16:12:54 ubuntu haproxy[3911860]: [WARNING] 358/211254 (3911860) : Server backend-lb_service3/pool3 is DOWN, reason: Layer4 timeout, check duration: 1>Dec 24 16:12:54 ubuntu haproxy[3911860]: [ALERT] 358/211254 (3911860) : backend 'backend-lb_service3' has no server available!
```

License
-------

Apache-2.0

Author Information
------------------

