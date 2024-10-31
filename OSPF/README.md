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

![image](https://github.com/user-attachments/assets/d8209a8f-f632-457d-8e02-dc8a23b776d2)


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
