## Тестирование

Подготовительный этап:
- На хостовой машине выполнено `sudo sysctl -w vm.max_map_count=262144` (иначе не работает Elasticsearch)
- Почти во всех tasks и roles добавлена переменная `ansible_become: false`, чтобы игнорировалось `become: true` 
(иначе падают ошибки при выполнении на `podman`)
- Сценарий инициирован: `molecule init scenario --driver-name podman`
- Дополнительно в папке `molecule/default` создана подпапка `files` (туда происходит скачивание)
- Запуск производился с помощью `molecule --debug -vvv test --destroy=never` (`molecule destroy` выполнялся отдельно)
- На хост заходила с помощью `molecule login --host centos7`
- `verify` иногда срабатывал не сразу, видимо, нужно было время, чтобы запустилась Kibana

Хосты:
- выполняется на `centos7` и `ubuntu` (`centos8` требует dnf по умолчанию, а у нас везде yum)
- в `platforms` также указан пустой `hostname`, иначе падает ошибка
- с `ubuntu` есть периодические проблемы со сменой пользователя (`su test`) - из-за `ansible_become: false` по-другому 
сменить пользователя не удаётся

`dependency`:
- роли подключаются через локальный git-репозиторий (т.к. в elasticsearch-role была необходима правка в handlers по
`when: ansible_virtualization_type != 'docker' and ansible_virtualization_type != 'podman'` - без условия по `podman` пытается запустить handler).
Для этого в `molecule.yml` в `provisioner` в `env` указана переменная `ANSIBLE_ROLES_PATH: /opt/ansible-testing` и закомментирован
блок с `dependency` и ссылкой на файл `requirements.yml`)

`prepare`:
- выполняется установка `ip route` (иначе в `hostvars` в `ubuntu` не появляется IP-адрес) и `Java` (для работы Elasticsearch)
- затем подключена роль `mnt-homeworks-ansible` для установки Elasticsearch
- создан пользователь `test`, добавлен в `sudoers` (по шаблону), добавлены права на файлы Elasticsearch и выполнен запуск от имени этого пользователя
  (под рутом он не работает)

`converge`:
- выполняются действия по установке `kibana-role` с параметрами `ansible_become: false` и `kibana_elastic_host: '{{ inventory_hostname }}'`

`side_effect`:
- выполняется запуск Kibana

`verify`:
- проверяется успешный http запрос
- проверяются логи 

Правки в `kibana-role`:
- В handlers добавлено условие по `ansible_virtualization_type != 'podman'`
- В шаблоне `kibana.yml.j2` вместо хардкода в виде `el-instance` указана переменная `kibana_elastic_host` (хост с Elasticsearch).
В роли эта переменная вынесена в `defaults`, прописана в README.md. В целях тестирования определяется как `inventory_hostname`


Итоговый ответ `verify` по `centos7`:
```
ok: [centos7] => {
    "accept_ranges": "bytes",
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "cache_control": "private, no-cache, no-store, must-revalidate",
    "changed": false,
    "connection": "close",
    "content_length": "135922",
    "content_security_policy": "script-src 'unsafe-eval' 'self'; worker-src blob: 'self'; style-src 'unsafe-inline' 'self'",
    "content_type": "text/html; charset=utf-8",
    "cookies": {},
    "cookies_string": "",
    "date": "Sat, 04 Dec 2021 03:13:50 GMT",
    "elapsed": 0,
    "invocation": {
        "module_args": {
            "attributes": null,
            "body": null,
            "body_format": "raw",
            "client_cert": null,
            "client_key": null,
            "creates": null,
            "dest": null,
            "follow_redirects": "safe",
            "force": false,
            "force_basic_auth": false,
            "group": null,
            "headers": {},
            "http_agent": "ansible-httpget",
            "method": "GET",
            "mode": null,
            "owner": null,
            "remote_src": false,
            "removes": null,
            "return_content": false,
            "selevel": null,
            "serole": null,
            "setype": null,
            "seuser": null,
            "src": null,
            "status_code": [
                200
            ],
            "timeout": 30,
            "unix_socket": null,
            "unsafe_writes": false,
            "url": "http://localhost:5601",
            "url_password": null,
            "url_username": null,
            "use_proxy": true,
            "validate_certs": true
        }
    },
    "kbn_license_sig": "1c91939b9c454b26f26f3ed8828ed28d03d029bf9a7d301a85f3c6d0c63a9c29",
    "kbn_name": "d92e5e0688e5",
    "msg": "OK (135922 bytes)",
    "redirected": true,
    "referrer_policy": "no-referrer-when-downgrade",
    "status": 200,
    "url": "http://localhost:5601/app/home",
    "vary": "accept-encoding",
    "x_content_type_options": "nosniff"
}
META: ran handlers
META: ran handlers

PLAY RECAP *********************************************************************
centos7                    : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

INFO     Verifier completed successfully.
```