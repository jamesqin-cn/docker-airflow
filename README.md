## 改造airflow
### 增加对MySQL的支持
> airflow镜像没有把mysql的包添加进去，需要改造一下，改动点如下：

- 操作系统库支持
```
apt-get install libmariadbd-dev
apt-get install libmysqlclient-dev
```

- python库支持
```
pip install apache-airflow[crypto,celery,postgres,`mysql`,hive,jdbc]==$AIRFLOW_VERSION
```

- 工具支持
增加两个常用工具，方便以后写bash脚本操作数据库和文件传输
```bash
apt-get install mysql-client
apt-get install rsync
```

- 修改配置文件的连接变量
找到airflow.cfg中的core区域，修改sql\_alchemy\_conn数据库连接变量
```ini
[core]
sql_alchemy_conn = mysql://MYSQL_USER:MYSQL_PASS@MYSQL_HOST/DBNAME
```

- 传入环境变量拉起scheduler进程
在默认启动模式不会拉起scheduler，一个完整的airflow系统应拉起webserver和scheduler两个进程
docker启动airflow容器传入EXECUTOR环境变量，以确保下面的两行能执行
```bash
$CMD initdb
exec $CMD webserver &
exec $CMD scheduler
```

- 创建MySQL数据库
需要提前创建好数据库airflow，注意，字符编码需要是 latin1。我一开始尝试用utf8，发现在airflow initdb的环节会报错
```bash
sqlalchemy.exc.OperationalError: (_mysql_exceptions.OperationalError) (1071, 'Specified key was too long; max key length is 1000 bytes') [SQL: '\nCREATE TABLE sla_miss (\n\ttask_id VARCHAR(250) NOT NULL, \n\tdag_id VARCHAR(250) NOT NULL, \n\texecution_date DATETIME NOT NULL, \n\temail_sent BOOL, \n\ttimestamp DATETIME, \n\tdescription TEXT, \n\tPRIMARY KEY (task_id, dag_id, execution_date), \n\tCHECK (email_sent IN (0, 1))\n)\n\n']
```
