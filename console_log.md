# Лог выполения домашнего задания №1
## Создание таблицы с данными (session 1)
```
    postgres=# begin;
    BEGIN
    postgres=*# create table persons (id serial, first_name text, second_name text);
    CREATE TABLE
    postgres=*# insert into persons(first_name, second_name) values('ivan', 'ivanov');
    INSERT 0 1
    postgres=*# insert into persons(first_name, second_name) values('petr', 'petrov');
    INSERT 0 1
    postgres=*# commit;
    COMMIT
    postgres=# show transaction isolation level;
     transaction_isolation 
    -----------------------
     read committed
    (1 row)
```

### Результат
Таблица создана.

## Проверка уровня изоляции и добавление записи в таблицу (session 1)
```
    postgres=# begin;
    BEGIN
    postgres=*# insert into persons(first_name, second_name) values('sergey', 'sergeev');
    INSERT 0 1
    postgres=*# select * from persons;
     id | first_name | second_name 
    ----+------------+-------------
      1 | ivan       | ivanov
      2 | petr       | petrov
      3 | sergey     | sergeev
    (3 rows)
```

### Результат
Запись добавлена, но транзакция не завершена.

## Проверка наличия новой записи из второй сессии (session 2)
```
    postgres=# begin;
    BEGIN
    postgres=*# select * from persons
    postgres-*# ;
     id | first_name | second_name 
    ----+------------+-------------
      1 | ivan       | ivanov
      2 | petr       | petrov
    (2 rows)
```

### Результат
Новая запись не видна, т.к. на данном уровне изоляции видны изменения, для которых сделан commit.

## Завершение транзакции первой сессии записью (session 1)
```
    postgres=*# commit;
    COMMIT
```

### Результат
Транзакция успешно завершена.

## Проверка наличия новой записи из второй сессии (session 2)
```
    postgres=*# select * from persons
    ;
     id | first_name | second_name 
    ----+------------+-------------
      1 | ivan       | ivanov
      2 | petr       | petrov
      3 | sergey     | sergeev
    (3 rows)
```

### Результат
Запись видна, потому что транзакция в первой сессии завершена сохранением (commit).

## Завершение транзакции для второй сессии (session 2)
```
    postgres=*# commit;
    COMMIT
```

### Результат
Успешно завершена.

## Старт новой транзакции и изменение уровня изоляции на repeatable read (session 1)
```
    postgres=# begin;
    BEGIN
    postgres=*# set transaction isolation level repeatable read;
    SET
```

### Результат
Транзакция открыта, уровень изоляции изменен.

## Старт новой транзакции и изменение уровня изоляции на repeatable read (session 2)
```
    postgres=# begin;
    BEGIN
    postgres=*# set transaction isolation level repeatable read;
    SET
```

### Результат
Транзакция открыта, уровень изоляции изменен.

## Добавление новой записи в таблицу presons (session 1)
```
    postgres=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
    INSERT 0 1
    postgres=*# select * from persons;
     id | first_name | second_name 
    ----+------------+-------------
      1 | ivan       | ivanov
      2 | petr       | petrov
      3 | sergey     | sergeev
      4 | sveta      | svetova
    (4 rows)
```

### Результат
Запись добавлена.

## Проверка добавления новой записи в таблицу persons (session 2)
```
    postgres=*# select * from persons;
     id | first_name | second_name 
    ----+------------+-------------
      1 | ivan       | ivanov
      2 | petr       | petrov
      3 | sergey     | sergeev
    (3 rows)
```

### Результат
Запись не отображается, т.к. установленный уровень изоляции строже чем read committed, но при этом транзакция из первой сессии еще __даже__ не зафиксирована.
Кроме этого для того, для работы с новой записью требуется начать транзакцию (с учетом того, что мы все действия выполняем в транзакциях) в session 2 после окончания транзакции в session 1.    

## Завершение транзакции первой сессии записью (session 1)
```
    postgres=*# commit;
    COMMIT
```

### Результат
Успешно завершена.

## Проверка добавления новой записи в таблицу persons (session 2)
```
    postgres=*# select * from persons;
     id | first_name | second_name 
    ----+------------+-------------
      1 | ivan       | ivanov
      2 | petr       | petrov
      3 | sergey     | sergeev
    (3 rows)
```

### Результат
Запись не отображается, т.к. транзакция в данной сессии была начата до commit в session 1.

## Завершение транзакции первой сессии записью (session 2)
```
    postgres=*# commit;
    COMMIT
```

### Результат
Успешно завершена.

## Проверка добавления новой записи в таблицу persons (session 2)
```
    postgres=# select * from persons;
     id | first_name | second_name 
    ----+------------+-------------
      1 | ivan       | ivanov
      2 | petr       | petrov
      3 | sergey     | sergeev
      4 | sveta      | svetova
    (4 rows)
```

### Результат
Запись отображается. Запуск команды выборки без явного старта транзакции равносилен транзакции из одной команды. 
В результате получается, что новая транзакция начата после сохранения изменения предыдущев в session 1.

