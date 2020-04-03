Пособие по Vagrant
================

Общение с узлами
------------------

Теперь мы готовы. Давайте поиграем с уже знакомой нам командой из прошлого раздела: `ansible`. Эта команда – одна из трех команд, которую Ansible использует для взаимодействия с узлами.

# Сделаем что-нибудь полезное

В прошлой команде `-m ping` означал "используй модуль _ping_". Это один из множества модулей, доступных в Ansible. Модуль `ping` очень прост, он не требует никаких аргументов. Модули, требующие аргументов, могут получить их через `-a`. Давайте взглянем на несколько модулей.

## Модуль shell

Этот модуль позволяет запускать shell-команды на удаленном узле:

ansible host0 -m shell -a 'uname -a'

Вывод должен быть вроде:

host0 | CHANGED | rc=0 >>
Linux host0 4.15.0-91-generic #92-Ubuntu SMP Fri Feb 28 11:09:48 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

Легко!

## Модуль copy

Не удивительно, модуль copy позволяет копировать файл из управляющей машины на удаленный узел. Представим что нам нужно скопировать наш `/etc/apt` в `/tmp` узла:

ansible host0 -m copy -a 'src=/etc/hosts dest=/tmp/'

Вывод:

host0 | CHANGED => {
    "changed": true,
    "checksum": "b32a2b1da6a659df1c7691e91f97e9bc6188be27",
    "dest": "/tmp/hosts",
    "gid": 1000,
    "group": "vagrant",
    "md5sum": "f3501c5237b07fafb9b34ee604d8a4cd",
    "mode": "0664",
    "owner": "vagrant",
    "size": 483,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1585904856.333801-48940972319216/source",
    "state": "file",
    "uid": 1000
}

Ansible (точнее, модуль _copy_, запущенный на узле) ответил кучей полезной информации в формате JSON. Мы увидим как это можно будет использовать позже.

У Ansible есть огромный
[список модулей](http://docs.ansible.com/list_of_all_modules.html), который покрывает практически все, что можно делать в системе. Если вы не нашли подходящего модуля, написание своего модуля – довольно простая задача (и не обязательно использовать Python, нужно лишь разговаривать с помощью JSON).

# Много хостов, одна команда

Ок, все что было выше – прикольно, но нам нужно управлять множеством хостов. Давайте попробуем.

Допустим, мы хотим собрать факты про узел и, например, хотим узнать какая версия Ubuntu установлена на узлах. Это довольно легко:

ansible all -a 'grep DISTRIB_RELEASE /etc/lsb-release'

`all` это синоним 'все хосты в inventory-файле'. Вывод будет примерно таким:

host0 | CHANGED | rc=0 >>
DISTRIB_RELEASE=18.04
host1 | CHANGED | rc=0 >>
DISTRIB_RELEASE=18.04
host2 | CHANGED | rc=0 >>
DISTRIB_RELEASE=18.04

# Больше фактов

Легко и просто. Однако, если нам нужно больше информации (IP-адреса, размеры ОЗУ, и пр.), такой подход может быстро оказаться неудобным. Решение – использовать модуль `setup`. Он специализируется на сборке _фактов_ с узлов.

Попробуйте:

    ansible host0 -m

ответ:

    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.0.60"
        ],
        "ansible_all_ipv6_addresses": [],
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "01/01/2007",
        "ansible_bios_version": "Bochs"
        },
        ---snip---
        "ansible_virtualization_role": "guest",
        "ansible_virtualization_type": "kvm"
    },
    "changed": false,
    "verbose_override": true

Вывод был сокращен для простоты, но вы можете узнать много интересного из этой информации. Вы также можете фильтровать ключи если вас интересует что-то конкретное.

Например, вам нужно узнать сколько памяти доступно на всех хостах. Это легко, запустите

ansible all -m setup -a 'filter=ansible_memtotal_mb'

ответ:

host2 | SUCCESS => {
    "ansible_facts": {
        "ansible_memtotal_mb": 355
    },
    "changed": false
}
host1 | SUCCESS => {
    "ansible_facts": {
        "ansible_memtotal_mb": 355
    },
    "changed": false
}
host0 | SUCCESS => {
    "ansible_facts": {
        "ansible_memtotal_mb": 355
    },
    "changed": false
}
Заметьте, что узлы ответили не в том порядке, в котором они отвечали выше. Ansible общается с хостами параллельно!

Кстати, при использовании модуля `setup` можно указывать `*` в выражении `filter=`. Как в shell.

# Выбор хостов

Мы видели, что `all` означает 'все хосты', но в Ansible есть
[куча иных способов выбирать хосты](http://ansible.cc/docs/patterns.html#selecting-targets):

- `host0.example.org:host1.example.org` будет запущен на host0.example.org и на
  host1.example.org
- `host*.example.org` будет запущен на всех хостах, названия которых начинается с 'host' и заканчивается на
'.example.org' (тоже как в shell)

Есть другие способы с использованием групп, о них мы узнаем в следующем шаге
[step-03](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-03).


