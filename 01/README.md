## ДЗ 1

## Создаем виртуальную машину и подключаемся к ней
[Работа с виртуальными машинами в Yandex Cloud](../create_ya_vm.md)

## Устанавливаем PostgreSQL 18 на ВМ
[Установка PostgreSQL](../install_postgresql_18.md)

## Скачиваем тестовую базу данных
[Читаем инструкцию по установке тестовой базы данных](https://github.com/aeuge/postgres16book/tree/main/database)

```bash
wget -P /tmp/ https://storage.googleapis.com/thaibus/thai_small.tar.gz
```

## Переключаемся на пользователя postgres
```bash
sudo -i -u postgres
```

## Импортируем тестовую базу данных
```bash
tar -xf /tmp/thai_small.tar.gz -C /tmp/ && psql < /tmp/thai.sql
```

## Проверяем успешный импорт базы данных
```bash
postgres-# \l # список баз данных
postgres-# \c thai # подключаемся к базе данных thai
thai=# \dn # список схем в базе данных
thai-# \dt book.* # список таблиц в схеме book
thai=# select count(*) from book.tickets; # считаем количество записей в таблице book.tickets
#   count
# ---------
#  5185505
# (1 row)
```
