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
  lb_service1: # haproxy instance name
    hosts: # (optional) on which nodes the haproxy shall be installed and configured
      - localhost
      - CA-00001205
    service: lb_service1 # used as name of systemd
    frontend: # can have multiple frontend
      - 192.168.1.200:8080
      - 172.10.1.200:8080
    backend:
      pool: # backend pool
        pool1: 172.10.1.1:8080
        pool2: 172.10.1.2:8080
        pool3: 172.10.1.3:8080
    health: # health check for the backend pool members
      - option httpchk GET /health
      - http-check expect status 200
    stat: # whether to enable the stat page 
      enabled: yes
      listen: 172.10.1.200:28080 # stat page listening interface and port

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

- If hosts presents in lbs.yml, HAProxy will be installed and configured only on the nodes whose hostname match with the hosts defined in lbs.yml.
- If no hosts is specified in lbs.yml, HAProxy will be installed and configured on all the nodes defined in the inventory.

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

Use systemctl status {instance_name}.service to check status of HAProxy instance, for example:

```shell
borball@ubuntu:~/github/ansible-test$ systemctl status lb_service3               
[0m lb_service3.service - HAProxy lb_service3
     Loaded: loaded (/lib/systemd/system/lb_service3.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2020-12-25 12:09:26 EST; 4s ago
    Process: 897715 ExecStartPre=/usr/sbin/haproxy -f $CONFIG -c -q (code=exited, status=0/SUCCESS)
   Main PID: 897716 (haproxy)
      Tasks: 2 (limit: 4620)
     Memory: 5.9M
     CGroup: /system.slice/lb_service3.service
             897716 /usr/sbin/haproxy -Ws -f /etc/haproxy/lb_service3/haproxy.cfg -p /run/lb_service3 -dS -dG -dV
             897717 /usr/sbin/haproxy -Ws -f /etc/haproxy/lb_service3/haproxy.cfg -p /run/lb_service3 -dS -dG -dV

Dec 25 12:09:26 ubuntu systemd[1]: Starting HAProxy lb_service3...
Dec 25 12:09:26 ubuntu haproxy[897716]: [NOTICE] 359/120926 (897716) : New worker #1 (897717) forked
Dec 25 12:09:26 ubuntu systemd[1]: Started HAProxy lb_service3.
```

License
-------

Apache-2.0

Author Information
------------------

