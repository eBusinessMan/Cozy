Cozy - Easy Java ORM（version:0.2.1）
===============

Cozy是一个简单的轻量级的Java ORM类库，设计灵感来源于go语言的beego ORM。

目前仍处于开发阶段，但api已基本稳定

v0.2.1——2015/09/13 update:
更正部分api，ORM的queryTable方法现在可以支持传入表名查询，也可以传入对象进行查询

v0.2——2015/09/04 update:
添加CRUD回调方法支持,更正异常处理方式

回调方法示例:
```java

//使用mysql驱动名，mysqlURL,数据库用户名，数据库密码，数据库类型创建一个OrmManager管理器，和JDBC用法差不多
OrmManager oManager = new OrmManager("com.mysql.jdbc.Driver", "jdbc:mysql://127.0.0.1:3306/cozytest", "root", "", "mysql");
//当调用orm.Insert(Student)时会在插入前调用beforeInsert(Student t,Ormer orm)方法，student和orm对象由cozy自动注入
//注意方法名必须是如下：beforeRead,afterRead,beforeInsert,afterInsert,beforeUpdate,afterUpdate,beforeDelete,afterDelete
//方法的第一个参数是要回调的对象，第二个是ormer接口
oManager.addCallback(new Object(){
	private void beforeInsert(Student t,Ormer orm){
			System.out.println("before t");
			System.out.println(t.getId()+" id before");
			System.out.println(orm);
		}
	private void afterInsert(Teacher t,Ormer orm){
		System.out.println("after t");
		System.out.println(t.getId()+" id after");
		System.out.println(orm);
	}
});

```

**Cozy设计初衷:**
* 熟悉java的反射和ORM的原理
* 对于小型项目用hibernate太重，直接上sql语句不爽
* 提高小项目开发效率

**支持数据库:**
* MySQL
* SQLite3(开发中)
* MSSQL(计划中)


对MySQl的JDBC驱动支持通过测试

**Cozy目前及计划特性:**
* 支持java的常见类型存储
* 轻松上手，采用简单的CRUD风格
* 允许直接使用SQL查询/映射
* 类似jquery的链式调用风格
* 内部处理了大部分Exception,让你的程序更干净
* 完全无XML配置，纯Annoation
* 内置简易的数据库连接池
* 极少的外部库依赖，其实也就是仅仅需要搭配JDBC驱动这个依赖
* 灵活可扩展的架构，可保证api不变前提下灵活定制替换自己的ORM实现
* 只支持核心的ORM功能，多余的事都不做，导致现在还不支持模型关系映射，不支持外键等关系

**暂时开发目标:**
* 增加更多数据库对应的查询操作
* SQLite3的支持
* MongoDB的支持
* 添加CRUD回调方法支持


**使用Cozy:**

