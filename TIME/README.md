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


хааа[https://docs.google.com/document/u/0/d/1gw-aewMqPLfv1nFOYQZU4Svufjf3aajY/mobilebasic#heading=h.ceqh8ghhevp0]
