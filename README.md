kibana-role
=========

Роль для установки Kibana на хостах с ОС: Debian, Ubuntu, CentOS, RHEL.

Requirements
------------

Поддерживаются только следующие ОС: Debian, Ubuntu, CentOS, RHEL.

Role Variables
--------------

| Variable name | Default | Description |
|-----------------------|----------|-------------------------|
| kibana_version | "7.14.0" | Параметр, который определяет, какой версии Kibana будет установлена |

Example Playbook
----------------

    - hosts: servers
      roles:
         - { role: kibana-  role }

License
-------

MIT

Author Information
------------------

Olga Ivanova, devops-10
