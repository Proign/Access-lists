# Стандартные списки доступа (Standard IP Access-Lists)
На маршрутизаторе **2600A** создадим список доступа, который заблокирует доступ от узла **F** на подсеть `172.16.40.0`:
```
2600A#config t
2600A(config)#access-list 10 deny host 172.16.50.3
2600A(config)#access-list 10 permit any
```

Стандартные списки доступа должны быть применены как можно ближе к сети/узлу назначения, поэтому список создан на **2600A**.
Применим список доступа к интерфейсу `serial 0/0` маршрутизатора **2600A** для входящего в интерфейс потока пакетов:
```
2600A(config)#interface serial 0/0
2600A(config-if)#ip access-group 10 in
```

Убедимся, что узел **F** не может больше пинговать `172.16.40.3`:

![HostF](https://github.com/Proign/Access-lists/blob/main/screenshots/HostF-ping.PNG)

Проверим, что все остальные устройства по-прежнему могут пинговать подсеть `172.16.40.0.` Сделаем, например, Ping с **2600C** на `172.16.40.3`:

![2600C](https://github.com/Proign/Access-lists/blob/main/screenshots/2600C-ping.PNG)

# Проверка работы стандартных списков доступа
Кроме `ping`, `traceroute`, `telnet`, существуют и другие способы проверки списков доступа.
Выполним на **2600A** `show access-list`, `show access-list 10` и `show ip interface`:

![2600A](https://github.com/Proign/Access-lists/blob/main/screenshots/2600A-access-list.PNG)

# Применение списков доступа к линиям VTY
Удалим ненужный список доступа на **2600A**:
```
2600A#config t
2600A(config)#no access-list 10
```

Удалим применение списка:
```
2600A(config)#interface serial 0/0
2600A(config-if)#no ip access-group 10 in
```

Проверим, что с узла **F** можно выполнить `telnet` на маршрутизатор **2600A**:

![HostF](https://github.com/Proign/Access-lists/blob/main/screenshots/HostF-telnet.PNG)

На маршрутизаторе **2600A** заблокируем telnet-доступ с узла **F**, но разрешим с любых других адресов:
```
2600A#config t
2600A(config)#access-list 20 deny host 172.16.50.3
2600A(config)#access-list 20 permit any
```

Применим список к линиям 0,1,2,3,4 VTY:
```
2600A(config)#line vty 0 4
2600A(config-line)#access-class 20 in
2600A(config-line)#^z
```

Убедимся, что теперь нельзя сделать `telnet` с узла **F** на **2600A**:

![HostF](https://github.com/Proign/Access-lists/blob/main/screenshots/HostF-no-telnet.PNG)

Убедимся, что можно сделать `telnet` с **2600C** на **2600A**:

![2600C](https://github.com/Proign/Access-lists/blob/main/screenshots/2600C-telnet.PNG)

# Расширенные списки доступа (Extended IP Access-Lists)

Удалим стандартный список доступа с **2600A**:
```
2600A#config t
2600A(config)#no access-list 10
```

Проверим, что хост **F** может теперь выполнять `ping` на `172.16.40.1` и `172.16.40.3`:

![HostF](https://github.com/Proign/Access-lists/blob/main/screenshots/HostF-ping-check.PNG)

Создадим список доступа на **2600A** для блокирования telnet-доступа в подсеть `172.16.40.0`, но, тем не менее, позволяем хосту **F** делать `ping` на хост **E**:
```
2600A#config t
2600A(config)#access-list 110 deny tcp host 172.16.50.3 172.16.40.0 0.0.0.255 eq telnet
2600A(config)#access-list 110 permit ip any any
```

Применяем сформированный таким образом расширенный список доступа к последовательному интерфейсу 0/0 маршрутизатора **2600A** для входящих пакетов:
```
2600A(config)#interface serial 0/0
2600A(config-if)#ip access-group 110 in
2600A(config-if)#^z
```

Проверяем, пытаясь сделать `telnet` на `172.16.40.1` с хоста **F**:

![HostF](https://github.com/Proign/Access-lists/blob/main/screenshots/HostF-another-no-telnet.PNG)

Все прочие устройства имеют возможность делать `telnet` на `172.16.40.1`:

![HostF](https://github.com/Proign/Access-lists/blob/main/screenshots/HostE-telnet.PNG)

# Проверка расширенных списков доступа
Используем те же команды, что и при проверке стандартных списков:

![2600A](https://github.com/Proign/Access-lists/blob/main/screenshots/2600A-extended-access-list.PNG)