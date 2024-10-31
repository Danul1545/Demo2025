# Demo2025

1. Настройка IP  [-->](./IP_add/README.md) 

2. Настройка Пользователей  [-->](./Users/README.md)

3. Настройка OSPF  [-->](./OSPF/README.md)

4. Настройка NAT  [-->](./NAT/README.md)

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
