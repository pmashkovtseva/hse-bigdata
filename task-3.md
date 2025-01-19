# Домашнее задание 3 (настройка Hive)

1. подключаемся к jump node по ssh (`ip` заменить на соответствующий ip)
```
ssh team@ip
```

2. (team@team-74-jn) заходим на name node:
```
ssh team-74-nn
```

3. (team@team-74-nn) устанавливаем postgresql:
```
sudo apt install postgresql
```

4. переключаемся на юзера `postgres`:
```
sudo -i -u postgres
```

5. (postgres@team-74-nn) подключаемся к консоли postgres:
```
psql
```

6. вставляем скрипт:
```sql
CREATE DATABASE metastore; -- создаем базу данных metastore
CREATE USER hive with password 'insert_password'; -- создаем пользователя hive и задаем ему пароль (заменить на собственный)
GRANT ALL PRIVILEGES ON DATABASE "metastore" TO hive; -- передаем пользователю hive права на бд metastore
ALTER DATABASE metastore OWNER TO hive; -- делаем пользователя hive владельцем бд metastore
```

7. выходим из psql:
```
\q
```

8. выходим из пользователя `postgres`:
```
exit
```

9. (team@team-74-nn) открываем конфиг `postgresql.conf`:
```
sudo nano /etc/postgresql/16/main/postgresql.conf
```
в разделе "CONNECTIONS AND AUTHENTICATION" (connection settings) убираем # из listen addresses, добавляем name node:
```
listen_addresses = 'team-74-nn'
```
сохраняем и выходим (write out - enter - exit)

10. открываем конфиг `pg_hba.conf`:
```
sudo nano /etc/postgresql/16/main/pg_hba.conf
```
в разделе "IPv4 local connections" меняем строчку с localhost на
```
host	metastore	hive	jn_ip/32	password
```
(`jn_ip` заменить на соответствующий ip)

сохраняем и выходим (write out - enter - exit)

11. перезапускаем postgresql:
```
sudo systemctl restart postgresql
```

12. проверяем статус postgresql:
```
sudo systemctl status postgresql
```

13. нажимаем ctrl+c, возвращаемся на джампноду:
```
exit
```

14. устанавливаем клиент postgres:
```
sudo apt install postgresql-client-16 # версия 16 на 19.01.2025
```

15. проверяем подключение к бд:
```
psql -h team-74-nn -p 5432 -U hive -W -d metastore
```
(ввести пароль)

16. выходим из консоли:
```
\q
```

17. переключаемся на пользователя `hadoop`:
```
sudo -i -u hadoop
```

