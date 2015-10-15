title: "学习JDBC:JDBC"
date: 2015-04-29 16:36:30
tags: [JDBC]
categories: SQL
description: 主要介绍了什么是JDBC，以及在JAVA里面如何关联数据库
---
#JDBC简介
应用程序、JDBC API、数据库驱动和数据库之间的关系![JDBC简介](http://askingwindy-gitcafe.qiniudn.com/JDBC简介.png)

driver实现了JDBC的接口，是由数据库开发
##连接数据的步骤
```java
/*
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import com.mysql.jdbc.Driver;
*/

/**
 * 不太规范的示范
 * @throws ClassNotFoundException
 * @throws SQLException
 */
public static void test() throws ClassNotFoundException, SQLException{
	//1.注册驱动
	DriverManager.registerDriver(new Driver());//或者写：com.mysql.jdbc.Drive()
	//2.建立连接
	Connection conn = DriverManager.getConnection(
			"jdbc:mysql://localhost:3306/jdbc", "root", "123456");
	//3.创建语句
	Statement st = conn.createStatement();
	
	//4.执行语句
	ResultSet rs = st.executeQuery("select * from user");
	
	//5.处理结果
	while(rs.next()){//按行遍历
		//1是第一列
		System.out.println(rs.getObject(1) + "\t" + rs.getObject(2) +"\t" 
				+rs.getObject(3) + "\t" + rs.getObject(4));
	}
	
	//6.释放资源：关闭的顺序与创建顺序相反
	rs.close();
	st.close();
	conn.close();
}
```
### 1. 注册驱动(只做一次)
同一个程序可以注册很多驱动
####第1种方法
若果jar包不存在，编译无法通过
第一种方式在类加载时会执行静态的类加载（类似第三种注册方式，表明已注册一次），但是由于语句声明又会被注册一次，所以第一种方式加载了两次。产生了垃圾注册
```java
DriverManager.registerDriver(new Driver());//或者写：com.mysql.jdbc.Drive()
```
####第2种方法
```java
//第2种方法
System.setProperty("jdbc.drivers", "com.mysql.jdbc.Drive()");//setProperty(key, value)
```
在`value`部分，利用冒号来区分多个驱动；例如："com.mysql..:com.sqlserver"
####第3种方法
推荐方式
```java
Class.forName("com.mysql.jdbc.Driver");//根据类的名字，将类加载到jvm
```
### 2. 建立连接
```java
String  url= "jdbc:mysql://localhost:3306/jdbc";
String root = "root";
String password = "123456";
Connection conn = DriverManager.getConnection(
		url, root, password);
```
####url格式
```java
"jdbc:子协议:子名称(mysql里面没有子名称)//主机名:端口号(如果缺省，可以省略主机名与端口号)/数据库名
```
### 3. 创建、执行语句
```java
//3.创建语句
Statement st = conn.createStatement();
//4.执行语句
ResultSet rs = st.executeQuery("select * from user");
```
###4.处理结果
`rs.next()`是按行遍历，从下标为1的第1列开始
```java
while(rs.next()){//按行遍历
	//1是第一列
	System.out.println(rs.getObject(1) + "\t" + rs.getObject(2) +"\t" 
			+rs.getObject(3) + "\t" + rs.getObject(4));
}
```
###5.释放资源
后创建的要先关闭
- 数据库建立连接能力有限

```java
rs.close();
st.close();
conn.close();
```
connection是尽量晚建立，早释放==>减少占用TCP/IP时间
##规范的示例，可当做模板使用
```java
#jdbc.java
public static void template() throws ClassNotFoundException, SQLException{
	Connection conn = null;
	Statement st = null;
	ResultSet rs = null;
	try {
			
		conn = JdbcUtils.getConnection();
		
		st = conn.createStatement();
		
		rs = st.executeQuery("select * from user");

		while(rs.next()){//按行遍历
			//1是第一列
			System.out.println(rs.getObject(1) + "\t" + rs.getObject(2) +"\t" 
					+rs.getObject(3) + "\t" + rs.getObject(4));
		}
	}finally{
		JdbcUtils.free(rs, st, conn);
	}	
}
```
其中，JdbcUtils是自定义的工具类，代码如下：
```java
package jdbc;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

/**
 * 工具类
 * @author RY
 *
 */
public final class JdbcUtils {
	private static String  url= "jdbc:mysql://localhost:3306/jdbc";
	private static String root = "root";
	private static String password = "123456";
	
	private JdbcUtils(){}
	//注册驱动
	static{
		try {
			Class.forName("com.mysql.jdbc.Driver");
		} catch (ClassNotFoundException e) {
			throw new ExceptionInInitializerError(e);
		}
	}
	
	/**
	 * 创建连接
	 * @return
	 * @throws SQLException
	 */
	public static Connection getConnection() throws SQLException{
		return  DriverManager.getConnection(url, root, password);
	}
	
	/**
	 * 释放资源
	 * @param rs
	 * @param st
	 * @param conn
	 */
	public static void free(ResultSet rs, Statement st, Connection conn){
		if(rs!=null){
			try {
				rs.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}finally{
				if(st!=null){
					try {
						st.close();
					} catch (SQLException e) {
						e.printStackTrace();
					}finally{
						if(conn!=null){
							try {
								conn.close();
							} catch (SQLException e) {
								e.printStackTrace();
							}
						}	
					}
				}	
			}
		}
	}	
}
```
***
#对行的操作：CRUD
C：create
R:  read
U: Update
D: Delete

使用**PreparedStatement**来替换Statement!!!

##C(create)
插入语句（`insert into ... values...`），利用：`st.executeUpdate(sql)`，其返回值是个int型的值，表示受到改变的行数
利用PreparedStatement：
```java
public static void create(String name, java.util.Date birthday, float money) throws SQLException{
	Connection conn = null;
	PreparedStatement ps = null;
	ResultSet rs = null;
	try{
		conn = JdbcUtils.getConnection();
		
		String sql = "INSERT INTO user (name, birthday, money) VALUES (?, ?, ?)";
		ps = conn.prepareStatement(sql);
		ps.setString(1, name);
		ps.setDate(2, new java.sql.Date(birthday.getTime()));
		ps.setFloat(3, money);
		int i = ps.executeUpdate();
		System.out.println("i = "+i);
	}finally{
		JdbcUtils.free(rs, ps, conn);
	}
}
```
###Date的转换
形参中的Date是java.util.*中的类，而sql接收的Date事java.sql.*中的类（是java.util的子类）
所以需要进行一个类的转换`new java.sql.Date(birthday.getTime()`
##R(read)
读取语句（`select ... from ... where...`），利用：`st.executeQuery(sql)`，其返回值是个ResultSet，可以逐行扫描读取结果
```java
public static void read() throws SQLException{
	Connection conn = null;
	Statement st = null;
	ResultSet rs = null;
	try {				
		conn = JdbcUtils.getConnection();		
		st = conn.createStatement();	
		//直接写*的缺点：导致不知道列名，以及取出数据量太大
		rs = st.executeQuery("select id, name, birthday, money from user");
		
		while(rs.next()){//按行遍历
			//根据列名来获取数据
			System.out.println(rs.getObject("id") + "\t" 
					+ rs.getObject("name") +"\t" 
					+rs.getObject("birthday") + "\t" 
					+ rs.getObject("money"));
		}
	}finally{
		JdbcUtils.free(rs, st, conn);
	}	
}
```
##U(update)
```java
public static void update() throws SQLException{
	Connection conn = null;
	Statement st = null;
	ResultSet rs = null;
	try {				
		conn = JdbcUtils.getConnection();		
		st = conn.createStatement();	
		
		String sql = "UPDATE user SET money =  money + 10";
		int i = st.executeUpdate(sql);
		System.out.println("i = "+i);

	}finally{
		JdbcUtils.free(rs, st, conn);
	}	
}
```
##D(delete)
```sql
public static void delete() throws SQLException{
	Connection conn = null;
	Statement st = null;
	ResultSet rs = null;
	try {				
		conn = JdbcUtils.getConnection();		
		st = conn.createStatement();	
		
		String sql = "DELEGE FROM user WHERE money < 1000";
		int i = st.executeUpdate(sql);
		System.out.println("i = "+i);

	}finally{
		JdbcUtils.free(rs, st, conn);
	}	
}
```
##注意
###不能将`conn`关闭，然后返回`rs`给上层
一旦`conn`关闭，`conn`后的都会无效，包括`st`与`rs`
- 但是必须全部关闭，后续会有**连接池**的概念
***
#其他
##SQL注入
```java
public static void read(String s){
...
rs = st.executeQuery("SELECT id, name, birthday, money FROM user WHERE name = '" + s + "'")；
}
```
期望：传入s=某个名字时，得到这个名字对应的一行的数据
但是，当`s = ' or 1 or ' `时，会输出所有数据，这是因为此时sql识别的语句是：
```sql
SELECT id, name, birthday, money 
FROM user
WHERE name = '' or 1 or ''#由于是or语法+1(恒为真)
```
###修复SQL注入：PreparedStatement
PreparedStatement是Statement的子类
- 将`Statement`改为`PreparedStatement`类，对sql语句进行预处理来避免SQL注入
- sql的语句里，不再利用拼接，而是利用`?`占位符
```sql
PreparedStatement ps = null;
String sql = "SELECT id, name, money, birthday FROM user WHERE name = ?";
ps = conn.preparedStatement(sql);
ps.setString(1, name);//表明第一个问号是name
rs = ps.executeQuery();
```
##大文本存储
数据库：clob_test
|field|type|
|--|--|
|id|int|
|big_text|text|
###如何从file中读取文件字符流
利用Reader与Writer来读取文件的字符流
利用装饰器：BufferedXXX
###往数据库写入大文本
从本地读取一个文件，写入数据库中
```java
String sql = "insert into clob_test(big_test) values (?) ";
ps = conn.prepareStatement(sql);
File file = new File("src/jdbc/JdbcUtils.java");
Reader reader = new BufferedReader(new FileReader(file));
ps.setCharacterStream(1, reader,(int)file.length());
int i = ps.executeUpdate();
reader.close();
```
###从数据库读取大文本
利用`Clob`类
```java
while(rs.next()){
	Clob clob = rs.getClob(1);
	Reader reader = clob.getCharacterStream();
	File file = new File("JdbcUtils_bak.java");
	Writer writer = new BufferedWriter(new FileWriter(file));
	char[]buff = new char[1024];
	for(int i=0;(i=reader.read(buff))>1; ){
		writer.write(buff, 0, i);
	}
	writer.close();
	reader.close();
}
```
##大字节数组
###如何从file中读取文件字节流
利用InputStream与OutputStream来读取文件的字符流
利用装饰器：BufferedXXX
###往数据库写入字节流
```java
String sql = "insert into blob_test(big_bit) values (?) ";
ps = conn.prepareStatement(sql);
File file = new File("src/jdbc/img.png");
InputStream reader = new BufferedInputStream(new FileInputStream(file));
ps.setBinaryStream(1, reader,(int)file.length());
int i = ps.executeUpdate();
reader.close();
```
###从数据库中读取字节流
```java
String sql = "select big_bit from blob_test";

rs = st.executeQuery(sql);	
while(rs.next()){
	InputStream in = rs.getBinaryStream(1);
	File file = new File("back.png");
	OutputStream out = new BufferedOutputStream(new FileOutputStream(file));
	byte[]buff = new byte[1024];
	for(int i=0;(i=in.read(buff))>1; ){
		out.write(buff, 0, i);
	}
	out.close();
	in.close();
}
```