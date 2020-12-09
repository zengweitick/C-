#  C++操作数据库

## 使用流程

```c++
#include <stdio.h>  
#include <WinSock.h>  //一定要包含这个 
#include "include/mysql.h"    //引入mysql头文件(一种方式是在vc++目录里面设置，一种是文件夹拷到工程目录，然后这样包含)  
#include <Windows.h>  
 
//包含附加依赖项，也可以在工程--属性里面设置  
#pragma comment(lib,"wsock32.lib") 
#pragma comment(lib,"libmysql.lib")
 
MYSQL mysql; //mysql连接
MYSQL_FIELD *fd;  //字段列数组
char field[32][32];  //存字段名二维数组
MYSQL_RES *res; //这个结构代表返回行的一个查询结果集
MYSQL_ROW column; //一个行数据的类型安全(type-safe)的表示，表示数据行的列
char query[150]; //查询语句
 
bool ConnectDatabase();    	//函数声明  
void FreeConnect();
bool QueryDatabase1();  	//查询1  
bool QueryDatabase2();  	//查询2  
bool InsertData();		//增
bool ModifyData();		//改
bool DeleteData();		//删
int main(int argc, char **argv)
{
	ConnectDatabase();
	QueryDatabase1();
	InsertData();
	QueryDatabase2();
	ModifyData();
	QueryDatabase2();
	DeleteData();
	QueryDatabase2();
	FreeConnect();
	return 0;
}
//连接数据库  
bool ConnectDatabase()
{
	//初始化mysql  
	mysql_init(&mysql);  //连接mysql，数据库  
	const char host[] = "localhost";
	const char user[] = "root";
	const char psw[] = "111111";
	const char table[] = "test";
	const int port = 3306;
	//返回false则连接失败，返回true则连接成功  
	if (!(mysql_real_connect(&mysql, host, user, psw, table, port, NULL, 0)) )
		//中间分别是主机，用户名，密码，数据库名，端口号（可以写默认0或者3306等），可以先写成参数再传进去  
	{
		printf("Error connecting to database:%s\n", mysql_error(&mysql));
		return false;
	}
	else
	{
		printf("Connected...\n");
		return true;
	}
}
//释放资源  
void FreeConnect()
{
	mysql_free_result(res);  //释放一个结果集合使用的内存。
	mysql_close(&mysql);	 //关闭一个服务器连接。
}
 
/***************************数据库操作***********************************/
//其实所有的数据库操作都是先写个sql语句，然后用mysql_query(&mysql,query)来完成，包括创建数据库或表，增删改查  
//查询数据  
bool QueryDatabase1()
{
	strcpy(query, "select * from user"); //执行查询语句，这里是查询所有，user是表名，不用加引号，用strcpy也可以  
	mysql_query(&mysql, "set names gbk"); //设置编码格式（SET NAMES GBK也行），否则cmd下中文乱码  
	//返回0 查询成功，返回1查询失败  
	if (mysql_query(&mysql, query)){        // 执行指定为一个空结尾的字符串的SQL查询。	
		printf("Query failed (%s)\n", mysql_error(&mysql));
		return false;
	}
	else{
		printf("query success\n");
	}
	//获取结果集  
	if (!(res = mysql_store_result(&mysql)))    //获得sql语句结束后返回的结果集  
	{
		printf("Couldn't get result from %s\n", mysql_error(&mysql));
		return false;
	}
 
	//打印数据行数  
	printf("number of dataline returned: %d\n", mysql_affected_rows(&mysql));
 
	//获取字段的信息  
	char *str_field[32];  //定义一个字符串数组存储字段信息
	for (int i = 0; i < 4; i++)   //在已知字段数量的情况下获取字段名 
	{
		str_field[i] = mysql_fetch_field(res)->name;	//返回一个所有字段结构的数组。
	}
	for (int i = 0; i < 4; i++)   //打印字段  
		printf("%10s\t", str_field[i]);
	printf("\n");
	//打印获取的数据  
	while (column = mysql_fetch_row(res))   //在已知字段数量情况下，获取并打印下一行  
	{
		printf("%10s\t%10s\t%10s\t%10s\n", column[0], column[1], column[2], column[3]);  //column是列数组  
	}
	return true;
}
bool QueryDatabase2()
{
	mysql_query(&mysql, "set names gbk");
	//返回0 查询成功，返回1查询失败  
	if (mysql_query(&mysql, "select * from user"))        //执行SQL语句  
	{
		printf("Query failed (%s)\n", mysql_error(&mysql));
		return false;
	}
	else
	{
		printf("query success\n");
	}
	res = mysql_store_result(&mysql);
	//打印数据行数  
	printf("number of dataline returned: %d\n", mysql_affected_rows(&mysql));
	for (int i = 0; fd = mysql_fetch_field(res); i++)  //获取字段名  
		strcpy(field[i], fd->name);
	int j = mysql_num_fields(res);  // 获取列数  
	for (int i = 0; i<j; i++)  //打印字段  
		printf("%10s\t", field[i]);
	printf("\n");
	while (column = mysql_fetch_row(res))
	{
		for (int i = 0; i<j; i++)
			printf("%10s\t", column[i]);
		printf("\n");
	}
	return true;
}
 
//插入数据  
bool InsertData()
{
	//可以想办法实现手动在控制台手动输入指令  
	strcpy(query, "insert into user values (NULL, 'Lilei', 'wyt2588zs','lilei23@sina.cn');");  
	
	if (mysql_query(&mysql, query))        //执行SQL语句  
	{
		printf("Query failed (%s)\n", mysql_error(&mysql));
		return false;
	}
	else
	{
		printf("Insert success\n");
		return true;
	}
}
 
//修改数据  
bool ModifyData()
{
	strcpy(query, "update user set email='lilei325@163.com' where name='Lilei'");
	if (mysql_query(&mysql, query))        //执行SQL语句  
	{
		printf("Query failed (%s)\n", mysql_error(&mysql));
		return false;
	}
	else
	{
		printf("Insert success\n");
		return true;
	}
}
//删除数据  
bool DeleteData()
{
	/*sprintf(query, "delete from user where id=6");*/
	char query[100];
	printf("please input the sql:\n");
	gets(query);  //这里手动输入sql语句  
	if (mysql_query(&mysql, query))        //执行SQL语句  
	{
		printf("Query failed (%s)\n", mysql_error(&mysql));
		return false;
	}
	else
	{
		printf("Insert success\n");
		return true;
	}
}
```

