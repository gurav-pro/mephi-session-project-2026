
root включен  
пользователь alex:123  

# Сессионный проект по курсу “Безопасность GNU/Linux”: Настройка базовых средств защиты ОС GNU/Linux

## Характеристика стенда:  
Установка проводилась на ВМ в VMWare ESXi  
**Конфигурация ВМ**:  
4 ядра CPU  
4 Гб ОЗУ  
SATA 30Гб  
ОС RedOS8 в редакции Standard Server Minimal (без GUI)  

## Раздел 1. Установка дистрибутива
Установка проводилась с выключенной сеть в ВМ, чтобы избежать обновления пакетов во время установки и была возможность показать в консоли как выполняется обновление ОС и пакетов.
**Имя пользователя при установке:** alex (с правами в группе wheel)
Учетная запись root включена
```
[alex@localhost ~]$ pwd
/home/alex
[alex@localhost ~]$ id
uid=1000(alex) gid=1000(alex) группы=1000(alex),10(wheel) контекст=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```
#### 1.1. Настройка сети
Сетевой интерфейс настроен динамически при помощи протокола DHCP вовремя установки ОС. Роутером ВМ был выдан адрес 192.168.1.10/24
Убеждаемся в правильности настроек.

Просмотр списка подключений
```
nmcli connection show --active
```

Вывод команды:
```
NAME   UUID                                  TYPE      DEVICE 
ens33  ffecc940-07dd-372b-a9f4-29a2a2070d81  ethernet  ens33  
lo     bccdd12b-0c46-4731-9523-46e9606be0c0  loopback  lo   
```

Проверка наличия аренды по DHCP:
```
nmcli -f DHCP4 device show ens33
```

Вывод команды:
```
DHCP4.OPTION[1]:                        dhcp_client_identifier = 01:00:0c:29:ab:00:02
DHCP4.OPTION[2]:                        dhcp_lease_time = 600
DHCP4.OPTION[3]:                        dhcp_server_identifier = 192.168.1.1
DHCP4.OPTION[4]:                        domain_name = 
DHCP4.OPTION[5]:                        domain_name_servers = 192.168.1.1
DHCP4.OPTION[6]:                        expiry = 1793175262
DHCP4.OPTION[7]:                        ip_address = 192.168.1.10
DHCP4.OPTION[8]:                        next_server = 192.168.1.1
DHCP4.OPTION[9]:                        ntp_servers = 87.103.245.205 92.255.126.11
DHCP4.OPTION[10]:                       requested_broadcast_address = 1
DHCP4.OPTION[11]:                       requested_domain_name = 1
DHCP4.OPTION[12]:                       requested_domain_name_servers = 1
DHCP4.OPTION[13]:                       requested_domain_search = 1
DHCP4.OPTION[14]:                       requested_host_name = 1
DHCP4.OPTION[15]:                       requested_interface_mtu = 1
DHCP4.OPTION[16]:                       requested_ms_classless_static_routes = 1
DHCP4.OPTION[17]:                       requested_nis_domain = 1
DHCP4.OPTION[18]:                       requested_nis_servers = 1
DHCP4.OPTION[19]:                       requested_ntp_servers = 1
DHCP4.OPTION[20]:                       requested_rfc3442_classless_static_routes = 1
DHCP4.OPTION[21]:                       requested_root_path = 1
DHCP4.OPTION[22]:                       requested_routers = 1
DHCP4.OPTION[23]:                       requested_static_routes = 1
DHCP4.OPTION[24]:                       requested_subnet_mask = 1
DHCP4.OPTION[25]:                       requested_time_offset = 1
DHCP4.OPTION[26]:                       requested_wpad = 1
DHCP4.OPTION[27]:                       routers = 192.168.1.1
DHCP4.OPTION[28]:                       subnet_mask = 255.255.255.0
```
Как видно из вывода, присутствуют поля DHCP4.OPTION, - адрес получен по DHCP

#### 1.2. Настройка имени хоста

