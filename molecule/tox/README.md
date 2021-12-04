## Тестирование tox

- Сценарий инициирован: `molecule init scenario tox --driver-name podman`
- На хостовой машине выполнено `sudo sysctl -w vm.max_map_count=262144`
- Собираем Dockerfile в образ и запускаем с помощью `docker run --privileged=True -v /home/olga/docs/projects/devops:/opt/ansible-testing -w /opt/ansible-testing -it img_for_ansible /bin/bash`
- В файле `tox-requirements.txt` - зависимости
- Дополнительно в папке `molecule/tox` создана подпапка `files` (туда происходит скачивание)
- Облегчённый сценарий для `molecule` проверялся с помощью команды `molecule test -s tox`
- Tox запускался командной `tox`