## 操作API

 >**mysql_affected_rows()** 返回被最新的UPDATE, DELETE或INSERT查询影响的行数。
 >**mysql_close()** 关闭一个服务器连接。
 >**mysql_connect()** 连接一个MySQL服务器。该函数不推荐；使用mysql_real_connect()代替。
 >**mysql_change_user()** 改变在一个打开的连接上的用户和数据库。
 >**mysql_create_db()** 创建一个数据库。该函数不推荐；而使用SQL命令CREATE DATABASE。
 >**mysql_data_seek()** 在一个查询结果集合中搜寻一任意行。
 >**mysql_debug()** 用给定字符串做一个DBUG_PUSH。
 >**mysql_drop_db()** 抛弃一个数据库。该函数不推荐；而使用SQL命令DROP DATABASE。
 >**mysql_dump_debug_info()** 让服务器将调试信息写入日志文件。
 >**mysql_eof()** 确定是否已经读到一个结果集合的最后一行。这功能被反对; mysql_errno()或**mysql_error()**可以相反被使用。
 >**mysql_errno()** 返回最近被调用的MySQL函数的出错编号。
 >**mysql_error()** 返回最近被调用的MySQL函数的出错消息。
 >**mysql_escape_string()** 用在SQL语句中的字符串的转义特殊字符。
 >**mysql_fetch_field()** 返回下一个表字段的类型。
 >**mysql_fetch_field_direct ()** 返回一个表字段的类型，给出一个字段编号。
 >**mysql_fetch_fields()** 返回一个所有字段结构的数组。
 >**mysql_fetch_lengths()** 返回当前行中所有列的长度。
 >**mysql_fetch_row()** 从结果集合中取得下一行。
 >**mysql_field_seek()** 把列光标放在一个指定的列上。
 >**mysql_field_count()** 返回最近查询的结果列的数量。
 >**mysql_field_tell()** 返回用于最后一个mysql_fetch_field()的字段光标的位置。
 >**mysql_free_result()** 释放一个结果集合使用的内存。
 >**mysql_get_client_info()** 返回客户版本信息。
 >**mysql_get_host_info()** 返回一个描述连接的字符串。
 >**mysql_get_proto_info()** 返回连接使用的协议版本。
 >**mysql_get_server_info()** 返回服务器版本号。
 >**mysql_info()** 返回关于最近执行得查询的信息。
 >**mysql_init()** 获得或初始化一个MYSQL结构。
 >**mysql_insert_id()** 返回有前一个查询为一个AUTO_INCREMENT列生成的ID。
 >**mysql_kill()** 杀死一个给定的线程。
 >**mysql_list_dbs()** 返回匹配一个简单的正则表达式的数据库名。
 >**mysql_list_fields()** 返回匹配一个简单的正则表达式的列名。
 >**mysql_list_processes()** 返回当前服务器线程的一张表。
 >**mysql_list_tables()** 返回匹配一个简单的正则表达式的表名。
 >**mysql_num_fields()** 返回一个结果集合重的列的数量。
 >**mysql_num_rows()** 返回一个结果集合中的行的数量。
 >**mysql_options()** 设置对mysql_connect()的连接选项。
 >**mysql_ping()** 检查对服务器的连接是否正在工作，必要时重新连接。
 >**mysql_query()** 执行指定为一个空结尾的字符串的SQL查询。
 >**mysql_real_connect()** 连接一个MySQL服务器。
 >**mysql_real_query()** 执行指定为带计数的字符串的SQL查询。
 >**mysql_reload()** 告诉服务器重装授权表。
 >**mysql_row_seek()** 搜索在结果集合中的行，使用从mysql_row_tell()返回的值。
 >**mysql_row_tell()** 返回行光标位置。
 >**mysql_select_db()** 连接一个数据库。
 >**mysql_shutdown()** 关掉数据库服务器。
 >**mysql_stat()** 返回作为字符串的服务器状态。
 >**mysql_store_result()** 检索一个完整的结果集合给客户。
 >**mysql_thread_id()** 返回当前线程的ID。
 >**mysql_use_result()** 初始化一个一行一行地结果集合的检索。