18. проверяем версию hadoop:
```
hadoop version
```
если пишет, что hadoop не установлен, устанавливаем согласно инструкции в [первом задании](https://github.com/pmashkovtseva/hse-bigdata/blob/main/task-1.md).

19. скачиваем hive:
```
wget https://dlcdn.apache.org/hive/hive-4.0.1/apache-hive-4.0.1-bin.tar.gz # версия 4.0.1 на 19.01.2025
```

20. распаковываем архив:
```
tar -xvzf apache-hive-4.0.1-bin.tar.gz
```

21. переходим в папку дистрибутива:
```
cd apache-hive-4.0.1-bin
```

22. проверяем наличие драйвера для доступа к postgres:
```
cd lib
ls -l | grep postgres
```

23. если драйвера нет (вывод консоли пустой), скачиваем его:
```
wget https://jdbc.postgresql.org/download/postgresql-42.7.4.jar
```

24. переходим в директорию `conf`:
```
cd ..
cd conf
```

25. создаем новый конфиг `hive-site.xml`:
```
nano hive-site.xml
```
добавляем:
```xml
<configuration>
	<property>
    	<name>hive.server2.authentication</name>
    	<value>NONE</value>
	</property>
	<property>
    	<name>hive.metastore.warehouse.dir</name>
    	<value>/user/hive/warehouse</value>
	</property>
	<property>
    	<name>hive.server2.thrift.port</name>
    	<value>5433</value>
    	<description>TCP port number to listen on, default 10000</description>
	</property>
	<property>
    	<name>javax.jdo.option.ConnectionURL</name>
    	<value>jdbc:postgresql://team-74-nn:5432/metastore</value>
	</property>
	<property>
    	<name>javax.jdo.option.ConnectionDriverName</name>
    	<value>org.postgresql.Driver</value>
	</property>
	<property>
    	<name>javax.jdo.option.ConnectionUserName</name>
    	<value>hive</value>
	</property>
	<property>
    	<name>javax.jdo.option.ConnectionPassword</name>
    	<value>password</value>
	</property>
</configuration>
```

26. открываем конфиг `profile`:
```
nano ~/.profile
```

27. добавляем в конец переменные:
```bash
export HIVE_HOME=/home/hadoop/apache-hive-4.0.1-bin
export HIVE_CONF_DIR=$HIVE_HOME/conf
export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib/*
export PATH=$PATH:$HIVE_HOME/bin
```

28. активируем окружение:
```
source ~/.profile
```

29. проверяем версию hive:
```
hive --version
```

30. создаем директорию warehouse:
```
hadoop fs -mkdir -p /user/hive/warehouse
```

31. предоставляем пользователю hive нужный доступ к папкам `warehouse` и `tmp`:
```
hadoop fs -chmod g+w /tmp
hadoop fs -chmod g+w /user/hive/warehouse
```

32. возвращаемся в папку дистрибутива:
```
cd ..
```

33. инициализируем схему hive:
```
schematool -dbType postgres -initSchema
```

34. запускаем hive:
```
hive --hiveconf hive.server2.enable.doAs=false --hiveconf hive.security.authorization.enabled=false --service hiveserver2 1>>/tmp/hs2.log 2>>/tmp/hs2.log &
```

35. проверяем, что hive запустился:
```
jps # должно появиться RunJar
```

36. подключаемся к hive через beeline:
```
beeline -u jdbc:hive2://team-74-jn:5433
```

37. тестируем работу hive:
```sql
SHOW DATABASES; -- должна быть только default
CREATE DATABASE test;
SHOW DATABASES; -- должны быть default и test
DESCRIBE DATABASE test;
```

38. выходим из beeline:
```
!quit
```

39. заходим в hadoop на namenode:
```
ssh team-74-nn
```

40. (hadoop@team-74-nn) загружаем данные из репозитория:
```
wget https://raw.githubusercontent.com/pmashkovtseva/hse-bigdata/refs/heads/main/team-74-data.csv
```

41. создадим папку для хранения данных:
```
hadoop fs -mkdir /input
```

42. предоставим к ней нужные права:
```
hadoop fs -chmod g+w /input
```

43. положим данные в созданную папку:
```
hadoop fs -put team-74-data.csv /input
```

44. посмотрим на блоки загруженного файла:
```
hadoop fsck /input/team-74-data.csv
```

45. подключаемся к beeline:
```
beeline -u jdbc:hive2://team-74-jn:5433
```

46. создадим таблицу и передадим в нее данные из датасета:
```sql
use test;
CREATE TABLE IF NOT EXISTS test.dataset (
	stressed INTEGER,
	pos INTEGER,
	file STRING,
	vowel STRING,
	nr_syll INTEGER,
	v_start DOUBLE,
	v_end DOUBLE,
	nucl_dur DOUBLE,
	syll_dur DOUBLE,
	rms DOUBLE,
	int_peak DOUBLE,
	trajectory DOUBLE,
	f0_max DOUBLE,
	f0_mean DOUBLE
)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
	TBLPROPERTIES ('skip.header.line.count'='1');

LOAD DATA INPATH '/input/team-74-data.csv' INTO TABLE test.dataset;
```

47. создадим партиционированную таблицу и загрузим в нее данные из непартиционированной:
```sql
CREATE TABLE IF NOT EXISTS test.dataset_part (
	stressed INTEGER,
	file STRING,
	vowel STRING,
	nr_syll INTEGER,
	v_start DOUBLE,
	v_end DOUBLE,
	nucl_dur DOUBLE,
	syll_dur DOUBLE,
	rms DOUBLE,
	int_peak DOUBLE,
	trajectory DOUBLE,
	f0_max DOUBLE,
	f0_mean DOUBLE
)
	PARTITIONED BY (pos INTEGER)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

INSERT INTO TABLE test.dataset_part PARTITION (pos) SELECT stressed, pos, file, vowel, nr_syll, v_start, v_end, nucl_dur, syll_dur, rms, int_peak, trajectory, f0_max, f0_mean from test.dataset;
```

48. посмотрим на записи:
```
SELECT * FROM test.dataset_part LIMIT 10;
```

49. посмотрим на партиции:
```
SHOW PARTITIONS dataset_part;
```

50. выйдем из beeline:
```
!quit
```