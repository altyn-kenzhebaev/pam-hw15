# PAM
Для выполнения этого действия требуется установить приложением git:
`git clone https://github.com/altyn-kenzhebaev/pam-hw15.git`
В текущей директории появится папка с именем репозитория. В данном случае pam-hw15. Ознакомимся с содержимым:
```
cd pam-hw15
ls -l
README.md
Vagrantfile
```
Здесь:
- README.md - файл с данным руководством
- Vagrantfile - файл описывающий виртуальную инфраструктуру для `Vagrant`
Запускаем ВМ:
```
vagrant up
```
Дальнейшие работы будет проведена под пользователем root:
```
vagrant ssh
sudo su -
```
## Настройка запрета для всех пользователей (кроме группы Admin) логина в выходные дни (Праздники не учитываются)
Создаём пользователя otusadm и otus:
```
useradd otusadm && useradd otus
```
Создаём пользователям пароли: :
```
echo "Otus2022!" | passwd --stdin otusadm && echo "Otus2022!" |  passwd --stdin otus
```
Создаём группу admin `groupadd -f admin`
Добавляем пользователей vagrant,root и otusadm в группу admin:
```
usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
```
Проверим, что пользователи root, vagrant и otusadm есть в группе admin:
```
$ cat /etc/group | grep admin
admin:x:1003:otusadm,root,vagrant
```
Создадим файл-скрипт /usr/local/bin/login.sh:
```
$ vi /usr/local/bin/login.sh

#!/bin/bash

if [ $(date +%a) = "Sat" ] || [ $(date +%a) = "Sun" ]; then
 if getent group admin | grep -qw "$PAM_USER"; then
        exit 0
      else
        exit 1
    fi
  else
    exit 0
fi

```
Добавим права на исполнение файла: `chmod +x /usr/local/bin/login.sh`
Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт:
```
$ vi /etc/pam.d/sshd 
#%PAM-1.0
auth       substack     password-auth
auth       include      postlogin
account    required     pam_exec.so /usr/local/bin/login.sh
account    required     pam_sepermit.so
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so
session    include      password-auth
session    include      postlogin
```
Отключаем сервисы синхронизации времени и сервис агента вирталбокса:
```
systemctl --now disable chronyd
systemctl --now disable vboxadd-service
```
Меняем время и пытаемся залогинится пользователем Otus и  Otusadm:
```
$ timedatectl set-time 2023-06-03
$ date
Sat Jun  3 00:00:01 UTC 2023
$ hwclock 
2023-06-03 00:00:04.121283+00:00
$ ssh otus@192.168.50.10
otus@192.168.50.10's password: 
/usr/local/bin/login.sh failed: exit code 1
Connection closed by 192.168.50.10 port 22
$ ssh otusadm@192.168.50.10
otusadm@192.168.50.10's password: 
$ 
```