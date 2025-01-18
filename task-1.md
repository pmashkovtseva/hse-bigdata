# Домашнее задание 1 (настройка Hadoop)

1. подключаемся к jump node по ssh (`ip` заменить на соответствующий ip):
```
ssh team@ip
```

2. (team@team-74-jn) создаем пользователя `hadoop`:
```
sudo adduser hadoop
```
вводим пароль для пользователя `team`

задаем и повторяем пароль для пользователя `hadoop`

опционально заполняем все остальные поля

3. заходим в пользователя `hadoop`:
```
sudo -i -u hadoop
```

4. (hadoop@team-74-jn) генерируем ssh-ключ:
```
ssh-keygen
```
опционально задаем папку и passphrase (дефолтно - папка `.ssh/id_ed25519.pub` и no passphrase)

5. копируем сгенерированный ключ и сохраняем куда-нибудь себе:
```
cat .ssh/id_ed25519.pub # либо папка, указанная на предыдущем шаге
```

6. выходим из пользователя `hadoop`:
```
exit
```

7. (team@team-74-jn) отключаем локальную адресацию:
```
sudo nano /etc/hosts
```
комментируем все адреса (через #) и вставляем адреса узлов (`jn_ip` и др. заменить на соответствующие ip):
```
jn_ip team-74-jn
nn_ip team-74-nn
dn0_ip team-74-dn-0
dn1_ip team-74-dn-1
```
сохраняем и выходим (write out - enter - exit)

8. проделываем шаги 2-7 для name node:
```
ssh team-74-nn
sudo adduser hadoop # вводим пароль
sudo -i -u hadoop
ssh-keygen # опционально заполнить папку и passphrase
cat .ssh/id_ed25519.pub # либо папка, указанная на предыдущем шаге; сохраняем себе ключ
exit
sudo nano /etc/hosts
# комментируем все через # и вставляем адреса узлов:
# jn_ip team-74-jn
# nn_ip team-74-nn
# dn0_ip team-74-dn-0
# dn1_ip team-74-dn-1
# сохраняем и выходим (write out - enter - exit)
```

9. проделываем шаги 2-7 для date node 0:
```
sh team-74-dn0
sudo adduser hadoop # вводим пароль
sudo -i -u hadoop
ssh-keygen # опционально заполнить папку и passphrase
cat .ssh/id_ed25519.pub # либо папка, указанная на предыдущем шаге; сохраняем себе ключ
exit
sudo nano /etc/hosts
# комментируем все через # и вставляем адреса узлов:
# jn_ip team-74-jn
# nn_ip team-74-nn
# dn0_ip team-74-dn-0
# dn1_ip team-74-dn-1
# сохраняем и выходим (write out - enter - exit)
```

10. проделываем шаги 2-7 для date node 1:
```
sh team-74-dn1
sudo adduser hadoop # вводим пароль
sudo -i -u hadoop
ssh-keygen # опционально заполнить папку и passphrase
cat .ssh/id_ed25519.pub # либо папка, указанная на предыдущем шаге; сохраняем себе ключ
exit
sudo nano /etc/hosts
# комментируем все через # и вставляем адреса узлов:
# jn_ip team-74-jn
# nn_ip team-74-nn
# dn0_ip team-74-dn-0
# dn1_ip team-74-dn-1
# сохраняем и выходим (write out - enter - exit)
```

11. возвращаемся на jump node:
```
ssh team-74-jn # либо вводить exit, пока не вернемся
```

12. (team@team-74-jn) заходим на пользователя `hadoop`:
```
sudo -i -u hadoop
```

13. (hadoop@team-74-jn) добавляем сгенерированные на шагах 5, 8-10 ключи в авторизованные:
```
nano .ssh/authorized_keys # write out - enter - exit
```

14. передаем авторизованные ключи на каждую ноду:
```
scp .ssh/authorized_keys team-74-nn:/home/hadoop/.ssh/
scp .ssh/authorized_keys team-74-dn-0:/home/hadoop/.ssh/
scp .ssh/authorized_keys team-74-dn-1:/home/hadoop/.ssh/
```

15. скачиваем дистрибутив hadoop (3.4.1 на 15.01.2025):
```
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz
```

16. передаем дистрибутив на все ноды:
```
scp hadoop-3.4.1.tar.gz team-74-nn:/home/hadoop/
scp hadoop-3.4.1.tar.gz team-74-dn-0:/home/hadoop/
scp hadoop-3.4.1.tar.gz team-74-dn-1:/home/hadoop/
```

17. распаковываем архив с hadoop на name node:
```
ssh team-74-nn
tar -xvzf hadoop-3.4.1.tar.gz
exit
```

18. распаковываем архив с hadoop на data node 0:
```
ssh team-74-dn0
tar -xvzf hadoop-3.4.1.tar.gz
exit
```

19. распаковываем архив с hadoop на data node 1:
```
ssh team-74-dn1
tar -xvzf hadoop-3.4.1.tar.gz
exit
```

20. возвращаемся на name node:
```
ssh team-74-nn
```

21. (team@team-74-nn) переходим на пользователя `hadoop`:
```
sudo -i -u hadoop
```

22. проверяем версию java (должна быть 11):
```
java -version
```

23. смотрим, по какому пути находится java
```
which java
```
и копируем куда-нибудь себе

24. выводим в терминале путь
```
readlink -f /usr/bin/java # либо другой, полученный на предыдущем шаге
```
и копируем куда-нибудь себе

25. открываем конфиг
```
nano ~/.profile
```
и в конец вставляем строки для переменных:
```
export HADOOP_HOME=/home/hadoop/hadoop-3.4.1
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 # либо другой, полученный на предыдущем шаге
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
сохраняем и выходим

26. копируем конфиг на датаноды:
```
scp ~/.profile team-74-dn-0:/home/hadoop/
scp ~/.profile team-74-dn-1:/home/hadoop/
```

27. переходим в папку дистрибутива:
```
cd hadoop-3.4.1/etc/hadoop
```
открываем редактором файл:
```
nano hadoop-env.sh
```
в конце вставляем переменную:
```
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64 # либо другой, полученный на шаге 24
```
закрываем и выходим (write out - enter - exit)

28. копируем конфиг на датаноды:
```
scp hadoop-env.sh team-74-dn-0:/home/hadoop/hadoop-3.4.1/etc/hadoop/
scp hadoop-env.sh team-74-dn-1:/home/hadoop/hadoop-3.4.1/etc/hadoop/
```

29. открываем `core-site.xml`:
```
nano core-site.xml
```

30. редактируем файл, содержимое (после комментариев) должно выглядеть так:
```xml
<configuration>
	<property>
    	<name>fs.defaultFS</name>
    	<value>hdfs://team-74-nn:9000</value>
	</property>
</configuration>
```
сохраняем и выходим (write out - enter - exit)

31. открываем `hdfs-site.xml`:
```
nano hdfs-site.xml
```

32. редактируем файл, содержимое должно выглядеть так:
```xml
<configuration>
	<property>
    	<name>dfs.replication</name>
    	<value>3</value>
	</property>
</configuration>
```

33. открываем файл `workers`:
```
nano workers
```

удаляем строку `localhost`, вставляем следующие строки:
```
team-74-nn
team-74-dn-0
team-74-dn-1
```
сохраняем и выходим (write out - enter - exit)

34. копируем файл `core-site.xml` на датаноды:
```
scp core-site.xml team-74-dn-0:/home/hadoop/hadoop-3.4.1/etc/hadoop
scp core-site.xml team-74-dn-1:/home/hadoop/hadoop-3.4.1/etc/hadoop
```

35. копируем файл `hdfs-site.xml` на датаноды:
```
scp hdfs-site.xml team-74-dn-0:/home/hadoop/hadoop-3.4.1/etc/hadoop
scp hdfs-site.xml team-74-dn-1:/home/hadoop/hadoop-3.4.1/etc/hadoop
```

36. копируем файл `workers` на датаноды:
```
scp workers team-74-dn-0:/home/hadoop/hadoop-3.4.1/etc/hadoop
scp workers team-74-dn-1:/home/hadoop/hadoop-3.4.1/etc/hadoop
```

37. возвращаемся в папку дистрибутива:
```
cd ~/hadoop-3.4.1
```

38. форматируем файловую систему:
```
bin/hdfs namenode -format
```

39. запускаем dfs:
```
sbin/start-dfs.sh
```

40. запускаем jps:
```
jps
```
проверяем, что есть SecondaryNameNode, NameNode и DataNode

41. возвращаемся на jump node:
```
ssh team-74-jn # либо вводить exit до возвращения
```

42. (team@team-74-jn) копируем конфиг nginx:
```
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/nn
```

43. открываем конфиг:
```
sudo nano /etc/nginx/sites-available/nn
```
меняем внутренности и сохраняем. итоговый файл должен выглядеть так:
```
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# https://www.nginx.com/resources/wiki/start/
# https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/
# https://wiki.debian.org/Nginx/DirectoryStructure
#
# In most cases, administrators will remove this file from sites-enabled/ and
# leave it as reference inside of sites-available where it will continue to be
# updated by the nginx packaging team.
#
# This file will automatically load configuration files provided by other
# applications, such as Drupal or Wordpress. These applications will be made
# available underneath a path with that package name, such as /drupal8.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
    	listen 9870 default_server;
    	# listen [::]:80 default_server;

    	# SSL configuration
    	#
    	# listen 443 ssl default_server;
    	# listen [::]:443 ssl default_server;
    	#
    	# Note: You should disable gzip for SSL traffic.
    	# See: https://bugs.debian.org/773332
    	#
    	# Read up on ssl_ciphers to ensure a secure configuration.
    	# See: https://bugs.debian.org/765782
    	#
    	# Self signed certs generated by the ssl-cert package
    	# Don't use them in a production server!
    	#
    	# include snippets/snakeoil.conf;

    	root /var/www/html;

    	# Add index.php to the list if you are using PHP
    	index index.html index.htm index.nginx-debian.html;

    	server_name _;

    	location / {
            	# First attempt to serve request as file, then
            	# as directory, then fall back to displaying a 404.
            	# try_files $uri $uri/ =404;
            	proxy_pass http://team-74-nn:9870;
    	}

    	# pass PHP scripts to FastCGI server
    	#
    	#location ~ \.php$ {
    	#   	include snippets/fastcgi-php.conf;
    	#
    	#   	# With php-fpm (or other unix sockets):
    	#   	fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    	#   	# With php-cgi (or other tcp sockets):
    	#   	fastcgi_pass 127.0.0.1:9000;
    	#}

    	# deny access to .htaccess files, if Apache's document root
    	# concurs with nginx's one
    	#
    	#location ~ /\.ht {
    	#   	deny all;
    	#}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#   	listen 80;
#   	listen [::]:80;
#
#   	server_name example.com;
#
#   	root /var/www/example.com;
#   	index index.html;
#
#   	location / {
#           	try_files $uri $uri/ =404;
#   	}
#}
```
сохраняем и выходим (write out - enter - exit)

44. делаем симлинк:
```
sudo ln -s /etc/nginx/sites-available/nn /etc/nginx/sites-enabled/nn
```

45. перезапускаем nginx:
```
sudo systemctl reload nginx
```

46. подключаемся к серверу:
```
ssh -L 9870:team-74-nn:9870 team@ip
```

47. переходим в браузере по `127.0.0.1:9870` и смотрим на веб-интерфейс.
