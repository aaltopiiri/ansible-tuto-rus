Пособие по Vagrant
================

Группировка хостов
--------------

Хосты в inventory можно группировать. Например, можно создать группу `debian`, группу `web-servers`, группу `production` и так далее.

    [debian]
    host0.example.org
    host1.example.org
    host2.example.org

Можно даже сократить:

    [debian]
    host[0-2].example.org

Если хотите задавать дочерние группы, используйте `[groupname:children]` и добавьте дочерние группы в него. Например, у нас есть разные дистрибутивы Линукса, их можно организовать следующим образом:

    [ubuntu]
    host0.example.org

    [debian]
    host[1-2].example.org

    [linux:children]
    ubuntu
    debian

Установка переменных
-----------------

Вы можете добавлять переменные для хостов в нескольких местах: в inventory-файле, файлах переменных хостов, файлах переменных групп и др.

Обычно я задаю все переменные в файлах переменных групп/хостов (подробнее об этом позже). Однако, зачастую я использую переменные напрямую в inventory-файле, например, `ansible_ssh_host`, которая задает IP-адрес хоста. По умолчанию Ansible резолвит имена хостов при соединении по SSH. Но когда вы инициализируете хост, он, возможно, еще не имеет IP-адреса. `ansible_ssh_host` будет полезен в таком случае.

При использовании команды `ansible-playbook` (не обычной команды `ansible`), переменные можно задавать с помощью флага `--extra-vars` (или `-e`). О команде `ansible-playbook` мы поговорим в следующем шаге.

`ansible_ssh_port`, как вы могли догадаться, используется чтобы задать порт соединения по SSH.

    [ubuntu]
    host0.example.org ansible_ssh_host=192.168.0.12 ansible_ssh_port=2222

Ansible ищет дополнительные переменные в файлах переменных групп и хостов. Он будет искать эти файлы в директориях `group_vars` и `host_vars`, внутри директории где расположен главный inventory-файл.

Ansible будет искать файлы по имени. Например, при использовании упомянутого ранее inventory-файла, Ansible будет искать переменные `host0.example.org` в файлах:

- `group_vars/linux`
- `group_vars/ubuntu`
- `host_vars/host0.example.org`

Если этих файлов не существует – ничего не произойдет, но если они существуют – они будут использованы.

Теперь когда мы познакомились с модулями, инвентаризацией и переменными, давайте, наконец, узнаем о настоящей силе Ansible с плейбуками.

Переходите к [step-04](https://github.com/freetonik/ansible-tuto-rus/tree/master/step-04).


