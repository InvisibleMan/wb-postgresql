## ДЗ 2

## Создаем виртуальную машину и подключаемся к ней
[Работа с виртуальными машинами в Yandex Cloud](../create_ya_vm.md)

## Устанавливаем PostgreSQL 18 на ВМ
[Установка PostgreSQL](../install_postgresql_18.md)

## Переключаемся на пользователя postgres (в 2-ух консолях)
```bash
sudo -i -u postgres
```

## Подключаемся к postgres (в 2-ух консолях)
```bash
psql
```

## Создаем базу данных и подключаемся к ней
```bash
CREATE DATABASE wb;
\l
\c wb
```

## Создаем таблицу и добавляем в нее данные
```bash
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE
); 

INSERT INTO users (name, email) VALUES
  ('Alice', 'alice@example.com'), 
  ('Bob', 'bob@example.com'), 
  ('Charlie', 'charlie@example.com');
```

## Исследуем уровень изоляции транзакций: read committed

### Проверим текущий уровень изоляции транзакций
```bash
wb=# show transaction isolation level;
 transaction_isolation
-----------------------
 read committed
(1 row)
```

### Запускаем транзакцию в обеих консолях с уровнем изоляции по-умолчанию (read committed)
```bash
BEGIN;
SELECT * FROM users;
```

### В первой консоли/транзакции добавляем новую запись
```bash
INSERT INTO users (name, email) VALUES ('Dave', 'dave@example.com');
```

### Во второй консоли/транзакции выполняем SELECT и видим ли мы новую запись?
```bash   
SELECT * FROM users;

 id |  name   |        email
----+---------+---------------------
  1 | Alice   | alice@example.com
  2 | Bob     | bob@example.com
  3 | Charlie | charlie@example.com
(3 rows)
```

Запись не видна, postgres не допускает грязное чтение (dirty read) на уровне изоляции read committed.

### Завершаем транзакцию в первой консоли
```bash
COMMIT;
```

### Во второй консоли/транзакции выполняем SELECT
```bash
SELECT * FROM users;

 id |  name   |        email
----+---------+---------------------
  1 | Alice   | alice@example.com
  2 | Bob     | bob@example.com
  3 | Charlie | charlie@example.com
  4 | Dave    | dave@example.com
(4 rows)
```

Теперь запись видна, так как транзакция в первой консоли была успешно завершена (COMMIT). На
уровне read committed возможно неповторяющееся чтение (non-repeatable read).

### Завершаем транзакцию во второй консоли
```bash
COMMIT;
```

## Исследуем уровень изоляции транзакций: repeatable read

### Запускаем новые транзакции, но уже на уровне repeatable read в обеих консолях
```bash
BEGIN transaction isolation level repeatable read;
```

### В первой консоли/транзакции добавляем новую запись
```bash
INSERT INTO users (name, email) VALUES ('Eve', 'eve@example.com');
```

### Во второй консоли/транзакции выполняем SELECT
```bash
SELECT * FROM users;
 id |  name   |        email
----+---------+---------------------
  1 | Alice   | alice@example.com
  2 | Bob     | bob@example.com
  3 | Charlie | charlie@example.com
  4 | Dave    | dave@example.com
(4 rows)
```

Запись не видна, postgres не допускает грязное чтение (dirty read) на уровне изоляции repeatable read.

### Завершаем транзакцию в первой консоли
```bash
COMMIT;
```

### Во второй консоли/транзакции выполняем SELECT
```bash
SELECT * FROM users;
 id |  name   |        email
----+---------+---------------------
  1 | Alice   | alice@example.com
  2 | Bob     | bob@example.com
  3 | Charlie | charlie@example.com
  4 | Dave    | dave@example.com
(4 rows)
```
Запись не видна, так как транзакция во второй консоли была запущена на уровне repeatable read и видит только те данные, которые были в момент ее запуска. На уровне repeatable read возможно фантомное чтение (phantom read).

### Завершаем транзакцию во второй консоли
```bash
COMMIT;
```

### Во второй консоли/транзакции выполняем SELECT
```bash
SELECT * FROM users;
 id |  name   |        email
----+---------+---------------------
  1 | Alice   | alice@example.com
  2 | Bob     | bob@example.com
  3 | Charlie | charlie@example.com
  4 | Dave    | dave@example.com
  5 | Eve     | eve@example.com
(5 rows)
```
Теперь запись видна, так как транзакция в первой консоли была успешно завершена (COMMIT). 

## Выводы
На уровне изоляции read committed транзакция видит только те данные, которые были успешно завершены (COMMIT) другими транзакциями. На уровне изоляции repeatable read транзакция видит только те данные, которые были в момент ее запуска, даже если другие транзакции были успешно завершены (COMMIT) после ее запуска. На уровне repeatable read возможно фантомное чтение (phantom read), когда транзакция видит разные наборы строк при выполнении одного и того же запроса в рамках одной транзакции.
