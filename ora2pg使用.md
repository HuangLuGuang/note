### oracle表结构迁移到postgresSQL

### ora2pg使用

#### 安装

```shell
docker pull georgmoser/ora2pg
```

#### 用法：

容器接受2个已安装的文件夹

- “/ config”（只读） - >在这里挂载包含“ora2pg.conf”文件的文件夹（示例配置：[ora2pg.conf](https://raw.githubusercontent.com/Guy-Incognito/ora2pg/master/config/ora2pg.conf)
- “/ data” - >挂载应在此处写入所有输出的文件夹

运行容器：

```shell
docker run  \
    --name ora2pg \
    -it \
    -v /path/to/local/config:/config \
    -v /path/to/local/data:/data \
    georgmoser/ora2pg
```

如果没有传递参数，则默认运行将是：

```shell
ora2pg --debug -c /config/ora2pg.conf
```

可以使用以下方法将自定义参数传递给ora2pg调用：

```shell
docker run  \
    --name ora2pg \
    -it \
    -v /path/to/local/config:/config \
    -v /path/to/local/data:/data \
    georgmoser/ora2pg \
    ora2pg [[ARG1..ARGN]]
```

而且

- CONFIG_LOCATION：ora2pg配置文件位置（在容器内）
- OUTPUT_LOCATION：转储的输出目录（在容器内）

可以通过环境变量传递：

```shell
docker run  \
    --name ora2pg \
    -e CONFIG_LOCATION=/config/myconfigfile.conf  \
    -e OUTPUT_LOCATION=/data/myfolder  \
    -it \
    -v /path/to/local/config:/config \
    -v /path/to/local/data:/data \
    georgmoser/ora2pg 
```

##### ora2pg.conf样例

```
ORACLE_DSN  dbi:Oracle:host=192.168.172.176;sid=orcl;port=1521
ORACLE_USER flxuser
ORACLE_PWD  flxuser
SCHEMA      flxuser
TYPE    TABLE
PG_DSN          dbi:Pg:dbname=test_tb;host=192.168.171.66;port=5432
PG_USER         postgres
PG_PWD          postgres
OUTPUT          huanglg.sql
OUTPUT_DIR  /data
```

##### 使用生成的sql创建数据库表

```shell
su postgresql
psql
\c test_tb
\i /path/to/local/data/huanglg.sql
```

#### 遇到的问题

##### 1.uuid_generate_v4()不存在

###### 解决方法：

```shell
# 安装
sudo yum install postgresql11-contrib.x86_64
# 在要创建的数据库下安装扩展
CREATE EXTENSION IF NOT EXISTS "uuid-ossp"
```

##### 2.字段名binary为pg关键字，改为binary_

##### 3.rawtohex函数不存在，替换掉即可



