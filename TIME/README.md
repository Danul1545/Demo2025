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
