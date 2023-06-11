# «Кластеризация и балансировка нагрузки» - Леонид Хорошев

### Задание 1
- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy.
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

Ссылка на конфигурационный файл haproxy.cfg

https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/haproxy.cfg

Скриншот с перенаправлением запросов

![alt text](https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/balancing1.1.png)

### Задание 2
- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

Ссылка на конфигурационный файл haproxy2.cfg, измененный по условиям Задания 2.

https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/haproxy2.cfg

Скриншот с перенаправлением запросов

![alt text](https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/balancing2.1.png)

---
### Задание 3*
- Настройте связку HAProxy + Nginx как было показано на лекции.
- Настройте Nginx так, чтобы файлы .jpg выдавались самим Nginx (предварительно разместите несколько тестовых картинок в директории /var/www/), а остальные запросы переадресовывались на HAProxy, который в свою очередь переадресовывал их на два Simple Python server.
- На проверку направьте конфигурационные файлы nginx, HAProxy, скриншоты с запросами jpg картинок и других файлов на Simple Python Server, демонстрирующие корректную настройку.

С данным заданием возникли сложности, если настроить Nпinx с переадресацией HAProxy удалось (сриншот ниже), то вот с открытием jpg файлов возникли проблемы, видимо не удалось корректно прописать данное условие в nginx.conf. При попытке открыть страницу через ngnix выходит стандартная страница index.html вместо картинок.

![alt text](https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/balancing3.1.png)

Неудачная попытка (тут должна была быть картинка вместо index.html)

![alt text](https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/balancing3.2.png)

Конфигурационные файлы Nginx:

https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/nginx.conf

https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/example-http.conf

Конфигурационный файл haproxy:

https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/haproxy3.cfg

---

### Задание 4*
- Запустите 4 simple python сервера на разных портах.
- Первые два сервера будут выдавать страницу index.html вашего сайта example1.local (в файле index.html напишите example1.local)
- Вторые два сервера будут выдавать страницу index.html вашего сайта example2.local (в файле index.html напишите example2.local)
- Настройте два бэкенда HAProxy
- Настройте фронтенд HAProxy так, чтобы в зависимости от запрашиваемого сайта example1.local или example2.local запросы перенаправлялись на разные бэкенды HAProxy
- На проверку направьте конфигурационный файл HAProxy, скриншоты, демонстрирующие запросы к разным фронтендам и ответам от разных бэкендов.

Конфигурационнй файл haproxy (в нем мы используем две секции backend - example1 и example2)

https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/haproxy4.cfg

Скриншот работы сервиса

![alt text](https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/clustering-and-balancing/balancing4.png)

------


