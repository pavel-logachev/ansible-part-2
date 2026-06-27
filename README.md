# Домашнее задание к занятию «Ansible. Часть 2» - Павел Логачев

## Стенд

- Управляемый хост: `devops-lab`
- ОС: Ubuntu Server 24.04.4 LTS
- Пользователь для подключения: `pavel`
- Inventory: [inventory.ini](inventory.ini)

## Подготовка

Если коллекция `community.general` еще не установлена:

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

Запуск плейбуков:

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini --private-key ~/.ssh/id_ed25519 playbooks/01_unpack_archive.yml
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini --private-key ~/.ssh/id_ed25519 playbooks/02_install_tuned.yml
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini --private-key ~/.ssh/id_ed25519 playbooks/03_motd_variable.yml
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini --private-key ~/.ssh/id_ed25519 playbooks/04_motd_facts.yml
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.ini --private-key ~/.ssh/id_ed25519 playbooks/05_apache_role.yml
```

## Задание 1

### 1. Скачать архив, создать папку и распаковать архив

Файл: [playbooks/01_unpack_archive.yml](playbooks/01_unpack_archive.yml)

Плейбук:

- создает директорию для скачивания;
- создает директорию для распаковки;
- скачивает архив Apache Commons Lang;
- распаковывает архив в `/opt/ansible-homework/commons-lang3`.

### 2. Установить tuned, запустить демон и добавить в автозагрузку

Файл: [playbooks/02_install_tuned.yml](playbooks/02_install_tuned.yml)

Плейбук:

- устанавливает пакет `tuned` из стандартного репозитория Ubuntu;
- запускает сервис `tuned`;
- включает автозагрузку сервиса.

### 3. Изменить приветствие системы с использованием переменной

Файл: [playbooks/03_motd_variable.yml](playbooks/03_motd_variable.yml)

Переменная задана в [group_vars/all.yml](group_vars/all.yml):

```yaml
motd_message: "Добро пожаловать на учебный сервер Ansible."
```

Плейбук записывает значение переменной в `/etc/motd`.

## Задание 2

Файл: [playbooks/04_motd_facts.yml](playbooks/04_motd_facts.yml)

Плейбук модифицирует решение из задания 1.3 и записывает в `/etc/motd`:

- hostname управляемого хоста;
- IP-адрес управляемого хоста;
- пожелание хорошего дня системному администратору.

Для этого используются Ansible facts:

```jinja2
{{ ansible_facts.hostname }}
{{ ansible_facts.default_ipv4.address }}
```

## Задание 3

Плейбук: [playbooks/05_apache_role.yml](playbooks/05_apache_role.yml)

Роль: [roles/apache_host_info](roles/apache_host_info)

Роль выполняет следующие действия:

- устанавливает Apache;
- при необходимости добавляет правило UFW для TCP-порта 80;
- генерирует `/var/www/html/index.html` из Jinja2-шаблона;
- выводит на веб-странице CPU, RAM, размер первого HDD и IP-адрес;
- запускает Apache и включает автозагрузку;
- проверяет доступность сайта модулем `uri`, ожидаемый код ответа: `200`;
- содержит handler `Restart Apache`, который перезапускает Apache только при изменении шаблона `index.html`.

В роли не используются модули `shell` и `command`.

## Вывод выполнения

Выводы сохранены в папке `evidence`:

- [evidence/00-ansible-version.txt](evidence/00-ansible-version.txt)
- [evidence/01-unpack-archive.txt](evidence/01-unpack-archive.txt)
- [evidence/02-install-tuned.txt](evidence/02-install-tuned.txt)
- [evidence/03-motd-variable.txt](evidence/03-motd-variable.txt)
- [evidence/04-motd-facts.txt](evidence/04-motd-facts.txt)
- [evidence/05-apache-role.txt](evidence/05-apache-role.txt)
- [evidence/06-apache-page-check.txt](evidence/06-apache-page-check.txt)
- [evidence/07-syntax-check.txt](evidence/07-syntax-check.txt)

Краткий результат выполнения:

```text
01_unpack_archive.yml: ok=3 changed=0 failed=0 skipped=1
02_install_tuned.yml: ok=2 changed=0 failed=0
03_motd_variable.yml: ok=1 changed=1 failed=0
04_motd_facts.yml: ok=2 changed=1 failed=0
05_apache_role.yml: ok=10 changed=2 failed=0
```

Проверка Apache:

```text
HTTP 200
Host facts: devops-lab
CPU: 4 vCPU
RAM: 7.8 GB
First HDD: sda 30.00 GB
IP address: 172.30.48.172
```

Синтаксическая проверка `ansible-playbook --syntax-check` успешно пройдена для всех пяти плейбуков.

## Архив роли

Архив роли для отправки: [apache_host_info-role.zip](apache_host_info-role.zip).
