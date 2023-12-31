# Ansible и Puppet

## Ansible

**Ansible** - это удивительный инструмент для автоматизации управления компьютерами и серверами. Он позволяет вам управлять множеством компьютеров одновременно, даже если они работают на разных операционных системах. Основное преимущество **Ansible** заключается в том, что вы можете выполнять задачи без необходимости писать сложные скрипты или программировать. Вместо этого вы описываете желаемое состояние системы, и **Ansible** заботится о том, чтобы оно было достигнуто.

Еще одним важным аспектом **Ansible** является его простота в использовании. Вы можете создавать "плейбуки", которые описывают последовательность шагов для выполнения задач. Это может быть что угодно, от установки программ и обновления конфигураций до управления пользователями и мониторинга серверов. **Ansible** делает это процессом, который понятен даже новичкам в автоматизации.

И, наконец, **Ansible** является открытым и бесплатным инструментом с активным сообществом пользователей. Это означает, что вы всегда можете найти помощь и ресурсы для обучения. В целом, **Ansible** - это мощный инструмент для упрощения управления вашей инфраструктурой, делая процессы автоматизации доступными для всех, независимо от уровня опыта.

### Puppet

**Puppet** - это инструмент для автоматизации управления компьютерами и серверами, который помогает вам легко контролировать и настраивать большое количество систем. Вместо того, чтобы ручным образом настраивать каждый сервер, **Puppet** позволяет вам описать желаемое состояние вашей инфраструктуры, и затем он заботится о том, чтобы это состояние было достигнуто. Это сильно упрощает жизнь системных администраторов.

Основным преимуществом **Puppet** является его способность обеспечивать непрерывность конфигурации серверов. Это значит, что если какие-либо изменения случайно или намеренно произойдут, **Puppet** автоматически вернет систему к желаемому состоянию. Это помогает предотвратить ошибки и обеспечивает стабильную работу вашей инфраструктуры.

Кроме того, **Puppet** является открытым и бесплатным инструментом с активным сообществом пользователей. Это обеспечивает доступ к множеству ресурсов и поддержку. В целом, **Puppet** - это мощный инструмент для упрощения управления вашей инфраструктурой, делая автоматизацию настройки серверов более доступной и надежной.

## Сравнение Ansible и Puppet 

Ansible и Puppet - два популярных инструмента для автоматизации управления серверами. Они оба позволяют вам контролировать состояние вашей инфраструктуры, но есть некоторые различия. Ansible более прост в использовании и не требует установки агентов на серверы, что делает его более легким в настройке. Puppet, с другой стороны, имеет более мощные возможности для управления конфигурациями и обеспечения их непрерывности.

Выбор между Ansible и Puppet зависит от ваших потребностей и предпочтений. Если вам нужна быстрая и простая автоматизация, Ansible может быть лучшим выбором. Если же вам нужна более сложная и масштабируемая система управления конфигурациями, то Puppet может быть более подходящим вариантом. В конечном итоге, оба инструмента имеют свои преимущества, и выбор зависит от вашей специфической ситуации.

### Шпаргалка по Ansible

1. **Установка Ansible**

```
sudo apt-get install ansible
```

2. **Проверка доступности хостов**

```
ansible all -m ping
```

3. **Запуск ad-hoc команды на хостах**

```
ansible <группа_серверов> -a "<команда>"
```

4. **Выполнить команду на удаленном сервере**

```
ansible <группа_серверов> -m <модуль> -a "<команда>"
```

5. **Создание плейбука** yaml

```
- name: Пример плейбука
  hosts: <группа_серверов>
  tasks:
    - name: Задача 1
      <модуль>: <аргументы>
```

6. **Запустить плейбук**

```
ansible-playbook playbook.yml
```

7. **Применение плейбука на несколько хостов**

```
ansible-playbook playbook.yml -l <группа_серверов>
```

8. **Проверка синтаксиса плейбука**

```
ansible-playbook --syntax-check playbook.yml
```

9. **Проверка, какие задачи будут выполнены без применения плейбука**

```
ansible-playbook playbook.yml --list-tasks
```

10. **Определение переменной в плейбуке**

```
my_var: значение
```

11. **Использование переменной в плейбуке** yaml

```
{{ my_var }}
```

12. **Использование переменных в командах**

```
ansible <группа_серверов> -a "echo {{ my_var }}"
```

13. **Копирование файла с использованием шаблона** yaml

```
- name: Копировать файл
  template:
    src: файл.j2
    dest: /путь/к/цели
```

14. **Получение информации об инвентаре**

```
ansible-inventory -i inventory.ini --list
```

15. **Установка фактов (гathering facts) на хостах**

```
ansible <группа_серверов> -m setup
```

### Шпаргалка по Puppet

1. **Применение текущей конфигурации Puppet**

```
puppet apply /etc/puppet/manifests/site.pp
```

1. **Проверка состояния системы с использованием фактов (facts)**

```
puppet facts
```

1. **Проверка ресурсов (resources) на хосте**

```
puppet resource <тип_ресурса> <имя_ресурса>
```

1. **Запуск Puppet агента (демона) в фоновом режиме**

```
puppet agent --daemonize
```

1. **Остановка Puppet агента (демона)**

```
puppet agent --disable
```

1. **Применение конфигурации через сервер (при использовании Puppet Master)**

```
puppet agent -t
```

1. **Сбор и отправка фактов (facts) на Puppet Master**

```
puppet facts upload
```

1. **Проверка синтаксиса конфигурационных файлов**

```
puppet parser validate <файл.pp>
```

1. **Получение информации о модулях Puppet**

```
puppet module list
```

1. **Установка модуля Puppet**

```
puppet module install <автор/модуль>
```

1. **Просмотр списка классов и ресурсов в манифестах**

```
puppet describe <класс_или_ресурс>
```

1. **Создание нового модуля Puppet**

```
puppet module generate <автор-модуля>-<название-модуля>
```

1. **Проверка, какие изменения будут внесены без применения конфигурации**

```
puppet agent --noop --test
```

1. **Запуск Puppet агента с использованием другого окружения**

```
puppet agent --environment <имя_окружения>
```



[![back](https://img.shields.io/badge/в_оглавление-646464)](README.md)