# Демонстрация работы с Ansible

## 1. Установка и первый запуск

Имеется ноутбук, с устастонывленным свежим Ubuntu Server 20.04.6. Установлена система без LVM на весь диск, установлен сервер SSH, ip 192.168.10.200, имя пользователя и пароль ansible.

Устанавливаем Ansible на управляющий хост `sudo apt install ansible -y`

Устанавливаем python на управляемый хост `sudo apt install python -y`. Разрешаем пользователю ansible работать без пароля: вводим `sudo visudo` и добавляем в конец строку `ansible ALL=(ALL) NOPASSWD: ALL`.

Дальше делаем настройку с управляемого хоста `ansible all -i "192.168.10.200," -m setup -u ansible -e "ansible_ssh_pass=ansible"` и пробуем его пинговать `ansible all -i "192.168.10.200," -m raw -a "ping" -u ansible -e "ansible_ssh_pass=ansible"`. Должен быть такой ответ:
```
192.168.10.200 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

Чтобы ноутбук не засыпал при закрытии крышки добавим строку в **/etc/systemd/logind.conf** `ansible all -i "192.168.10.200," -m lineinfile -a "dest=/etc/systemd/logind.conf line='HandleLidSwitch=ignore' create=yes state=present" -u ansible -e "ansible_ssh_pass=ansible" -b`. Затем перезапустим службу **systemd-logind** `ansible all -i "192.168.10.200," -m systemd -a "name=systemd-logind state=restarted" -u ansible -e "ansible_ssh_pass=ansible" -b`.

## Создаём первый плейбук

Создаём папку под проект **testbook1**. В ней размещаем файл Vagrant с таким содержимым:
```
Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.hostname = "testserver"
  config.vm.network "forwarded_port", id: 'ssh', guest: 22, host: 2202, host_ip: "127.0.0.1", auto_correct: false
  config.vm.network "forwarded_port", id: 'http', guest: 80, host: 8080, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", id: 'https', guest: 443, host: 8443, host_ip: "127.0.0.1"
  # запретить обновление дополнений гостевой системы
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
  end
  config.vm.provider "virtualbox" do |virtualbox|
    virtualbox.name = "ch03"
  end
end
```

В файл **ignore_close_lid.yml**:
```
---
- name: Отключение засыпания ноутбука при закрытии крышки 
  hosts: 192.168.10.200
  become: yes
  become_user: root
  tasks:
    - name: Добавляем 'HandleLidSwitch=ignore' в logind.conf
      lineinfile:
        dest: /etc/systemd/logind.conf
        line: 'HandleLidSwitch=ignore'
        create: yes
        state: present
      when: ansible_distribution == 'Ubuntu'
    - name: Перезапускаем systemd-logind
      systemd:
        name: systemd-logind
        state: restarted

```

- В первой задаче мы используем модуль lineinfile для добавления строки HandleLidSwitch=ignore в файл /etc/systemd/logind.conf.

- Во второй задаче мы используем модуль systemd для перезапуска службы systemd-logind.

- Опция become: yes включает выполнение команд с правами суперпользователя через sudo.

- Опция become_user: root указывает, что выполнение команд в роли суперпользователя.

- Условие when: ansible_distribution == 'Ubuntu' позволяет выполнить первую задачу только на системах с операционной системой Ubuntu.

Чтобы выполнить этот плейбук, используйте команду: `ansible-playbook ignore_close_lid.yml -u ansible -e ansible_ssh_pass=ansible"`.