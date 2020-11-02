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
I.
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
3. В файле /etc/security/time.conf
Дописываем
ssh;*;!admin;Wk0000-2400
ssh;*;admin;Al0000-2400

В файле `/etc/pam.d/sshd`
Добавляем 
`account    required     pam_time.so`

Настраиваем, чтобы ssh пускал:
В файле `/etc/ssh/sshd_config`
Переключаем параметр `PasswordAuthentication` на `yes`
И перезапускаем демона SSH: `service sshd restart`

>Готово!

//ToDo: сделать список праздников, с помощью скрипта проверять текущий день (date) на присутствие в файле. Запрещать, если праздник
