# Demo2025

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


## Настраиваем IP адреса

### ISP

```
configure

int gi1/0/3
ip add 172.16.4.1/28
ip firewall disable
no shutdown

int gi1/0/2
ip add 172.16.5.1/28
ip firewall disable
no shutdown

commit
confirm
```

### HQ-RTR - EcoRouter

Создаем сущность интерфейса и назначаем IP

```
int TO-ISP
ip add 172.16.4.2/28
no shutdown
```

Привязываем созданный интерфейс к физическому протоколу

1. Заходим в `port ge0`
2. Создаем service-int `service-int SI-ISP`
3. Указываем, что кадры на этом интерфейсе будут без тега `encapsulation untagged`
4. Привязываем сущьность интрефейса к порту `connect ip interface TO-ISP`

```
port ge0
service-int SI-ISP
encapsulation untagged
connect ip interface TO-ISP
```

Создаем интерфейсы, которые будут обрабатывать трафик vlan 10, 20, 30

```
interface HQ-SRV
 ip mtu 1500
 ip add 192.168.0.1/26
!
interface HQ-CLI
 ip mtu 1500
 ip add 192.168.0.65/28
!
interface HQ-MGMT
 ip mtu 1500
 ip add 192.168.0.81/29
!
```

Заходим на порт и создаем для каждой `vlan` свой `service-int`.

1. Заходим в `port ge1`.
2. Создаем service-int `service-int ge1/vlan10`
3. Указываем инкапсуляцию для `10 vlan`
4. Чтобы кадры из этого интерфейса выходили с тегом задаем настройку `rewrite pop 1`
5. Привязываем сущьность интрефейса к порту `connect ip interface HQ-SRV`

Проделываем эти действия для vlan 20 и 30

```
port ge1
 mtu 9234
 service-int ge1/vlan10
  encapsulation dot1q 10
  rewrite pop 1
  connect ip interface HQ-SRV
 service-int ge1/vlan20
  encapsulation dot1q 20
  rewrite pop 1
  connect ip interface HQ-CLI
 service-int ge1/vlan30
  encapsulation dot1q 30
  rewrite pop 1
  connect ip interface HQ-MGMT
```

Создаем GRE туннель

```
int tunnel.1
 ip mtu 1400
 ip add 172.16.1.1/30
 ip tunnel 172.16.4.2 172.16.5.2 mode gre
```

Задаем маршрут по умолчанию в сторону ISP

```
ip route 0.0.0.0/0 172.16.4.1
```

### HQ-SRV

командой `nano` заходим в файл конфигурации и настраиваем:

```
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
```

```
echo 192.168.0.2/26 > /etc/net/ifaces/ens192/ipv4add
```

```
echo default via 192.168.0.1 > /etc/net/ifaces/ens192/ipv4route
```

```
systemctl restart network
```

### BR-RTR

```
configure

int gi1/0/1
  ip firewall disable
  ip add 172.16.5.2/28
  no sh
exit
interface gi1/0/2
  ip firewall disable
  ip add 192.168.1.1/27
  no sh
exit
tunnel gre 1
  ttl 16
  mtu 1400
  ip firewall disable
  local add 172.16.5.2
  remote add 172.16.4.2
  ip add 172.16.1.2/30
  enable
exit
ip route 0.0.0.0/0 172.16.5.1

commit
confirm
```

### BR-SRV

командой `nano` заходим в файл конфигурации и настраиваем:

```
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
```

```
echo 192.168.1.2/26 > /etc/net/ifaces/ens192/ipv4add
```

```
echo default via 192.168.1.1 > /etc/net/ifaces/ens192/ipv4route
```

```
echo nameserver 192.168.0.2 > /etc/net/ifaces/ens192/resolv.conf
```

```
systemctl restart network
```

Проверяем командой `ip a`

## Создание локальных учетных записей

### HQ-SRV и BR-SRV

```
useradd -m -u 1010 sshuser
passwd sshuser
```

```
nano /etc/sudoers.d/sshuser
```

```
sshuser ALL=(ALL) NOPASSWD:ALL
```

Проверка:

```
su - sshuser
sudo whoami
```

### HQ-RTR (EcoRouter)

```
conf t
username net_admin
password P@ssw0rd
role admin
activate
```

### BR-RTR (Eltex - vESR)

```
configure
username net_admin
password P@ssw0rd
privilege 15
end
!
commit
confirm
!
```

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