下载编译好的[Cozy.jar](https://github.com/Comdex/Cozy/releases/download/0.2/cozy.jar)包在eclipse中导入使用就OK

### Cozy API使用注释
[Ormer接口](src/com/reflectsky/cozy/Ormer.java)

[QuerySet接口](src/com/reflectsky/cozy/QuerySet.java)

[RawPreparer接口](src/com/reflectsky/cozy/RawPreparer.java)

[RawSet接口](src/com/reflectsky/cozy/RawSet.java)

### 简单示例
先上个简单的实体类，约定优于配置，采取默认配置，实体类无须做配置，不强制要求有setter,getter方法

```java

public class User{
	//当数据类型为int,long,short时且没有其他主键时，字段名为id的字段会默  	认当做自增长主键
	private int id;
	//在mysql实现中，String类型默认对应varchar(255)
	private String name;
	
	.....此处省略若干setter,getter方法
}

```
使用方法如下(内置了mysql实现所以针对mysql的用例，务必先添加MySQL的JDBC驱动):

```java

public class Test{
	public static void main(String[] args){
		//使用mysql驱动名，mysqlURL,数据库用户名，数据库密码，数据库类型创建一个OrmManager管理器，和JDBC用法差不多
		OrmManager oManager = new OrmManager("com.mysql.jdbc.Driver", "jdbc:mysql://127.0.0.1:3306/cozytest", "root", "", "mysql");
		
		//要使用CRUD方法首先得注册模型
		oManager.RegisterModel(User.class);
		
		//设置参数false代表不进行drop table后重新建表，不用设置这个也是默认参数
		oManager.Force(false);
		
		//设置是否显示Cozy执行sql操作命令,默认为false
		oManager.Debug(true);
		
		//设置是否显示Cozy的建表操作，默认为false
		oManager.Verbose(true);
		
		//执行该方法表示自动建数据库表
		oManager.runSyncDB();
		
		//获取一个可用的Ormer
		Ormer orm = oManager.NewOrm();
		
		User user = new User();
		user.setName("blabla");
		
		//插入数据，该方法会返回user的自增长id
		orm.insert(user);
		
		//更新数据,该方法返回-1代表执行失败
		user.setName("hehe");
		orm.update(user);
		
		//读入数据
		user.setId(9);
		orm.read(user);
		
		//删除数据
		orm.delete(user);
		//这些方法的文档使用细节请参看接口注释
		
		//-------------------------------------------//
		//高级查询QuerySet接口的部分使用例子
		orm.queryTable("user")//返回一个QuerySet接口对象,参数为表名
		orm.queryTable("user").filter("id",1)//等同于sql：where user.id = 1
		orm.queryTable("user").filter("id",1).filter("name","pp")//链式调用，等同于sql：where user.id = 1 and user.name = 'pp'
		
		orm.quertTable("user").filter("id__gt",5)//id__(双下划线)gt是一个查询表达式，gt是操作符号关键词，等同于sql: where user.id>5
		
		/**当前支持的操作符号：
           exact / iexact 等于
           gt / gte 大于 / 大于等于
           lt / lte 小于 / 小于等于
           istartswith 以…起始
           iendswith 以…结束
           in
		*/
		
		//事务处理
		orm.begin();//事务开始
		........事务方法
		boolean ok = orm.commit();//事务提交，错误会返回false
		if(!ok){
			orm.rollback();//事务出错，就回滚事务
		}
		
		//暂时觉得ORM不爽或只用Sql才能更好实现的业务逻辑Cozy也提供了原生的JDBC操作Sql的方法
		Statement stmt = orm.createStatement();//获取jdbc的Statement接口
		PreparedStatement pstmt = orm.createPreparedStatement("select * from user where id = ?");
		
		//用完ORM记得释放资源
		orm.close();
	}
}

```

## 模型字段与数据库类型的对应

在此列出 ORM 推荐的对应数据库类型，自动建表功能也会以此为标准。

默认所有的字段都是 **NOT NULL**

#### MySQL

| java		   |mysql
| :---   	   | :---
| int - 设置 auto 或者名称为 `Id` 时 | integer AUTO_INCREMENT
| long - 设置 auto 或者名称为 `Id` 时 | bigint AUTO_INCREMENT
| short - 设置 auto 或者名称为 `Id` 时 | tinyint AUTO_INCREMENT
| boolean | bool
| string - 默认为 size 255 | varchar(size)
| string - 设置 type(text) 时 | longtext
| Date - 设置 type 为 date 时 | date
| Date | datetime
| float | double
| double | double
| double - 设置 digits, decimals 时  | numeric(digits, decimals)

## 注解的使用

@TableName("表名")：定义类对应的数据库表名

@Orm(pk=true):定义主键

@Orm(auto=true):定义自增键

@Orm(column="数据表列名")：定义类的属性对应的数据库表列名

@Orm(ignore=true):设置类属性被忽略

@Orm(type=""):设置类属性对应的数据库类型，仅对String,double,Date字段有效

@Orm(digits=,decimals=):仅对double有效，设置对应数据库的类型精度

@Orm(size=):设置类属性对应数据库类型的长度，目前仅对String有效，对应mysql的varchar(size)

@Orm(other="null;unique;index"):设置类型可空，唯一及建索引，用分号隔开

### 注解简单示例

```java

@TableName("myuser")
public class User{
	@Orm(pk=true,auto=true)
	private int sno;
	@orm(size=500)
	private String name;
	@Orm(type="longtext")
	private String address;
	.....不想写了
}

```

### 还有更多使用示例帮助待补充，本介绍尚未完善。。。。。。