После установки ОС в качестве hosname установлен localhost
Проверим командами
```
hostnamectl
cat /etc/hostname
```
Вывод команд:
```
[alex@localhost ~]$ hostnamectl
   Static hostname: (unset)                               
Transient hostname: localhost
         Icon name: computer-vm
           Chassis: vm 🖴
        Machine ID: b6ef062823c74539b3cca98e5f7e331f
           Boot ID: 94b8bf43f2844686aceaa0e3bc906ba9
    Virtualization: vmware
  Operating System: RED OS 8.0.3 (Standard Server Minimal)
       CPE OS Name: cpe:/o:redos:redos:8
            Kernel: Linux 6.12.92-1.red80.x86_64
      Architecture: x86-64
   Hardware Vendor: VMware, Inc.
    Hardware Model: VMware Virtual Platform
  Firmware Version: 6.00
     Firmware Date: Thu 2020-11-12
[alex@localhost ~]$ cat /etc/hostname

```
Также, hostname указан в приглашении терминала `[@localhost]`

Чтобы задать новый hostname воспользуемся командой
```
sudo hostnamectl set-hostname mephi-2026.domain.local
```

Убедимся в этом командами выше
```
[alex@localhost ~]$ hostnamectl
 Static hostname: mephi-2026.domain.local
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: b6ef062823c74539b3cca98e5f7e331f
         Boot ID: 94b8bf43f2844686aceaa0e3bc906ba9
  Virtualization: vmware
Operating System: RED OS 8.0.3 (Standard Server Minimal)
     CPE OS Name: cpe:/o:redos:redos:8
          Kernel: Linux 6.12.92-1.red80.x86_64
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
Firmware Version: 6.00
   Firmware Date: Thu 2020-11-12
[alex@localhost ~]$ cat /etc/hostname
mephi-2026.domain.local
```
#### 1.3. Проверка сетевой связности

Проверка связи с внешним миром проводилась командой:
```
ping -c 4 8.8.8.8 > ~/ping.out
```

Результат выполнения команды (проверено через `cat ping.out`):
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=113 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=113 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=113 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=113 time=116 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 2997ms
rtt min/avg/max/mdev = 113.190/113.925/115.748/1.055 ms
```
## Раздел 2. Управление программным обеспечением

#### 2.1. Обновление дистрибутива

Обновление дистрибутива проводится командой:
```
sudo dnf -y update
```
После обновления произведена перезагрузка командой `sudo reboot`

После обновления система загрузилась на новом ядре Linux 6.12.101-1.red80.x86_64 (старая версия ядра Linux 6.12.92-1.red80.x86_64)

#### 2.2. Установка пакетов из репозиториев

Для установки пакетов nginx и libcap-ng-utils использовалась команда
```
sudo dnf install nginx libcap-ng-utils
```

#### 2.3. Установка локального RPM-пакета

Для скачивания пакета tcpdump в директорию /tmp с помощью менеджера пакетов dnf download и его установкой в систему, воспользуемся командами:

```
dnf download tcpdump --destdir /tmp
sudo rpm -ivh /tmp/tcpdump-4.99.5-1.red80.x86_64.rpm
rpm -q tcpdump
dnf history > ~/dnf.out
cat ~/dnf.out
```

## Раздел 3. Управление файловыми системами

#### 3.1. Создание файловой системы

В качестве дополнительного диска в VMWare было добавлено новое устройство, объемом 10Гб

Вывод команды `lsblk` показывает, что устройство добавлено в системе и мы можем определить его имя, в данном случае это sdb:
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   30G  0 disk 
├─sda1   8:1    0    1M  0 part 
├─sda2   8:2    0    1G  0 part /boot
├─sda3   8:3    0    3G  0 part [SWAP]
└─sda4   8:4    0   26G  0 part /
sdb      8:16   0   10G  0 disk          <------ новый диск
sr0     11:0    1 1024M  0 rom  
zram0  252:0    0  3,8G  0 disk [SWAP]
```

Для форматирования и настройки диска использовались команды:
```
sudo parted -s /dev/sdb mklabel gpt mkpart primary ext4 1MiB 100%
sudo mkfs.ext4 -L MEPHI_WEB /dev/sdb1
lsblk -f                                         
```

