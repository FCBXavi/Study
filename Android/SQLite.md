#SQLite
1. 继承SQLiteOpenHelper，实现onCreate()和onUpgrade()方法，在单例的构造方法中调用super新建数据库，在onCreate()方法中新建表。

2. 调用SQLiteOpenHelper的getReadableDatabase()和getWritableDatabase()可以得到可读和可写的数据库，然后可以进行操作。

WCDB加密的数据库是在调用helper的super方法时传入密码。

--
###建立索引的语句：

	//为整张表建立索引
	CREATE INDEX index_name ON table_name;
	//创建单列索引
	CREATE INDEX index_name ON table_name (column_name);
需要注意的是，索引会使查询的速度加快，但是却会使插入，删除和更新的速度变慢，在数据量比较小的时候，新建索引反而会增大数据库的大小。

--
###编译SQL语句

对于需要重复执行很多次的SQL语句，可以编译成SQLiteStatement语句。

 * 编译sql语句获得SQLiteStatement对象，参数使用?代替
 * 在循环中对SQLiteStatement对象进行具体数据绑定，bind方法中的index从1开始，不是0

--
###使用事务

批量操作，可以使用事务，减少反复数据库文件的IO操作。

--

###其它的优化

1. db.query语句，只返回需要的列；
2. 遍历cursor时，提前通过curcor.getColumnIndex()获取到index，不在循环中做；
3. ContentValue底层是用HashMap实现的，根据需要指定容量；
4. db和cursor对象及时关闭；
5. 耗时的操作放在子线程；

--