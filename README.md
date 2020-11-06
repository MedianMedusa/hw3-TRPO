# ДЗ 3 по технологиям разработки ПО

## Задание
Надо было создать нескольких пользователей и кое-что сделать с их уровнями доступа. ТЗ:
>1.
>>- Создать нескольких пользователей, задать им пароли, домашние директории и шеллы;
>>- Создать группу admin;
>>- Включить нескольких из ранее созданных пользователей, а также пользователя root, в группу admin;
>>- Запретить всем пользователям, кроме группы admin, логин в систему по SSH в выходные дни (суббота и воскресенье, без учета праздников).
>>- * С учётом праздничных дней.
>>(Для упрощения проверки можно разрешить парольную аутентификацию по SSH и использовать ssh user@localhost проверяя логин с этой же машины)
>
>2.
>>- Установить docker; дать конкретному пользователю:
>>>- права работать с docker (выполнять команды docker ps и т.п.);
>>>- * возможность перезапускать демон docker (systemctl restart docker) не выдавая прав более, чем для этого нужно;

## Инструкция
I. Первая часть
1. Создаём пользователей

По дефолту для новых пользователей установлены след значения: 
```
[vagrant@localhost /]$ useradd –D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```
Создаём трёх пользователей:
```
[vagrant@localhost /]$ sudo useradd user1
[vagrant@localhost /]$ sudo useradd user2
[vagrant@localhost /]$ sudo useradd user3
```
Проверяем:
```
[vagrant@localhost /]$ ls /home
user1  user2  user3  vagrant

[vagrant@localhost ~]$ groups user1 user2 user3
user1 : user1
user2 : user2
user3 : user3
```
Меняем пароли на 123456 новым пользователям, чтобы можно было залогиниться:
```
[vagrant@localhost ~]$ sudo passwd user1
[vagrant@localhost ~]$ sudo passwd user2
[vagrant@localhost ~]$ sudo passwd user3
```
2. Создаём группу admin и добавляем туда двух пользователей и рута:
```
[vagrant@localhost ~]$ sudo groupadd admin
[vagrant@localhost ~]$ sudo usermod user1 -aG admin
[vagrant@localhost ~]$ sudo usermod user2 -aG admin
[vagrant@localhost ~]$ sudo usermod root -aG admin

[vagrant@localhost ~]$ groups user1 user2 user3 root vagrant
user1 : user1 admin
user2 : user2 admin
user3 : user3
root : root admin
vagrant : vagrant
```
3. В файле `/etc/security/time.conf`
Дописываем
```
sshd;*;!admin;Wk0000-2400
sshd;*;admin;Al0000-2400
```

В файле `/etc/pam.d/sshd`
добавляем 
`account    required     pam_time.so`

Настраиваем, чтобы ssh пускал:
- В файле `/etc/ssh/sshd_config`
переключаем параметр `PasswordAuthentication` на `yes`
- Перезапускаем демона SSH: `service sshd restart`

>Готово!

4. Задание со звёздочкой
- Устанавливаем pam_script из epel-release;
- Добавляем в `/etc/pam.d/sshd` строку `account    required     pam_script.so `
- В `/etc` удаляем существующий файл-ссылку `pam_script_acct` и создаём обычный файл с таким же именем, заполняем как у меня в репозитории
- Тут же создаём файл `holidays` со списком [праздников](https://guide.travel.ru/russia/entertainment/holidays/ "Брал отсюда") (копируем у меня же из репозитория)
Пробуем:

```
[vagrant@localhost etc]$ ssh user3@localhost
user3@localhost's password:
Connection closed by ::1 port 22
[vagrant@localhost etc]$
[vagrant@localhost etc]$ ssh user2@localhost
user2@localhost's password:
Last login: Fri Nov  6 13:51:02 2020 from ::1
[user2@localhost ~]$
```
>Готово!

II. Вторая часть
1. Устанавливаем Docker по [инструкции](https://docs.docker.com/engine/install/centos/) (используя репозиторий)
2. Запускаем демона `sudo systemctl start docker` и проверяем, что он работает:
```
[vagrant@localhost ~]$ sudo docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
```
3. Добавляем пользователя в группу docker
```
[vagrant@localhost ~]$ sudo usermod -aG docker user3
[vagrant@localhost ~]$ cat /etc/group | grep docker
docker:x:989:vagrant,root,user3
```
И проверяем:
```
[vagrant@localhost ~]$ ssh user3@localhost
user3@localhost's password:
[user3@localhost ~]$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
```
При этом у user2 ошибка:
```
[vagrant@localhost ~]$ ssh user2@localhost
user2@localhost's password:
[user2@localhost ~]$ docker images
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.40/images/json: dial unix /var/run/docker.sock: connect: permission denied
```
>Готово!
4. Задание со звёздочкой:
- Добавляем user3 в sudoers командой `sudo visudo -f /etc/sudoers` следующим образом:
```
#homework
user3 ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart docker
```
Теперь user3 может выполнять sudo restart docker без запроса пароля:
```
[user3@localhost ~]$ sudo systemctl restart docker
[user3@localhost ~]$ sudo systemctl start docker

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for user3:
```
>Готово!
