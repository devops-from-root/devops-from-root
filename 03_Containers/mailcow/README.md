# Установка сервера Mailcow

## DNS и порты

Перед установкой надо открыть порты __25|80|110|143|443|465|587|993|995|4190__.
Минимальная настройка DNS:

```text
# Name              Type       Value
mail                IN A       1.2.3.4
autodiscover        IN CNAME   mail.example.org.
autoconfig          IN CNAME   mail.example.org.
@                   IN MX 10   mail.example.org.
```

## Подготовка к установке

Настройка на чистом сервере Ubuntu 20.04/22.04. Входим под пользователем, созданным при установке. Переключаемся в суперпользователя `sudo -i`. Задаём пароль `passwd root` и повторяем его. Разрешаем вход руту `echo "PermitRootLogin yes" > /etc/ssh/sshd_config.d/99-allow-root.conf`. Редактируем `nano /etc/fstab`, если в нём не закомментирован _/swap.img_, то надо закомментировать. Перезагружаем сервер. После перезагрузки входим как _root_, вся дальнейшая работа от рута. Командой `free` проверяем, что _swap_ не подключен. 

## Настройка сервера

Это однострочный скрипт сделает все настройки и перезагрузит сервер в конце. Необходимо заменить только __DOMAIN_NAME__ в самом начале.

```bash
DOMAIN_NAME=mail.example.com && PUB_IP=$(ip -4 addr show $(ip -4 route ls | grep default | grep -Po '(?<=dev )(\S+)' | head -1) | grep -oP '(?<=inet\s)\d+(\.\d+){3}') && echo -e "PUB_IP=$PUB_IP\nDOMAIN_NAME=$DOMAIN_NAME" > /etc/environment && source /etc/environment && echo "$DOMAIN_NAME=$PUB_IP" && hostnamectl set-hostname $DOMAIN_NAME && snap list | awk 'NR>1 && $1 !~ /^core/ && $1 !~ /^snapd$/ {print $1}' | xargs -n1 snap remove --purge; snap list | awk '$1 ~ /^core/ {print $1}' | xargs -n1 snap remove --purge; snap remove snapd; apt autoremove snapd -y; rm -rf /snap; rm -rf /root/snap && awk -F: '$3 > 999 && $3 < 65534' /etc/passwd | cut -d: -f1 | xargs -I {} deluser --remove-all-files {} ; getent group | awk -F: '$3 > 999 && $3 < 65534 {print $1}' | xargs -I {} groupdel {} && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get dist-upgrade -y && DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && echo 'vm.swappiness=10' | tee -a /etc/sysctl.conf && echo 'vm.vfs_cache_pressure=50' | tee -a /etc/sysctl.conf && if [ -z "$(grep -E '^/dev/[a-zA-Z0-9]+\s+swap\s' /etc/fstab)" ] && [ -z "$(swapon --show)" ]; then rm -rf /swap.img && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends dphys-swapfile && echo -e 'CONF_SWAPFILE=/swap.img\nCONF_MAXSWAP=2048' > /etc/dphys-swapfile && dphys-swapfile setup; fi && timedatectl set-timezone Europe/Moscow && rm -f /etc/update-motd.d/{50-motd-news,90-updates-available,91-release-upgrade} && reboot
```