#### 3.2. Монтирование файловой системы

Монтирование файловой системы выполнялось командами:
```
sudo mkdir /mephi-web
echo 'LABEL=MEPHI_WEB /mephi-web ext4 defaults 0 2' | sudo tee -a /etc/fstab
sudo systemctl daemon-reload
sudo mount /mephi-web
findmnt /mephi-web
sudo findmnt --verify
```

## Раздел 4. Управление сервисами

#### 4.1. Управление веб-сервером

Для запуска и и включения автозапуска у сервиса nginx воспользуемся командой:
```
sudo systemctl enable --now nginx
```

Вывод команды
```
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
```

Проверим статус сервиса командой
```
systemctl status nginx
```

Вывод команды:
```
nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (running) since Wed 2026-10-28 13:06:54 MSK; 2min 52s ago
    Process: 1947 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1948 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1949 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1950 (nginx)
      Tasks: 5 (limit: 4662)
     Memory: 5.3M
        CPU: 155ms
     CGroup: /system.slice/nginx.service
             ├─1950 "nginx: master process /usr/sbin/nginx"
             ├─1951 "nginx: worker process"
             ├─1952 "nginx: worker process"
             ├─1953 "nginx: worker process"
             └─1954 "nginx: worker process"

окт 28 13:06:53 mephi-2026.domain.local systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
окт 28 13:06:53 mephi-2026.domain.local nginx[1948]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
окт 28 13:06:53 mephi-2026.domain.local nginx[1948]: nginx: configuration file /etc/nginx/nginx.conf test is successful
окт 28 13:06:54 mephi-2026.domain.local systemd[1]: Started nginx.service - The nginx HTTP and reverse proxy server.
```

Также, проверим что сервис слушает порт 80
```
ss -tunlp | grep 80
```

Вывод команды:
```
tcp   LISTEN 0      511          0.0.0.0:80        0.0.0.0:*          
tcp   LISTEN 0      511             [::]:80           [::]:* 
```

#### 4.2. Журналирование

Сохраним журнал nginx с момента последней загрузки в файл journalctl.out
```
sudo journalctl -u nginx -b > ~/journalctl.out
```

Проверим
```
cat ~/journalctl.out
```

Вывод команды:
```
окт 28 13:06:53 mephi-2026.domain.local systemd[1]: Starting nginx.service - The nginx HTTP and reverse proxy server...
окт 28 13:06:53 mephi-2026.domain.local nginx[1948]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
окт 28 13:06:53 mephi-2026.domain.local nginx[1948]: nginx: configuration file /etc/nginx/nginx.conf test is successful
окт 28 13:06:54 mephi-2026.domain.local systemd[1]: Started nginx.service - The nginx HTTP and reverse proxy server.
```
## Раздел 5. Управление доступом

#### 5.1. Дискреционное управление доступом (DAC)

Создадим каталог 
```
sudo mkdir -p /data/mephi-2026
```

Создадим группы для кураторов и разработчиков с нужными GID
```
sudo groupadd -g 4444 curators
sudo groupadd -g 5555 developers
```

Добавляем пользователей в систему
```
sudo useradd -u 5501 -m -G developers user1
sudo useradd -u 5502 -m -G developers user2
sudo useradd -u 5503 -m -G developers user3
sudo useradd -m -G curators curator1
sudo useradd -m -G curators curator2
```

Задаем права на каталог и проверим каталог
```
sudo chown root:developers /data/mephi-2026 
sudo chmod 2770 /data/mephi-2026 
ls -ld /data/mephi-2026
```

Вывод команды
```
drwxrws---. 2 root developers 4096 окт 28 12:29 /data/mephi-2026
```

Настроим ACL для кураторов и наследование прав, а также проверим
```
sudo setfacl -m  g:developers:rwx,g:curators:rx,o::--- /data/mephi-2026
sudo setfacl -d -m g:developers:rwx,g:curators:rx,o::--- /data/mephi-2026
getfacl /data/mephi-2026
```

