# Demo2025

## Топология №1.

![Снимок экрана (1)](https://github.com/user-attachments/assets/37a24fe6-c981-45c7-9220-683a2d091c96)


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
| BR-RTR         | gi1/0/1   | 172.16.5.2  |       /28       | 172.16.5.1  |
|                | gi1/0/2   | 192.168.1.1 |       /27       |             |
| HQ-SRV         | ens192    | 192.168.0.2 |       /26       | 192.168.0.1 |
| HQ-CLI         | ens192    | DHCP        |       /28       | 192.168.0.65|
| BR-SRV         | ens192    | 192.168.1.2 |       /27       | 192.168.1.1 |


### 1. Настройка интерфесов на BR-SRV, HQ-CLI, HQ-SRV.

Проверить существующие адреса. `ip a`

После этого приступим к настройке статической маршрутизации  
Нужно открыть файл options для нужного интерфейса:  
```
nano /etc/net/ifaces/ens192/options
```
Там поменял все как на примере:   
```
BOOTPROTO=static
TYPE=eth
CONFIG_WIRELESS=no
SYSTEMD_BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no
```
Если нет нужного интерфейса прописыаем такую команду:
```+
mkdir /etc/net/ifaces/xxx
```
После этого задал нужный адрес на интрефейс из таблицы №2:  
```
echo xxx.xxx.xxx.xxx/xx > /etc/net/ifaces/ensxxx/ipv4address
```
Если нужно добавить шлюз по умолчанию, то нужна эта команда:  
```
echo default via xxx.xxx.xxx.xxx > /etc/net/ifaces/xxx/ipv4route
```
`Вместо x, нужно вставить IP-адрес и номер интерфейса из таблицы №2`  
Если нужно указать информацию о DNS-сервере, прописываем команду:  
```
echo nameserver 8.8.8.8 > /etc/resolv.conf
```
После этого перезагружаем сетевую службу:  
```
service network restart

##Настройка IP адресов на ISP

```
configure

int gi1/0/3
ip address 172.16.4.1/28
no shutdown

int gi1/0/2
ip address 172.16.5.1/28
no shutdown

commit
confirm
```

## настройка IP адресов на HQ-RTR - EcoRouter

Создаем сущность интерфейса и назначаем IP

```
int TO-ISP
ip address 172.16.4.2/28
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

Создаем интерфейсы, которые будут обрабатывать трафик vlan 100, 200, 999

```
interface HQ-SRV
 ip mtu 1500
 ip address 192.168.0.1/26

interface HQ-CLI
 ip mtu 1500
 ip address 192.168.0.65/28

interface HQ-MGMT
 ip mtu 1500
 ip address 192.168.0.81/29
```

Заходим на порт и создаем для каждой `vlan` свой `service-instance`.

1. Заходим в `port ge1`.
2. Создаем service-instance `service-instance ge1/vlan100`
3. Указываем инкапсуляцию для `100 vlan`
4. Чтобы кадры из этого интерфейса выходили с тегом задаем настройку `rewrite pop 1`
5. Привязываем сущьность интрефейса к порту `connect ip interface HQ-SRV`

Проделываем эти действия для vlan 200 и 300

```
port ge1
 mtu 9234
 service-instance ge1/vlan100
  encapsulation dot1q 100
  rewrite pop 1
  connect ip interface HQ-SRV
 service-instance ge1/vlan200
  encapsulation dot1q 200
  rewrite pop 1
  connect ip interface HQ-CLI
 service-instance ge1/vlan999
  encapsulation dot1q 999
  rewrite pop 1
  connect ip interface HQ-MGMT
```

<img src="02.png" width='600'>

show run

```
HQ-RTR#sh run

no service password-encryption

hostname HQ-RTR


hw mgmt ip 192.168.255.1/24

bgp extended-asn-cap 

ip vrf management

class-map default

security default

security none vrf management

mpls propagate-ttl
mpls label mode per-prefix
mpls exp stack-size all

ip domain-lookup


ip pim register-rp-reachability

line con 0
line vty 0 39

port ge0
 mtu 9234
 service-instance SI-ISP
  encapsulation untagged

port ge1
 mtu 9234
 service-instance ge1/vlan100
  encapsulation dot1q 100
  rewrite pop 1
 service-instance ge1/vlan200
  encapsulation dot1q 200
  rewrite pop 1
 service-instance ge1/vlan999
  encapsulation dot1q 999
  rewrite pop 1

interface TO-ISP
 ip mtu 1500
 connect port ge0 service-instance SI-ISP
 ip address 172.16.4.2/28

interface HQ-SRV
 ip mtu 1500
 connect port ge1 service-instance ge1/vlan100
 ip address 192.168.0.1/26

interface HQ-CLI
 ip mtu 1500
 connect port ge1 service-instance ge1/vlan200
 ip address 192.168.0.65/28

interface HQ-MGMT
 ip mtu 1500
 connect port ge1 service-instance ge1/vlan999
 ip address 192.168.0.81/29

arp request-interval 1
arp request-number 3
arp expiration-period 5
arp solicitation-rate 2
arp ip-collision-time 8
arp incomplete-time 60

end
```

## HQ-SRV

В заходим в деректорию /etc/net/ifaces/ensxx/options `nano` или `vim` и изменяем параметры:

```
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
```

```
echo 192.168.0.2/26 > /etc/net/ifaces/ens192/ipv4address
```

```
echo default via 192.168.0.1 > /etc/net/ifaces/ens192/ipv4route
```

## HQ-CLI









