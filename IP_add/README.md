# Настраиваем IP адреса

## ISP

```
configure

int gi1/0/2
ip add 172.16.5.1/28
ip firewall disable
no shutdown

int gi1/0/3
ip add 172.16.4.1/28
ip firewall disable
no shutdown

commit
confirm
```

## HQ-RTR - EcoRouter

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

## HQ-SRV

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

## BR-RTR

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

## BR-SRV

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
