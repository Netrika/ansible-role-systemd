# systemd-service

Create and set systemd unit for service.

## Role Variables

| Parameter                | Choices/Defaults                  | Comments                              |
| ------------------------ | --------------------------------- | ------------------------------------- |
| systemd_service_list     | **Default**: empty                | Список сервисов которые будут созданы |
| systemd_service_template | **Default**: `systemd.service.j2` | Путь до шаблона юнита                 |

Из `systemd_service_list` используются некоторые значения для определения дальнейшего поведения роли после создания сервиса. Рассмотрим пример:

```
systemd_service_list:
  - name: "{{ project_name }}-app"
    runtimedir: "{{ project_name }}"
    environmentfile: "{{ django_settings_path }}"
    pidfile: "/run/{{ project_name }}/{{ project_name }}.pid"
    workingdir: "{{ backend_current }}"
    user: "{{ project_user }}"
    execstart: >-
      {{ venv_path }}/bin/gunicorn staff.wsgi:application
      --pid /run/{{ project_name }}/{{ project_name }}.pid -c {{ backend_current }}/gunicorn.py
    execreload: /bin/kill -s HUP $MAINPID
    execstop: /bin/kill -s TERM $MAINPID
    description: "{{ project_name|upper }} gunicorn service."
    state: restarted
    enabled: true
```

Переменные определяющие дальнейшее поведение:

| Parameter | Choices/Defaults                  | Comments                                                                                                         |
| --------- | --------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| enabled   | **Default**: `false`              | Должен ли юнит запускаться при запуске ОС. Если значение `state` не указано данный параметр будет проигнорирован |
| name      | **Default**: undefined            | Имя создаваемого юнита                                                                                           |
| state     | **Default**: undefined            | В каком состоянии должен находится сервис                                                                        |
| unit_path | **Default**: `etc/systemd/system` | Путь где будет создан юнит                                                                                       |

Если используется стандартный шаблон можно выбрать следующие опции:

| Parameter        | Choices/Defaults         | Comments                                                                                                                                                                                                       |
| ---------------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| config_dest      | **Default**: undefined   | `ЭКСПЕРЕМЕНТАЛЬНЫЙ ФУНКЦИОНАЛ`. Позволяет скопировать дополнительные файлы конфигурации для запуска сервиса                                                                                                    |
| config_src       | **Default**: undefined   | `ЭКСПЕРЕМЕНТАЛЬНЫЙ ФУНКЦИОНАЛ`. Позволяет скопировать дополнительные файлы конфигурации для запуска сервиса                                                                                                    |
| description      | **Default**: `item.name` | Описание сервиса                                                                                                                                                                                               |
| environmentfile  | **Default**: undefined   | Путь до файла с переменными окружения. [See more](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#EnvironmentFile=)                                                                         |
| execreload       | **Default**: undefined   | Команда перезапуска сервиса. [See more](https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecReload=)                                                                                     |
| execstart        | **Default**: undefined   | Команда запуска сервиса. [See more](https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecStart=)                                                                                          |
| execstop         | **Default**: undefined   | Команда остановки сервиса. [See more](https://www.freedesktop.org/software/systemd/man/systemd.service.html#ExecStop=)                                                                                         |
| group            | **Default**: `item.user` | Группа от имени которой будет запущен сервис. [See more](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#User=)                                                                             |
| privatetmp       | **Default**: `true`      | [See more](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#PrivateTmp=)                                                                                                                     |
| restart          | **Default**: `always`    | Определяет поведение сервиса при неожиданной остановке                                                                                                                                                         |
| restartsec       | **Default**: `1min`      | Таймаут между попытки перезапуска сервиса                                                                                                                                                                      |
| runtimedir       | **Default**: undefined   | Имя директории, которая будет создана в каталоге /run/. [See more](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#RuntimeDirectory=)                                                       |
| syslogidentifier | **Default**: undefined   | Если параметр определен потоки вывода и ошибок будут перенаправленны в syslog с тэгом указанным в переменной. [See more](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#SyslogIdentifier=) |
| type             | **Default**: `Simple`    | [See more](https://www.freedesktop.org/software/systemd/man/systemd.service.html#Type=)                                                                                                                        |
| user             | **Default**: undefined   | Пользователь от имени которого будет запущен сервис. [See more](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#User=)                                                                      |
| workingdir       | **Default**: undefined   | Каталог в котором будет запущенно приложение. [See more](https://www.freedesktop.org/software/systemd/man/systemd.exec.html#WorkingDirectory=)                                                                 |

## Example Playbook

```
- name: Converge
  hosts: all
  become: true
  vars:
    - project: foo
    - systemd_service_list:
      - name: "{{ project_name }}-app"
        runtimedir: "{{ project_name }}"
        environmentfile: "{{ django_settings_path }}"
        pidfile: "/run/{{ project_name }}/{{ project_name }}.pid"
        workingdir: "{{ backend_current }}"
        user: "{{ project_user }}"
        execstart: >-
          {{ venv_path }}/bin/gunicorn staff.wsgi:application
          --pid /run/{{ project_name }}/{{ project_name }}.pid -c {{ backend_current }}/gunicorn.py
        execreload: /bin/kill -s HUP $MAINPID
        execstop: /bin/kill -s TERM $MAINPID
        description: "{{ project_name|upper }} gunicorn service."
        state: restarted
        enabled: true

  roles:
    - role: systemd-service
```

## License

MIT
