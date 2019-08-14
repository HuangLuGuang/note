## PLV8入门教程

PLV8是PostgreSQL 的Javascript语言扩展。它可以用于*存储过程*，*触发器*等。

PLV8适用于大多数版本的Postgres，但最适用于`9.1`和以上。

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
--安装plv8扩展
CREATE EXTENSION plv8;
--查看安装情况
SELECT extversion FROM pg_extension WHERE extname = 'plv8';
--结果如下
```

![1565702652595](C:\Users\Huanglg\AppData\Roaming\Typora\typora-user-images\1565702652595.png)



#### 使用

##### Scalar Function Calls

```sql
--创建一个Scalar函数
CREATE FUNCTION plv8_test(keys TEXT[], vals TEXT[]) RETURNS JSON AS 
$$
    var o = {};
    for(var i=0; i<keys.length; i++){
        o[keys[i]] = vals[i];
    }
    return o;
$$ LANGUAGE plv8 IMMUTABLE STRICT;
```

```sql
--调用Scalar函数
SELECT plv8_test(ARRAY['name', 'age'], ARRAY['Tom', '29']);
```

##### PLV8 可以返回set类型

```sql
--1.创建set类型
CREATE TYPE rec AS (i integer, t text);
--2.创建返回set类型的函数
CREATE FUNCTION set_of_records() RETURNS SETOF rec AS
$$
    plv8.return_next( { "i": 1, "t": "a" } );
    plv8.return_next( { "i": 2, "t": "b" } );
    return [ { "i": 3, "t": "c" }, { "i": 4, "t": "d" } ];
$$
LANGUAGE plv8;
--3.调用该函数
SELECT * FROM set_of_records();
--4.结果如下：
i | t
---+---
1 | a
2 | b
3 | c
4 | d
(4 rows)
```

> 如果函数的返回类型为SETOF，tuplestore每次调用都会准备一次，可以根据需要调用plv8.return_next()返回一行，还可以返回array返回一组记录。

##### PLV8创建触发器函数

```sql
CREATE FUNCTION test_trigger() RETURNS TRIGGER AS
$$
    plv8.elog(NOTICE, "NEW = ", JSON.stringify(NEW));
    plv8.elog(NOTICE, "OLD = ", JSON.stringify(OLD));
    plv8.elog(NOTICE, "TG_OP = ", TG_OP);
    plv8.elog(NOTICE, "TG_ARGV = ", TG_ARGV);
    if (TG_OP == "UPDATE") {
        NEW.i = 102;
        return NEW;
    }
$$
LANGUAGE "plv8";

-- 创建触发器
CREATE TRIGGER test_trigger
    BEFORE INSERT OR UPDATE OR DELETE
    ON test_tb FOR EACH ROW
    EXECUTE PROCEDURE test_trigger('foo', 'bar');

-- 插入一条数据观察结果
INSERT INTO public.test_tb(
	id, name)
	VALUES (1, 'hlg');
	
-- 结果如下
NOTICE:  NEW =  {"id":"1         ","name":"hlg                 "}
NOTICE:  OLD =  undefined
NOTICE:  TG_OP =  INSERT
NOTICE:  TG_ARGV =  foo,bar
INSERT 0 1
```

如果触发器类型是`INSERT`或`UPDATE`，则可以指定`NEW`变量的属性 以更改此操作存储的实际元组。PLV8触发器函数将具有以下包含触发器状态的特殊参数：

- `NEW`
- `OLD`
- `TG_NAME`
- `TG_WHEN`
- `TG_LEVEL`
- `TG_OP`
- `TG_RELID`
- `TG_TABLE_NAME`
- `TG_TABLE_SCHEMA`
- `TG_ARGV`

> 有关更多信息，请参阅[PostgreSQL手册中](https://www.postgresql.org/docs/current/static/plpgsql-trigger.html)的[触发器部分](https://www.postgresql.org/docs/current/static/plpgsql-trigger.html)。

##### 类型数组

在PLV8中数组类型和JavaScript中的对应，比如：

- `plv8_int2array` ->`int2[]`
- `plv8_int4array  `->`int4[]`
- `plv8_float4array` ->`float4[]`
- `plv8_float8array` ->`float8[]`

```sql
--求和函数
CREATE FUNCTION int4sum(ary plv8_int4array) RETURNS int8 AS
$$
	var sum=0;
	for (var i=0; i < ary.length; i++) {
		sum += ary[i];
	}
	return sum;