Вывод последней команды
```
getfacl: Removing leading '/' from absolute path names
# file: data/mephi-2026
# owner: root
# group: developers
# flags: -s-
user::rwx
group::rwx
group:curators:r-x
group:developers:rwx
mask::rwx
other::---
default:user::rwx
default:group::rwx
default:group:curators:r-x
default:group:developers:rwx
default:mask::rwx
default:other::---
```

#### 5.2. Привилегии (уменьшение количества set-UID-программ)

Для настройки исполняемого файла tcpdump таким образом, чтобы его мог запустить обычный пользователь, воспользуемся следующими командами/

Для начала выясним текущее состояние:
```
command -v tcpdump 
ls -l /usr/sbin/tcpdump 
getcap /usr/sbin/tcpdump
```

Вывод команд
```
[alex@mephi-2026 ~]$ command -v tcpdump 
/usr/sbin/tcpdump
[alex@mephi-2026 ~]$ ls -l /usr/sbin/tcpdump 
-rwxr-xr-x. 1 root root 1299064 июл 30  2025 /usr/sbin/tcpdump
[alex@mephi-2026 ~]$ getcap /usr/sbin/tcpdump
пустой вывод
```
Бит set-UID не установлен, следовательно его не надо убирать, но тем не менее команда
```
sudo -u user1 tcpdump -i any -c 1 -nn
```
завершается ошибкой
```
tcpdump: any: You don't have permission to capture on that device
(socket: Operation not permitted)
```

Назначим необходимые привилегии и проверим результат
```
sudo setcap cap_net_raw,cap_net_admin+ep /usr/sbin/tcpdump
getcap /usr/sbin/tcpdump
ls -l /usr/sbin/tcpdump
```

Вывод команды `getcap /usr/sbin/tcpdump`
```
/usr/sbin/tcpdump cap_net_admin,cap_net_raw=ep
```

