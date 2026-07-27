# Домашнее задание 28: 

## Задания
Дано<br>

https://github.com/erlong15/otus-linux/tree/network
(ветка network)<br>

Vagrantfile с начальным построением сети<br>

inetRouter<br>
centralRouter<br>
centralServer<br>
тестировалось на virtualbox<br>


Планируемая архитектура<br>

построить следующую архитектуру<br>

Сеть office1<br>

192.168.2.0/26 - dev<br>
192.168.2.64/26 - test servers<br>
192.168.2.128/26 - managers<br>
192.168.2.192/26 - office hardware<br>


Сеть office2<br>

192.168.1.0/25 - dev<br>
192.168.1.128/26 - test servers<br>
192.168.1.192/26 - office hardware<br>


Сеть central<br>

192.168.0.0/28 - directors<br>
192.168.0.32/28 - office hardware<br>
192.168.0.64/26 - wifi<br>

```
Office1 ---\
                   -----> Central --IRouter --> internet                
Office2----/
```
Итого должны получится следующие сервера<br>

inetRouter<br>
centralRouter<br>
office1Router<br>
office2Router<br>
centralServer<br>
office1Server<br>
office2Server<br>

Теоретическая часть<br>

Найти свободные подсети<br>
Посчитать сколько узлов в каждой подсети, включая свободные<br>
Указать broadcast адрес для каждой подсети<br>
проверить нет ли ошибок при разбиении<br>

Практическая часть<br>
Соединить офисы в сеть согласно схеме и настроить роутинг<br>
Все сервера и роутеры должны ходить в инет черз inetRouter<br>
Все сервера должны видеть друг друга<br>
у всех новых серверов отключить дефолт на нат (eth0), который вагрант поднимает для связи<br>
при нехватке сетевых интервейсов добавить по несколько адресов на интерфейс<br>


Формат сдачи ДЗ - vagrant + ansible


## Структура
├── README.md<br>
└── 



## Выполнение

### 
```
```

### 
```
```

### 
```
```

### 
```
```

### 
```
```
