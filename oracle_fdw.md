1.安装扩展

```sql
create extension oracle_fdw;
```

2.创建oracle连接

```sql
-- oracle是连接名
create server oracle foreign data wrapper oracle_fdw options(dbserver '192.168.172.183:1521/orcl');	
```

3.给用户权限

```sql
-- 把创建的连接给postgres用户
grant usage on foreign server oracle to postgres;
```

4.创建用户映射

```sql
create user mapping for postgres server oracle options(user 'flxuser',password 'flxuser');
```

5.创建外部表

```sql

create foreign table employee
(
	id SERIAL options(key 'true') NOT NULL, --必须要设置主键，否则无法update,delete
	name varchar(100),
	givenname varchar(50),
	middlename varchar(50),
	familyname varchar(50),
	addressid bigint,
	contactid bigint,
	employeeno varchar(20) NOT NULL,
	loginname varchar(20) NOT NULL,
	password varchar(256),
	hiredate timestamp,
	terminationdate timestamp,
	department varchar(40),
	spokenlanguageid bigint DEFAULT 1033,
	writtenlanguageid bigint DEFAULT 1033,
	lastsignondate timestamp DEFAULT LOCALTIMESTAMP,
	lastsignoffdate timestamp DEFAULT LOCALTIMESTAMP,
	lastfacility varchar(40),
	employeevaliddate timestamp NOT NULL DEFAULT LOCALTIMESTAMP,
	loginexpirationdate timestamp DEFAULT LOCALTIMESTAMP,
	loginlockoutdate timestamp,
	passwordlastchangedate timestamp DEFAULT LOCALTIMESTAMP,
	unionname varchar(70),
	laborjurisdictionname varchar(70),
	immigrationstatus varchar(70),
	employeeclass bigint,
	employeetype bigint,
	defaultfacility varchar(40),
	defaultteam varchar(40),
	defaultroleid bigint,
	earningscurrencycode varchar(3),
	costcenterdivision varchar(70),
	costcenter varchar(70),
	defaultmenuitemid bigint,
	trackattendanceflag smallint,
	tracklaborflag smallint,
	trackresourcelaborflag smallint,
	externallogin varchar(50),
	resourceid bigint NOT NULL,
	maximumequipmentlogins bigint,
	title varchar(80),
	pictureurl varchar(2000),
	currentcountry varchar(3),
	citizenshipcountry varchar(3),
	logincode varchar(1024),
	forcepasswordchange smallint,
	referenceid bigint,
	lastupdateon timestamp,
	lastupdatedby varchar(50),
	createdon timestamp,
	createdby varchar(50),
	active smallint NOT NULL DEFAULT 1,
	lastdeleteon timestamp,
	lastdeletedby varchar(50),
	lastreactivateon timestamp,
	lastreactivatedby varchar(50),
	archiveid bigint,
	lastarchiveon timestamp,
	lastarchivedby varchar(50),
	lastrestoreon timestamp,
	lastrestoredby varchar(50),
	rowversionstamp numeric(38) DEFAULT 1,
	dashboardbusinessobjectid bigint,
	countryofbirth varchar(3),
	dateofbirth timestamp
) server oracle  options(schema 'FLXUSER',table 'EMPLOYEE'); --主要表名必须大写
```

##### oracle和postgres数据类型对比

```shell
Oracle type              | Possible PostgreSQL types
-------------------------+--------------------------------------------------
CHAR                     | char, varchar, text
NCHAR                    | char, varchar, text
VARCHAR                  | char, varchar, text
VARCHAR2                 | char, varchar, text, json
NVARCHAR2                | char, varchar, text
CLOB                     | char, varchar, text, json
LONG                     | char, varchar, text
RAW                      | uuid, bytea
BLOB                     | bytea
BFILE                    | bytea (read-only)
LONG RAW                 | bytea
NUMBER                   | numeric, float4, float8, char, varchar, text
NUMBER(n,m) with m<=0    | numeric, float4, float8, int2, int4, int8,
                         |    boolean, char, varchar, text
FLOAT                    | numeric, float4, float8, char, varchar, text
BINARY_FLOAT             | numeric, float4, float8, char, varchar, text
BINARY_DOUBLE            | numeric, float4, float8, char, varchar, text
DATE                     | date, timestamp, timestamptz, char, varchar, text
TIMESTAMP                | date, timestamp, timestamptz, char, varchar, text
TIMESTAMP WITH TIME ZONE | date, timestamp, timestamptz, char, varchar, text
TIMESTAMP WITH           | date, timestamp, timestamptz, char, varchar, text
   LOCAL TIME ZONE       |
INTERVAL YEAR TO MONTH   | interval, char, varchar, text
INTERVAL DAY TO SECOND   | interval, char, varchar, text
XMLTYPE                  | xml, char, varchar, text
MDSYS.SDO_GEOMETRY       | geometry (see "PostGIS support" below)
```