Проверим результат еще раз командой `sudo -u user1 tcpdump -i any -c 5 -nn icmp`
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
```

Как видим, tcpdump запустился и прослушивает сетевой трафик.
Отправим с другого компьютера icmp-запросы и получим результат
```
12:57:29.912667 ens33 In  IP 192.168.1.11 > 192.168.1.10: ICMP echo request, id 1, seq 1, length 40
12:57:29.912720 ens33 Out IP 192.168.1.10 > 192.168.1.11: ICMP echo reply, id 1, seq 1, length 40
12:57:31.011530 ens33 In  IP 192.168.1.11 > 192.168.1.10: ICMP echo request, id 1, seq 2, length 40
12:57:31.011574 ens33 Out IP 192.168.1.10 > 192.168.1.11: ICMP echo reply, id 1, seq 2, length 40
12:57:32.112040 ens33 In  IP 192.168.1.11 > 192.168.1.10: ICMP echo request, id 1, seq 3, length 40
5 packets captured
6 packets received by filter
0 packets dropped by kernel
```

#### 5.3. Мандатное управление доступом (MAC)

Необходимо убедиться, что SELinux работает в режиме Enforcing, и настроить nginx так, чтобы он отдавал файлы из нестандартной директории `/mephi-web`

Проверим SELinux командой `getenforce `
Получим вывод:
```
Enforcing
```

Настроим nginx. Редактируем основной конфиг /etc/nginx/nginx.conf и меняем значение параметра root
По-умолчанию nginx ищет файлы в каталоге  /usr/share/nginx/html, поэтому изменим его на /mephi-web

После правки конфига, проверим правильность командой sudo nginx -t
Вывод команды:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Конфиг корректный, перезапустим nginx командой `sudo systemctl reload nginx`

Установим необходимые пакеты и настроим контекст SELinux для /mephi-web
```
sudo dnf install -y policycoreutils-python-utils
sudo semanage fcontext -a -t httpd_sys_content_t "/mephi-web(/.*)?"
sudo restorecon -Rv /mephi-web
ls -dZ /mephi-web
```







## Раздел 6. Аутентификация

#### 6.1. Ограничение входа для пользователей

Ограничим пользователям curator1 и curator2 возможность локального входа в систему
```
sudo usermod -s /usr/sbin/nologin curator1
sudo usermod -s /usr/sbin/nologin curator2
```

Проверим командой `getent passwd curator1 curator2`
```
curator1:x:5504:5504::/home/curator1:/usr/sbin/nologin
curator2:x:5505:5505::/home/curator2:/usr/sbin/nologin
```

А остальные пользователи могут входить. Проверим командой `getent passwd user1 user2 user3`
```
user1:x:5501:5501::/home/user1:/bin/bash
user2:x:5502:5502::/home/user2:/bin/bash
user3:x:5503:5503::/home/user3:/bin/bash
```

Помимо этого, проверим командами:
```
sudo -u curator1 login
sudo -u user1 login
```
Вывод команд:
```
[alex@mephi-2026 ~]$ sudo su - curator1
This account is currently not available.
[alex@mephi-2026 ~]$ sudo su - user1
```
#### 6.2. Управление паролями

Установим срок пароля для новых пользователей отредактировав файл:
```
sudo nano /etc/login.defs
```

Изменим строку PASS_MAX_DAYS (по-умолчанию установлено значение 99999) на 90

Проверим:
```
[alex@mephi-2026 ~]$ grep PASS_MAX_DAYS /etc/login.defs
#       PASS_MAX_DAYS   Maximum number of days a password may be used.
PASS_MAX_DAYS   90
```

Установим минимальную длину пароля отредактировав файл:
```
sudo nano /etc/security/pwquality.conf
```

Изменим строку minlen (раскомментируем и заменим значение 8 на 12)
```
[alex@mephi-2026 ~]$ grep minlen /etc/security/pwquality.conf
minlen = 12
```

Изменим срок пароля для уже созданных пользователей (пользователь `alex` созданный во время установки не включен в этот список):
```
sudo chage -M 90 user1
sudo chage -M 90 user2
sudo chage -M 90 user3
sudo chage -M 90 curator1
sudo chage -M 90 curator2
```

Проверка:
```
sudo chage -l user1 
sudo chage -l curator1
```

```
[alex@mephi-2026 ~]$ sudo chage -l user1
Последний раз пароль был изменён                                : окт 28, 2026
Срок действия пароля истекает                                   : янв 26, 2027
Пароль будет деактивирован через                                : никогда
Срок действия учётной записи истекает                           : никогда
Минимальное количество дней между сменой пароля                 : 0
Максимальное количество дней между сменой пароля                : 90
Количество дней с предупреждением перед деактивацией пароля     : 7
[alex@mephi-2026 ~]$ sudo chage -l curator1
Последний раз пароль был изменён                                : окт 28, 2026
Срок действия пароля истекает                                   : янв 26, 2027
Пароль будет деактивирован через                                : никогда
Срок действия учётной записи истекает                           : никогда
Минимальное количество дней между сменой пароля                 : 0
Максимальное количество дней между сменой пароля                : 90
Количество дней с предупреждением перед деактивацией пароля     : 7
```

Проверим минимальную длину пароля командой (команда возвращает цифру - надежность пароля):
```
[alex@mephi-2026 ~]$ echo 'minipass' | pwscore
Проверка сложности пароля завершилась неудачей:
Пароль должен содержать не менее 12 символов
[alex@mephi-2026 ~]$ echo 'verybigpassword' | pwscore
45
```

## Раздел 7. Тестирование

#### 7.1. Создание web-страницы

Убедимся в том, что раздел смонтирован
```
[alex@mephi-2026 ~]$ mount | grep mephi-web
/dev/sdb1 on /mephi-web type ext4 (rw,relatime,seclabel)
```

Создадим файл index.html
```
[alex@mephi-2026 ~]$ echo "Hello from Student: М265224" | sudo tee /mephi-web/index.html
Hello from Student: М265224
```
#### 7.2. Проверка доступности

Запросим страницу через curl:
```
[alex@mephi-2026 ~]$ curl http://localhost/
Hello from Student: М265224
```





