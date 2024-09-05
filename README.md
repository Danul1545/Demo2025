# Demo2025

## Топология №1.

![Снимок экрана (1)](https://github.com/user-attachments/assets/37a24fe6-c981-45c7-9220-683a2d091c96)

### Таблица №1 (разбита на подсети).

| Имя устройства | Итерфейс |  IPv4/IPv6   | Маска/Префикс |       Шлюз       |
| :------------: | :------: |  :---------: | :-----------: |  :-------------: |
|                | ens 192  | DHCP         | /24           | DHCP             |
| ISP            | ens 224  | 192.168.0.166| /30           |                  |
|                | ens 256  | 192.168.0.162| /30           |                  |
| HQ-R           | ens 192  | 192.168.0.2  | /25           |                  |
|                | ens 224  | 192.168.0.165| /30           |                  |
| BR-R           | ens 192  | 192.168.0.132| /27           |                  |
|                | ens 224  | 192.168.0.161| /30           |                  |
| HQ-SRV         | ens 192  | 192.168.0.1  | /25           | 192.168.0.2      |
| BR-SRV         | ens 192  | 192.168.0.130| /27           | 192.168.0.132    |


