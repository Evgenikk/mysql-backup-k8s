Посмотреть список таблиц в базе:
kubectl exec -it $MYSQLPOD -- mysql -ppassword2019 -e "use operator-db; show tables;"
Посмотреть список баз:
kubectl exec -it $MYSQLPOD  -- mysql -ppassword2019 -e "show databases"

```
mysql: [Warning] Using a password on the command line interface can be insecure.
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| operator-db        |
| performance_schema |
| sys                |
+--------------------+
```


Создаем и заполняем таблицу test в db operator-db:
```
kubectl exec -it $MYSQLPOD -- mysql -u root  -ppassword2019 -e "CREATE TABLE test ( id smallint unsigned not null auto_increment, name varchar(20) not null, constraint pk_example primary key (id) );" operator-db

kubectl exec -it $MYSQLPOD -- mysql -ppassword2019  -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data' );" operator-db

kubectl exec -it $MYSQLPOD -- mysql -ppassword2019  -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data-2' );" operator-db
```

Проверяем, содержимое:
```
kubectl exec -it $MYSQLPOD -- mysql -ppassword2019  -e "select * from test;" operator-db
```

+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+

Запускаем backup-job:
```
kubectl apply -f jobs/backup-job.yml
```
Удаляем таблицу, проверям, что все стало чисто:
```
kubectl exec -it $MYSQLPOD -- mysql -ppassword2019  -e "DROP table test;" operator-db
kubectl exec -it $MYSQLPOD -- mysql -ppassword2019  -e "select * from test;" operator-db
```
ERROR 1146 (42S02) at line 1: Table 'operator-db.test' doesn't exist
command terminated with exit code 1

Восстанавливаемся при помощи resotre-job:
```
kubectl apply -f jobs/restore-job.yml
```

Снова проверяем содержимое таблицы test:

```
kubectl exec -it $MYSQLPOD -- mysql -ppassword2019  -e "select * from test;" operator-db
```

+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
