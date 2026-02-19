Описание:
--------------

Ansible-роль для запуска и настройки RabbitMQ Server в докер-контейнерах
Допускается активация плагинов, установка\верификация объявленных пользователей и vhosts, в режимах standalone и cluster
Допускается разворачивание сразу нескольких инстансов RabbitMQ на одном хосте (разделяя порты\data-dirs как угодно)
Для управления используются dict rabbitmq

За основу взято это: https://github.com/usegalaxy-eu/ansible-rabbitmq/

Пример описания для хоста
--------------
#### Общий вид
```
rabbitmq:                                             # Объявление dict rabbitmq
  debug: <True|False>                                 # False (Default) - именно для дебага роли (set True - if you know what are you doing)
  override_coockies: <True|False>                     # False (Default) - Erlang Coockie устанавливается только если ее (файла) не существовало на сервере в data_path/erlang.coockie
                                                      # т.е. если файл существует - то его верификация произведена не будет (не рекомендуется использовать False)
                                                      # True - всегда будет производиться верификция erlang coockie для всех инстансов
                                                      # Если для инстанса не указана erlang_coockie - будет сгенерирована рандомная
                                                      # False существует только для того, чтобы без особых изменений создавать (лепить) standalone-инстансы
  instances:                                          # Объявление инстансов, в рамках этого dict производится скалирование
    instance1:                                        # Произвольное имя, по которому производится настройка\скалирование инстанса
      docker_container:                               # Параметры docker-контейнера
        name: 'RabbitMQ-1'                            # Имя контейнера
        image: 'rabbitmq:3.12.14-management'          # Какой образ использовать для инстанса
        hostname: "RabbitMQ-1"                        # Hostname в контейнере
        ports: []                                     # list портов для проброса (docker run -p ..... )
                                                      # Если list пустой: 
                                                      #     будут использованы порты 15672,5672 для standalone реализации
                                                      #     будут использованы порты 15672,5672,4369,25672 для кластерной реализации
        env: {}                                       # dict переменных для контейнера. Если пустой будет добавлена переменная:
                                                      #   RABBITMQ_USE_LONGNAME: !standalone (длинные имена важны при кластере)
      templates_path: '.'                             # Путь, где искать шаблоны для конфигураций (erlang.cookie.j2,rabbitmq.conf.j2)
      cluster_name: 'rmq1'                            # Имя кластера - неважно, если standalone: True
      standalone: <True|False>                        # True - инстанс одиночный
                                                      # False - инстанс в составле RMQ кластера
      data_path: /opt/rabbitmq                        # Где на хосте хранятся конфигурация и данные инстанса
      cluster_members: []                             # Важно, если standalone==False, list нод входящих в кластер
      users: []                                       # list dict'ов: (если не пустой, производится удаление пользователя guest)
                                                      #   - user: '<login>'
                                                      #     password: '<password>'
                                                      #     tags: <теги через запятую>
                                                      #     vhost: <виртуальный хост>
                                                      # При этом:
                                                      # 1) vhost - создается при необходимости
                                                      # 2) проверяются явки пользователь и если они ошибочны (или пользователя не существует) --> пользователь пересоздается
      plugins:                                        # list плагинов (rabbitmq-plugins enable ...)
        - rabbitmq_management                         # По-умолчанию включаем rabbitmq_management
      configuration:                                  # объявление конфигураций для шаблона rabbitmq.conf.j2 (зависит от шаблона)
        vm_memory_high_watermark_relative: <value>    # WaterMark для RAM (0<x<1) (Default: 0.95)
        custom: []                                    # list любых строк добавляемых в rabbitmq.conf
```
Пример playbook (rabbitmq.yml)
--------------
```
---
- hosts: all
  become: True
  become_method: sudo
  roles:
    - rabbitmq
```
Запуск playbook
--------------
```
ansible-playbook -i inventory -l <some_host> rabbitmq.yml [-CD]
```
