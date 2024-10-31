# Demo2025

1. Настройка IP  [-->](./IP_add/README.md) 

2. Настройка Пользователей  [-->](./Users/README.md)

3. Настройка OSPF  [-->](./OSPF/README.md)

## Топология №1.

![Снимок экрана (15)](https://github.com/user-attachments/assets/6254982b-e9ee-4b59-bb7a-c9da1cf8c9bf)


### Таблица №2 (Операциоенные системы).

| Имя устройства | Процессор | Оперативная память | Жёский диск |     Операционная Система   |
| :------------: | :-------: |  :---------------: | :---------: |       :-------------:      |
| ISP            |  2 (ГЦ)   |        4 (ГБ)      |   8 (ГБ)    |            Eltex           |
| HQ-RTR         |  2 (ГЦ)   |        4 (ГБ)      |   6 (ГБ)    |          EcoRouter         |
| BR-RTR         |  2 (ГЦ)   |        4 (ГБ)      |   8 (ГБ)    |            Eltex           |
| HQ-SRV         |  2 (ГЦ)   |        4 (ГБ)      |  35 (ГБ)    |      ALT Linux Server      |
| BR-SRV         |  2 (ГЦ)   |        4 (ГБ)      |  35 (ГБ)    |      ALT Linux Server      |
| HQ-CLI         |  2 (ГЦ)   |        4 (ГБ)      |  35 (ГБ)    |    ALT Linux WorkStation   |


### Таблица №2 (разбита на подсети).

| Имя устройства | Интерфейс |     IP      |      Маска      |     Шлюз    |
| :------------: |:---------:| :----------:| :-------------: | :---------: |
| ISP            | gi1/0/1   | DHCP        |                 |             |
|                | gi1/0/2   | 172.16.5.1  |       /28       |             |
|                | gi1/0/3   | 172.16.4.1  |       /28       |             |
| HQ-RTR         | ge0       | 172.16.4.2  |       /28       | 172.16.4.1  |
|                | ge1.100   | 192.168.0.1 |       /26       |             |
|                | ge1.200   | 192.168.0.65|       /28       |             |
|                | ge1.999   | 192.168.0.81|       /29       |             |
|                | tunnel.1  | 172.16.1.1  |       /30       |             |
| BR-RTR         | gi1/0/1   | 172.16.5.2  |       /28       | 172.16.5.1  |
|                | gi1/0/2   | 192.168.1.1 |       /27       |             |
|                | gre1      | 172.16.1.2  |       /30       |             |
| HQ-SRV         | ens192    | 192.168.0.2 |       /26       | 192.168.0.1 |
| HQ-CLI         | ens192    | DHCP        |       /28       | 192.168.0.65|
| BR-SRV         | ens192    | 192.168.1.2 |       /27       | 192.168.1.1 |

## Настраиваем OSPF

### HQ-RTR

```
router ospf 1
 network 172.16.1.0 0.0.0.3 area 0.0.0.0
 network 192.168.0.0 0.0.0.255 area 0.0.0.0
!
interface tunnel.1
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 Demo2025
 ip ospf network point-to-point
```

### BR-RTR

```
key-chain ospfkey
  key 1
    key-string ascii-text Demo
  exit
exit

router ospf 1
  area 0.0.0.0
    enable
  exit
  enable
exit

interface gi1/0/2
  ip ospf int 1
  ip ospf
exit
tunnel gre 1
  ip ospf int 1
  ip ospf network point-to-point
  ip ospf authentication key-chain ospfkey
  ip ospf authentication algorithm md5
  ip ospf
exit

commit
confirm
```

Проверка

### HQ-RTR

```
sh ip ospf neighbor 
sh ip ospf int br
sh ip route
```

[<img src="01.png" width='600'>](https://github.com/netadmin-str/demo2025/blob/main/ospf_conf/01.png)

### BR-RTR

```
sh ip ospf neighbor
sh ip route
```

![image](https://github.com/user-attachments/assets/99d937a0-bb74-4c0d-8609-2572ab8ebdac)

```
sh ip ospf interface
```

![image](https://github.com/user-attachments/assets/086c7fb8-6025-45c5-810a-3673ef7f74ab)

## Настройка NAT

### ISP

```
object-group network LOCAL_NET
  ip address-range 172.16.5.1-172.16.5.2
  ip address-range 172.16.4.1-172.16.4.2
exit

nat source
  ruleset SNAT
    to interface gi1/0/1
    rule 1
      match source-address LOCAL_NET
      action source-nat interface
      enable
    exit
  exit
exit
```

### HQ-RTR

```
interface TO-ISP
 ip nat outside
!
interface HQ-SRV
 ip nat inside
!
interface HQ-CLI
 ip nat inside
!
interface HQ-MGMT
 ip nat inside
!
ip nat pool NAT_POOL 192.168.0.1-192.168.0.254
!
ip nat source dynamic inside-to-outside pool NAT_POOL overload interface TO-ISP
```

### BR-RTR

```
object-group network LOCAL_NET
  ip address-range 192.168.1.1-192.168.1.30
exit

nat source
  ruleset SNAT
    to interface gi1/0/1
    rule 1
      match source-address LOCAL_NET
      action source-nat interface
      enable
    exit
  exit
exit
```

## Настройка DHCP на HQ-RTR (EcoRouter)

```
ip pool HQ-NET 1
 range 192.168.0.66-192.168.0.70
!
dhcp-server 1
 lease 86400
 mask 255.255.255.0
 pool HQ-NET 1
  dns 192.168.0.2
  domain-name au-team.irpo
  gateway 192.168.0.65
  mask 255.255.255.240
!
interface HQ-CLI
 dhcp-server 1
!
```
## Настройка SSH на HQ-SRV и BR-SRV

Делаем на всякий случай бэкап конфига:

```
cd /etc/openssh
cp -a sshd_config sshd_config.bak
```

Редактируем файл конфигурации SSH сервера

```
nano sshd_config
```

Изменяем следующие параметры. Не забываем их раскоментировать. Если какой-то параметр не находиться, то просто добавьте его сами

```
Port 2024
MaxAuthTries 3
Banner /etc/openssh/banner
AllowUsers sshuser
```

Создаем файл с баннером

```
nano /etc/openssh/banner
```

Вставляем в него следующие содержимое

```
******************************************************
*                     WARNING!                       *
*                                                    *
*           Access only authorized users!            *
*                                                    *
*  If you not authorized user, close this session!   *
*                                                    *
******************************************************
```

Перезагружаем `SSH`

```
systemctl restart sshd
```

Проверяем командой
```
ssh sshuser@::1 -p 2024
```

## Проверка и настройка времени

Проверяем какой часовой пояс установлен

```
timedatectl status
```

Если отличается, то устанавливаем
 
```
timedatectl set-timezone Asia/Yekaterinburg
```

### HQ-RTR (EcoRouter)

```
conf t
ntp timezone utc+5
```
Проверяем:

```
show ntp timezone
```

### BR-RTR (Eltex)

```
configure
clock timezone gmt +5
end
commit
confirm
```

Проверяем:

```
show date
```
