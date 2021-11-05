## 连接例程

用C语言连接MySQL数据库包含两个步骤“

- 初始化一个连接句柄结构；
- 实际进行连接。

### 初始化连接句柄

```
#include<mysql.h>
MYSQL *mysql_init(MYSQL *);
```

- 通常传递NULL给这个例程，它会返回一个指向新分配的连接句柄结构的指针。
- 如果传递一个已有的结构，它将被重新初始化。
- 出错返回NULL。

### 连接

```
MYSQL *mysql_real_connect(MYSQL *connection,
		const char *server_host,
		const char *sql_user_name,
		const char *sql_password,
		const char *db_name,
		unsigned int port_number,
		const char *unix_socket_name,
		unsigned int flags);
```

- 指针connection必须指向已经被mysql_init初始化过的结构。
- server_host既可以是主机名，也可以是IP地址。如果是连接到本地，可以通过指定localhost来优化连接类型。
- sql_user_name和sql_password，用户名和密码。
- port_number和unix_socket_name应该分别为0和NULL，除非改变了MySQL安装的默认设置。
- flag参数用来对一些定义的位模式进行OR操作，使得改变使用协议的某些特性。0。
- 如果无法连接，它将返回NULL。

### 关闭连接

```
void mysql_close(MYSQL *connection);
```

### 设置选项

```
int mysql_options(MYSQL *connection ,enum option_to_set,const char *argument);
```

- 一次只能设置一个选项，所以每设置一个选项就得调用它一次。

- 成功返回0。

- 仅能在mysql_init和mysql_real_connect之间调用。

- 并不是所有的选项都是char类型，因此它们必须被转换为const char *。

  | enum选项                  | 实际参数类型         | 说明                     |
  | ------------------------- | -------------------- | ------------------------ |
  | MySQL_OPT_CONNECT_TIMEOUT | const unsigned int * | 连接超时之前的等待秒数   |
  | MySQL_OPT_COMPRESS        | None，使用NULL       | 网络连接中使用压缩机制   |
  | MySQL_OPT_COMMAND         | const char *         | 每次连接建立后发送的命令 |

- 设置连接超时时间为7秒

  ```
  unsigned int timeout = 7;
  ...
  connection = mysql_init(NULL);
  mysql_options(connection,MYSQL_OPT_CONNECT_TIMEOUT,(const char *)&timeout);
  
  connection = mysql_real_connect(connection...
  ```

### 错误处理

```
unsigned int mysql_errno(MYSQL *connection);

char *mysql_error(MYSQL *connection);
```

- mysql_errno的返回值实际上是错误码。
- mysql_error的返回值是文本错误信息。

### 执行SQL语句

```
int mysql_query(MYSQL *connection,const char *query);
```

- 接受连接结构指针二号文本字符串形式的有效SQL语句（没有结束的分号）。

### 受影响的行数(UPDATE,INSERT或DELETE)

```
my_ulonglong mysql_affected_rows(MYSQL *connection);
```

- 返回受之前执行的UPDATE,INSERT或DELETE查询影响的行数。

### 返回数据的语句(SELECT)

#### 一次提取所有数据

```
MYSQL_RES *mysql_store_result(MYSQL *connection);
```

- 在mysql_store_result调用成功之后，调用mysql_num_rows来得到返回记录的数目。

  `my_ulonglong mysql_num_rows(MYSQL_RES *result);`

- 从使用mysql_store_result得到的结果结构中提取一行，并把它放到一个行结构中。

  `MYSQL_ROW mysql_fetch_row(MYSQL_RES *result);`

- 这个函数用来在结果集中进行跳转，设置将会被下一个mysql_fetch_row操作返回的行。参数offset的值是一个行号，0是第一行。

  `void mysql_data_seek(MYSQL_RES *result,my_ulonglong offset);`

- mysql_row_tell返回一个偏移值，用来表示结果集中的当前位置，它不是行号，不能用于mysql_data_seek.

  `MYSQL_ROW_OFFSET mysql_row_tell(MYSQL_RES *result);`

- 完成了对数据的所有操作后，必须明确的调用mysql_free_result来让MySQL库完成善后处理

  `void mysql_free_result(MYSQL_RES *result);`



#### 一次提取一行数据

```
MYSQL_RES *mysql_use_result(MYSQL *connection);
```

### 处理返回的数据

1. `unsigned int mysql_field_count(MYSQL *connection);`

- 它接受连接对象，并返回结果集中的字段（列）数目。

简单的打印数据的代码：

```
void display_row(){
	unsigned int field_count;
	
	field_count = 0;
	while(field_count < mysql_field_count(&my_connection)){
		printf("%s",sqlrow[field_count]);
		field_count++;
	}
	printf("\n");
}
```



2. `MYSQL_FIELD *mysql_fetch_field(MYSQL_RES *result);`

- 将元数据和数据提取到一个新的结构中

| MySQL_FIELD结构中的成员 | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| char *name;             | 列名，字符串                                                 |
| char *table;            | 列所属的表名                                                 |
| unsigned int flags;     | NOT _NULL _FLAG、PRI_ KEY_ FLAG、UNSIGNED FLAG. AUTO_ INCREMENT_FLAC和BINARY_ FLAG |

