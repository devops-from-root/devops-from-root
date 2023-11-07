# Практический пример работы с Git

## Подключение к аккаунту на гите

Проверяем имя пользователя в текущем аккаунте гита `git config user.name` и меняем, если необходимо `git config --global user.name "Ваше Имя"`.

Проверяем текущее имя пользователя гита `git config user.email` и меняем, если необходимо `git config --global user.email "ваш@адрес.почты"`.

Переходим в рабочую папку `cd /путь/к/вашему/локальному/репозиторию`, проверяем что в ней `ls'

Используйте команду git remote add, чтобы связать ваш локальный репозиторий с удаленным. Вам нужно будет указать имя (обычно origin) и URL удаленного репозитория. `git remote add origin https://github.com/devops-from-root/devops-from-root.git`

Если репозиторий уже подключен, `то git remote -v` выдаст информацию
```
origin  https://github.com/devops-from-root/devops-from-root.git (fetch)
origin  https://github.com/devops-from-root/devops-from-root.git (push)
```

Создаём подпапку `mkdir 00_Git`, добавляем её к отслеживанию `git add 00_Git/` и переходим в неё `cd 00_Git`

Содаём файл `echo "## Демонстрация работы с Git и GitHub" > git_demo.md`, проверяем статус `git status` и добавляем новый файл к отслеживанию `git add git_demo.md`

Делаем первый коммит и добавляем в staging все отслеживаемые файлы `git commit -am "Создана папка 00_Git с файлом git_demo.md"`

Проверяем, что коммит есть в истории `git log`. Смотрим информацию о последнем коммите `git show`.

[![back](https://img.shields.io/badge/в_начало-646464)](../README.md)