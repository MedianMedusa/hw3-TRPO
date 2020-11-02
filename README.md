# ДЗ 3 по технологиям разработки ПО

## Задание
Надо было создать нескольких пользователей и кое-что сделать с их уровнями доступа. ТЗ:
>1.
>А. Создать нескольких пользователей, задать им пароли, домашние директории и шеллы;
>Б. Создать группу admin;
>В. Включить нескольких из ранее созданных пользователей, а также пользователя root, в группу admin;
>Г. Запретить всем пользователям, кроме группы admin, логин в систему по SSH в выходные дни (суббота и воскресенье, без учета праздников).
>Д*. С учётом праздничных дней.
>(Для упрощения проверки можно разрешить парольную аутентификацию по SSH и использовать ssh user@localhost проверяя логин с этой же машины)
>
>2. Установить docker; дать конкретному пользователю:
>А. права работать с docker (выполнять команды docker ps и т.п.);
>Б*. возможность перезапускать демон docker (systemctl restart docker) не выдавая прав более, чем для этого нужно;

## Инструкция
1. Создаём пользователей

По дефолту установлены след значения: (useradd –D)
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes

[vagrant@localhost /]$ sudo useradd user1
[vagrant@localhost /]$ sudo useradd user2
[vagrant@localhost /]$ sudo useradd user3

[vagrant@localhost /]$ ls /home
user1  user2  user3  vagrant

[vagrant@localhost ~]$ groups user1 user2 user3
user1 : user1
user2 : user2
user3 : user3

[vagrant@localhost ~]$ sudo passwd user1… (123456)

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

В файле /etc/security/time.conf
Дописываем
ssh;*;user1|user2;Wk0000-2400
ssh;*;vagrant|root|user3;Al0000-2400

В файле /etc/pam.d/sshd
Добавляем 
account    required     pam_time.so

Настраиваем, чтобы ssh пускал:
В файле /etc/ssh/sshd_config
Переключаем параметр PasswordAuthentication на yes
И перезапускаем демона ссх: service sshd restart

Теперь должно работать

//ToDo: сделать список праздников, с помощью скрипта проверять текущий день (date) на присутствие в файле. Запрещать, если праздник