Дальше будут объяснения, на случай если скрипт прервётся из-за ошибки. Если выполнится нормально и перезагрузится, переходим к [Установке Docker'а](#установка-docker)

### Имя сервера

```bash
DOMAIN_NAME=mail.example.com && PUB_IP=$(ip -4 addr show $(ip -4 route ls | grep default | grep -Po '(?<=dev )(\S+)' | head -1) | grep -oP '(?<=inet\s)\d+(\.\d+){3}') && echo -e "PUB_IP=$PUB_IP\nDOMAIN_NAME=$DOMAIN_NAME" > /etc/environment && source /etc/environment && echo "$DOMAIN_NAME=$PUB_IP" && hostnamectl set-hostname $DOMAIN_NAME
```

### Удалить snapd

```bash
snap list | awk 'NR>1 && $1 !~ /^core/ && $1 !~ /^snapd$/ {print $1}' | xargs -n1 snap remove --purge; snap list | awk '$1 ~ /^core/ {print $1}' | xargs -n1 snap remove --purge; snap remove snapd; apt autoremove snapd -y; rm -rf /snap; rm -rf /root/snap
```

### Удаление пользователей кроме root

```bash
awk -F: '$3 > 999 && $3 < 65534' /etc/passwd | cut -d: -f1 | xargs -I {} deluser --remove-all-files {} ; getent group | awk -F: '$3 > 999 && $3 < 65534 {print $1}' | xargs -I {} groupdel {}
```

### Обновление

```bash
apt-get update && DEBIAN_FRONTEND=noninteractive apt-get dist-upgrade -y && DEBIAN_FRONTEND=noninteractive apt-get dist-upgrade -y
```

### Тюнинг swap

```bash
sysctl vm.swappiness=10 && sysctl vm.vfs_cache_pressure=50 && echo 'vm.swappiness=10' | tee -a /etc/sysctl.conf && echo 'vm.vfs_cache_pressure=50' | tee -a /etc/sysctl.conf && if [ -z "$(grep -E '^/dev/[a-zA-Z0-9]+\s+swap\s' /etc/fstab)" ] && [ -z "$(swapon --show)" ]; then rm -rf /swap.img && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends dphys-swapfile && echo -e 'CONF_SWAPFILE=/swap.img\nCONF_MAXSWAP=2048' > /etc/dphys-swapfile && dphys-swapfile setup; fi
```

### Временная зона, удаление рекламы и перезагрузка

```bash
echo "PermitRootLogin yes" > /etc/ssh/sshd_config.d/99-allow-root.conf && systemctl restart ssh.service && timedatectl set-timezone Asia/Yekaterinburg && rm -f /etc/update-motd.d/{50-motd-news,90-updates-available,91-release-upgrade} && reboot
```

## Установка Docker

```bash
DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends ca-certificates curl p7zip-full git && install -m 0755 -d /etc/apt/keyrings && curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc && chmod a+r /etc/apt/keyrings/docker.asc && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" >  /etc/apt/sources.list.d/docker.list && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin && systemctl start docker && systemctl enable docker
```

## Установка Mailcow

В переменных нужно установить правильные порты.

```bash
HTTP_PORT=8034 && HTTPS_PORT=8443 && echo -e "HTTP_PORT=$HTTP_PORT\nHTTPS_PORT=$HTTPS_PORT" >> /etc/environment && source /etc/environment && echo "$HTTP_PORT/$HTTPS_PORT" && cd /opt && git clone https://github.com/mailcow/mailcow-dockerized && cd mailcow-dockerized && ./generate_config.sh <<< $DOMAIN_NAME'\n' && sed -i 's/SKIP_LETS_ENCRYPT=n/SKIP_LETS_ENCRYPT=y/' mailcow.conf && sed -i "s/HTTP_BIND=.*/HTTP_BIND=$PUB_IP/" mailcow.conf && sed -i "s/HTTP_PORT=80.*/HTTP_PORT=$HTTP_PORT/" mailcow.conf && sed -i "s/HTTPS_BIND=.*/HTTPS_BIND=$PUB_IP/" mailcow.conf && sed -i "s/HTTPS_PORT=443.*/HTTPS_PORT=$HTTPS_PORT/" mailcow.conf && docker compose pull && docker compose up -d && echo "work on http://$DOMAIN_NAME:$HTTP_PORT (http://$PUB_IP:$HTTP_PORT) or https://$DOMAIN_NAME:$HTTPS_PORT (https://$PUB_IP:$HTTPS_PORT)"
```

## Настройка Mailcow

### Базовая настройка

Заходим через браузер по предоставленной ссылке __https://...__. И ждём, пока всё запустится. Около 10-15минут в среднем занимает первый запуск. Как заработает, авторизуемся логин __admin__ пароль __moohoo__.
На странице __https://.../edit/admin/admin__ меняем пароль админа.
На странице __https://.../mailbox__ в __Домены->Домены__ добавляем домен `example.com`.
После того как домен будет добавлен, в строке с именем домена нажимаем синюю кнопку _DNS_, выведется таблица с настройками DNS. В колонке __Статус__ должны уже стоять галочки на значениях __A__, __MX__ и две записи __CNAME__. Остальные записи не обязательны и будет стоять цифра _2_. Их необходимо добавить в настройках регистратора доменных имён вручную.
- __PTR__ меняется у хостера, значение должно равняться статусу;
- __TLSA__ на текущий момент reg.ru не поддерживает;
- __SRV__ название будет `_autodiscover._tcp.example.com`, а значение `mail.example.com 8443`, где 8443 - это порт https;
- **TXT DKIM** основная подпись сервера, название будет `dkim._domainkey.example.com`, а значение `	v=DKIM1;k=rsa;t=s;s=email;p=ключ`, где ключ будет сгенерирован Mailcow;
- **TXT SPF** название `example.com`, а значение `v=spf1 mx a -all` позволит отправлять с этого домена почту из других доменов;
- **TXT DMARK** фильтрует почту, не прошедшую проверку через DKIM и SPF, название `_dmarc.example.com`, а значение `v=DMARC1; p=quarantine`.

### Почтовые ящики

На странице __https://.../mailbox__ в __Почтовые ящики->Почтовые ящики__ добавляем почтовые ящики `catchall@example.com` и `bounce@example.com`.

#### Catch-all

На странице __https://.../mailbox__ в __Псевдонимы->Псевдонимы__ добавляем псевдоним. В поле _Псевдоним/ы_ вводим `@example.com`, а в поле _Владельцы псевдонима_ вводим `catchall@example.com`.

#### Bounce

На странице __https://.../mailbox__ во вкладке __Фильтры__ редактируем значения __Global Prefilter__, добавив 3 строки:

```text
if address :is "from" "MAILER-DAEMON@example.com" {
  redirect "bounce@example.com";
}
```

Нажимаем __Проверить__ и __Сохранить изменения__.
