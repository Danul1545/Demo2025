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
2. Создаем service-instance `service-instance SI-ISP`
3. Указываем, что кадры на этом интерфейсе будут без тега `encapsulation untagged`
4. Привязываем сущьность интрефейса к порту `connect ip interface TO-ISP`

```
port ge0
service-instance SI-ISP
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

Заходим на порт и создаем для каждой `vlan` свой `service-instance`.

1. Заходим в `port ge1`.
2. Создаем service-instance `service-instance ge1/vlan10`
3. Указываем инкапсуляцию для `10 vlan`
4. Чтобы кадры из этого интерфейса выходили с тегом задаем настройку `rewrite pop 1`
5. Привязываем сущьность интрефейса к порту `connect ip interface HQ-SRV`

Проделываем эти действия для vlan 20 и 30

```
port ge1
 mtu 9234
 service-instance ge1/vlan10
  encapsulation dot1q 10
  rewrite pop 1
  connect ip interface HQ-SRV
 service-instance ge1/vlan20
  encapsulation dot1q 20
  rewrite pop 1
  connect ip interface HQ-CLI
 service-instance ge1/vlan30
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