$$ LANGUAGE plv8 IMMUTABLE STRICT;

--使用该函数
select int4sum(ARRAY[1, 2, 3, 4, 5]);
```

`plv8.elog`可以向客户端或日志文件打印日志，日志级别如下：

- `DEBUG5`

- `DEBUG4`

- `DEBUG3`

- `DEBUG2`

- `DEBUG1`

- `LOG`

- `INFO`

- `NOTICE`

- `WARNING`

- `ERROR`

  例如:

  ```sql
  DO 
  $$
  	var msg = 'world';
  	plv8.elog(NOTICE, 'Hello', `${msg}!`);
  $$ language plv8;
  ```

##### 访问数据库

```
plv8.execute(sql [, args])
```

执行SQL语句并检索结果。该`sql`参数是必需的，而`args`参数是一个可选`array`含在SQL查询传递任何参数。对于`SELECT`查询，返回值是`array` 的`objects`。每个`object`代表一行，`object`属性映射为列名。对于非`SELECT`查询，返回结果是受影响的行数。

例如:

```sql
-- 查询
DO 
$$  
	var json_result = plv8.execute('SELECT * FROM test_tb');
	plv8.elog(NOTICE, JSON.stringify(json_result), 'inline', 'code');
$$ LANGUAGE plv8;
-- 更新
DO 
$$  
	var json_result = plv8.execute('SELECT * FROM test_tb');
	plv8.elog(NOTICE, JSON.stringify(json_result), 'inline', 'code');
$$ LANGUAGE plv8;
```

### `plv8.prepare`

```
plv8.prepare(sql [, typenames])
```

打开或创建准备好的声明。该`typename`参数是`array` 每个元素`string`对应于每个`bind`参数的PostgreSQL类型名称的参数。返回值是该`PreparedPlan`类型的对象。`plan.free()`在离开函数之前必须释放此对象。

例如

```sql
DO
$$
	var plan = plv8.prepare('SELECT * FROM test_tb WHERE id = $1', [ 'character' ]);
	var rows = plan.execute([ '3' ]);
	for (var i = 0; i < rows.length; i++) {
	  plv8.elog(NOTICE, JSON.stringify(rows[i]), i);
	}
	// 释放
	plan.free();
$$ language plv8;
```

### `PreparedPlan.cursor`

```
PreparedPlan.cursor([ args ])
```

从准备好的语句中打开游标。该`args`参数是一样的东西将需要`plv8.execute()`和`PreparedPlan.execute()`。返回的对象是类型`Cursor`。必须`Cursor.close()` 在离开功能之前将其关闭。

```sql
DO
$$
	var plan = plv8.prepare('SELECT * FROM test_tb WHERE id = $1', [ 'character' ]);
	var cursor = plan.cursor([ '2' ]);
	var row;
	while(row = cursor.fetch()) {
			plv8.elog(NOTICE, JSON.stringify(row));  
	}
	// 释放
  	cursor.close();
	plan.free();
$$ language plv8;
```

### `plv8.subtransaction`

```
plv8.subtransaction(func)
```

`plv8.execute()`每次执行时都会创建一个子事务。如果需要原子操作，则需要调用`plv8.subtransaction()`以创建子事务块。

```
DO
$$
try {
		 plv8.subtransaction(function(){
			 plv8.execute(`insert into test_tb (id, name) values('4','hhxy1')`);
			 plv8.execute(`insert into test_tb (id, name) values(1/0,'hhxy1')`);
		});
	 }catch(e){
			plv8.elog(ERROR, e, 'rollback')
			}
$$ language plv8;
--或者
DO
$$
try {
	 plv8.execute(`insert into test_tb (id, name) values('4','hhxy1')`);
	 plv8.execute(`insert into test_tb (id, name) values(1/0,'hhxy1');`);
	 }catch(e){
			plv8.elog(ERROR, e, 'rollback')
			}
$$ language plv8;
```

