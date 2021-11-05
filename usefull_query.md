## Ускорение и оптимизация настроек PostgreSQL 

По умолчанию PostgreSQL настроен таким образом, чтобы расходовать минимальное количество ресурсов для работы с небольшими базами до 4 Gb на не очень производительных серверах. То есть, если дело касается систем посерьезней, то вы столкнетесь с большими потерями производительности базы данных лишь потому, что дефолтные настройки могут в корне не соответствовать производительности вашего северного оборудования. Настройки выделения ресурсов оперативной памяти RAM для работы PostgreSQL хранятся в файле postgresql.conf.


## Доступен как из папки, куда установлен PostgreSQL / Data, так и из pgAdmin:


**shared_buffers**

Это размер памяти, разделяемой между процессами PostgreSQL, отвечающими за выполнения активных операций. Максимально-допустимое значение этого параметра – 25% всего количества RAM

Например, при 1-2 Gb RAM на сервере, достаточно указать в этом параметре значение 64-128 Mb (8192-16384).

**temp_buffers**

Это размер буфера под временные объекты (временные таблицы). Среднее значение 2-4% всего количества RAM

Например, при 1-2 Gb RAM на сервере, достаточно указать в этом параметре значение 32-64 Mb.

**work_mem**

Это размер памяти, используемый для сортировки и кэширования таблиц.

При 1-2 Gb RAM на сервере, рекомендуемое значение 32-64 Mb.

Для вступления новых значений в силу, потребуется перезапуск службы, поэтому лучше делать во вне рабочее время.

Еще два важных параметра это maintenance_work_mem (для операций VACUUM, CREATE INDEX и других) и max_stack_depth
Примеры оптимальных настроек:

### Hardware:
```
    CPU: E3-1240 v3 @ 3.40GHz
    RAM: 32Gb 1600Mhz
    Диски: Plextor M6Pro
```

### postgresql.conf:

```
    shared_buffers = 8GB
    work_mem = 128MB
    maintenance_work_mem = 2GB
    fsync = on
    synchronous_commit = off
    wal_sync_method = fdatasync
    checkpoint_segments = 64
    seq_page_cost = 1.0
    random_page_cost = 6.0
    cpu_tuple_cost = 0.01
    cpu_index_tuple_cost = 0.0005
    cpu_operator_cost = 0.0025
    effective_cache_size = 24GB
``` 
    
    


## Полезные запросы:

Блокировки БД по пользователям
Код SQL

 select a.usename, count(l.pid) from pg_locks l inner join pg_stat_activity a on a.procpid = l.pid where not(mode = ‘AccessShareLock’) group by a.usename;   

Вывести все таблицы, размером больше 10 Мб
Код SQL

 SELECT tableName, pg_size_pretty(pg_total_relation_size(CAST(tablename as text))) as size
from pg_tables
where tableName not like ‘sql_%’ and pg_size_pretty(pg_total_relation_size(CAST(tablename as text))) like ‘%MB%’;   

Определение размеров таблиц в базе данных PostgreSQL
Код SQL

```sql
 SELECT tableName, pg_size_pretty(pg_total_relation_size(CAST(tablename as text))) as size
from pg_tables
where tableName not like ‘sql_%’
order by size;   
```


## Пользователи блокирующие конкретную таблицу
```sql
 select a.usename, t.relname, a.current_query, mode from pg_locks l inner join pg_stat_activity a on a.procpid = l.pid inner join pg_stat_all_tables t on t.relid=l.relation where t.relname = ‘tablename’;   
```

```sql
 select relation::regclass, mode, a.usename, granted, pid from pg_locks l inner join pg_stat_activity a on a.procpid = l.pid where not mode = ‘AccessShareLock’ and relation is not null;   
```

## Запросы с эксклюзивными блокировками
```sql
 select a.usename, a.current_query, mode from pg_locks l inner join pg_stat_activity a on a.procpid = l.pid where mode ilike ‘%exclusive%’;   
```

## Количество блокировок по пользователям

```sql
 select a.usename, count(l.pid) from pg_locks l inner join pg_stat_activity a on a.procpid = l.pid where not(mode = ‘AccessShareLock’) group by a.usename;   
```

## Количество подключений по пользователям
```sql
 select count(usename), usename from pg_stat_activity group by usename order by count(usename) desc;   
 ```
 
