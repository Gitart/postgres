Полезные запросы:

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

 SELECT tableName, pg_size_pretty(pg_total_relation_size(CAST(tablename as text))) as size
from pg_tables
where tableName not like ‘sql_%’
order by size;   

Пользователи блокирующие конкретную таблицу
Код SQL

 select a.usename, t.relname, a.current_query, mode from pg_locks l inner join pg_stat_activity a on a.procpid = l.pid inner join pg_stat_all_tables t on t.relid=l.relation where t.relname = ‘tablename’;   

Код SQL

 select relation::regclass, mode, a.usename, granted, pid from pg_locks l inner join pg_stat_activity a on a.procpid = l.pid where not mode = ‘AccessShareLock’ and relation is not null;   

Запросы с эксклюзивными блокировками
Код SQL

 select a.usename, a.current_query, mode from pg_locks l inner join pg_stat_activity a on a.procpid = l.pid where mode ilike ‘%exclusive%’;   

Количество блокировок по пользователям
Код SQL

 select a.usename, count(l.pid) from pg_locks l inner join pg_stat_activity a on a.procpid = l.pid where not(mode = ‘AccessShareLock’) group by a.usename;   

Количество подключений по пользователям
Код SQL

 select count(usename), usename from pg_stat_activity group by usename order by count(usename) desc;   
