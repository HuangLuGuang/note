## PLV8入门教程

PLV8是PostgreSQL 的*可信* Javascript语言扩展。它可以用于*存储过程*，*触发器*等。

PLV8适用于大多数版本的Postgres，但最适用于`9.1`和以上，包括`10.0`和`11`。

### PLV8安装

使用docker安装plv8

```bash
# 拉取镜像
docker pull clkao/postgres-plv8
# 创建运行容器，并映射端口
docker run -d --name plv8 -p 5432:5432 clkao/postgres-plv8
# 进入容器
docker exec -it plv8 bash
# 切换用户
psql -U postgres
```

```sql
# 安装plv8扩展
CREATE EXTENSION plv8;
```



 