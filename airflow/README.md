注：这个文章是基于MacOS环境搭建的，window或linux可能会有不同，请自己google

## 安装
0, 安装python3

1, 先安装virtualenv

2, 安装airflow
``` shell
pip install apache-airflow
```

3, 增加AIRFLOW_HOME环境变量
```shell
export AIRFLOW_HOME="/Users/YOUR_HOME/airflow"
```

4, 增加语言环境变量
``` shell
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
```

5, 安装mysqlclient
5.1, 先安装mysql-connector-c
``` shell
brew install mysql-connector-c
```
如果没有安装brew，参考这个安装https://brew.sh/

5.2, 编辑```/usr/local/bin/mysql_config```
找到以下内容：
``` shell
# Create options

libs="-L$pkglibdir"

libs="$libs -l "
```
改成：
``` shell
# Create options

libs="-L$pkglibdir"

libs="$libs -lmysqlclient -lssl -lcrypto"
```

5.3 增加openssl环境变量
如果没有安装openssl，则先
``` shell
brew install openssl
```

``` shell
export LDFLAGS="-L/usr/local/opt/openssl/lib"
export CPPFLAGS="-I/usr/local/opt/openssl/include"
```

5.4 安装mysqlclient
``` shell
pip install mysqlclient
```

5.5 安装airflow mysql package
``` shell
pip install 'apache-airflow[mysql]'
```

6, 修改excutor为LocalExecutor
修改airflow.cfg
``` shell
# The executor class that airflow should use. Choices include
# SequentialExecutor, LocalExecutor, CeleryExecutor, DaskExecutor, KubernetesExecutor
executor = LocalExecutor
```


## 配置
1, 配置mysql为后台数据库
1.1, 新建一个mysql数据库```airflow```

1.2, 修改airflow.cfg中的sql_alchemy_conn
``` shell
# The SqlAlchemy connection string to the metadata database.
# SqlAlchemy supports many different database engine, more information
# their website
sql_alchemy_conn = mysql://username:password@host:ip/airflow
```

1.3, 初始化数据库
``` shell
airflow initdb
```

## 启动
1, 启动webserver
``` shell
airflow webserver -p 8080
```

2, 启动scheduler
``` shell
airflow scheduler
```
如果是分布式部署，则最好加上-p 参数，这样scheduler会发送pickle对象给worker，即使worker没有相应的dags
``` shell
airflow scheduler -p
```

3, 启动worker
``` shell
airflow worker -p
```

## 配置oracle
1, install cx_oracle
``` shell
pip install 'apache-airflow[oracle]'
```

2, install oracle client
https://www.oracle.com/database/technologies/instant-client/downloads.html
下载1，basic package，SQL Plus package，tools package
3, 解压到~/oracle_client

## 分布式
1, 安装celery
``` shell
pip install 'apache-airflow[celery]'
```

1.1 修改airflow.cfg
``` shell
# The executor class that airflow should use. Choices include
# SequentialExecutor, LocalExecutor, CeleryExecutor, DaskExecutor, KubernetesExecutor
executor = CeleryExecutor
```

2, 配置broker
2.1, 使用mysql
修改airflow.cfg
``` shell
# The Celery broker URL. Celery supports RabbitMQ, Redis and experimentally
# a sqlalchemy database. Refer to the Celery documentation for more
# information.
# http://docs.celeryproject.org/en/latest/userguide/configuration.html#broker-settings
broker_url = sqla+mysql://username:password@host:port/airflow


# The Celery result_backend. When a job finishes, it needs to update the
# metadata of the job. Therefore it will post a message on a message bus,
# or insert it into a database (depending of the backend)
# This status is used by the scheduler to update the state of the task
# The use of a database is highly recommended
# http://docs.celeryproject.org/en/latest/userguide/configuration.html#task-result-backend-settings
result_backend = db+mysql://username:password@host:port/airflow
```

2.2, 使用RabbitMQ

2.3, 使用redis
配置broker_url = redis://IP:PORT/QUEUE_ID
