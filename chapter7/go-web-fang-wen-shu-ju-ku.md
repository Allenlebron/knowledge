#Go Web 访问数据库   
Go没有内置的驱动支持任何的数据库,但是Go定义了database/sql接口,用户可以基于驱动接口开发相应数据库 的驱动     

##database /sql 接口   
* sql.Register    
	此函数用来注册数据库驱动

~~~
//https://github.com/mattn/go-sqlite3驱动 func init() {
~~~   
其内部是通过一个map 来进行数据存储的， 可以定义多个驱动  
`var drivers = make(map[string]driver.Driver)`  只要不重复    

>  _ 的意思是引入后面的包名 而不直接使用这个包中定义的函数,变量等资源。  
> 包在引入的时候会自动调用包的init函数以完成 对包的初始化    

* driver.Driver   
是一个数据库驱动的接口，定义了一个 `method: Open(name string)` 返回一个数据库的Conn接口  

~~~  
type Driver interface {
~~~  
__注意__： 返回的Conn只能用来进行一次goroutine操作， 不能用于多个   

* driver.Conn  
是一个数据库连接的接口定义，定义了一系列方法。    

~~~  
type Conn interface {
    Begin() (Tx, error)
}     
~~~   
Prepare函数返回与当前连接相关的执行Sql语句的准备状态,可以进行查询、删除等操作。     
Close函数关闭当前的连接,执行释放连接拥有的资源等清理工作。因为驱动实现了database/sql里面建议的conn pool,所以你不用再去实现缓存conn之类的,这样会容易引起问题。     

* driver.Stmt   
Stmt是一种准备好的状态,和Conn相关联,而且只能应用于一个goroutine中,不能应用于多个goroutine。    

~~~  
type Stmt interface {
~~~    
Close函数关闭当前的链接状态,但是如果当前正在执行query,query还是有效返回rows数据。    
NumInput函数返回当前预留参数的个数,当返回>=0时数据库驱动就会智能检查调用者的参数。当数据库驱动包不 知道预留参数的时候,返回-1。
Exec函数执行Prepare准备好的sql,传入参数执行update/insert等操作,返回Result数据     
Query函数执行Prepare准备好的sql,传入需要的参数执行select操作,返回Rows结果集      


* driver.Tx   
事务处理一般就两个过程,递交或者回滚。数据库驱动里面也只需要实现这两个函数就可以    

~~~  
type Tx interface {
~~~  

* driver.Execer   
是一个Conn可选实现接口     

~~~  
type Execer interface {
~~~   
如果这个接口没有定义,那么在调用DB.Exec,就会首先调用Prepare返回Stmt,然后执行Stmt的Exec,然后关闭St mt。     

* driver.Result    
是执行Update/Insert 等操作返回的结果接口定义    

~~~  
type Result interface {
~~~   
LastInsertId函数返回由数据库执行插入操作得到的自增ID号。    RowsAffected函数返回query操作影响的数据条目数。     

* driver.Rows    
Rows是执行查询返回的结果集接口定义    

~~~  
type Rows interface {
~~~  
Columns函数返回查询数据库表的字段信息,这个返回的slice和sql查询的字段一一对应,而不是返回整个表的所 有字段。      
Close函数用来关闭Rows迭代器。     
Next函数用来返回下一条数据,把数据赋值给dest。dest里面的元素必须是driver.Value的值除了string,返回 __的数据里面所有的string都必须要转换成[]byte__。如果最后没数据了,Next函数最后返回io.EOF。      

* driver.RowsAffected     
RowsAffected其实就是一个int64的别名,但是他实现了Result接口,用来底层实现Result的表示方式      

~~~  
type RowsAffected int64
~~~  

* driver.Value   
是一个空接口，可以容纳任何的数据     
drive的value必须是驱动能够操作的Value，要么是nil, 要么是下面的任意一种    

~~~  
int64
~~~   

* driver.ValueConverter   
ValueConverter接口定义了如何把一个普通的值转化成driver.Value的接口   

~~~  
type ValueConverter interface {
~~~   

好处：  
1. 转化driver.value到数据库表相应的字段,例如int64的数据如何转化成数据库表uint16字段     
2.  把数据库查询结果转化成driver.Value值    
3. 在scan函数里面如何把driver.Value值转化成用户定义的值
Valuer接口定义了返回一个driver.Value的方式     

~~~  
type Valuer interface {
~~~   

很多类型都实现了这个Value方法,用来自身与driver.Value的转化。
* database/sql   
database/sql在database/sql/driver提供的接口基础上定义了一些更高阶的方法,用以简化数据库操作,同时内 部还建议性地实现一个conn pool。    

~~~  
type DB struct {
~~~
我们可以看到Open函数返回的是DB对象,里面有一个freeConn,它就是那个简易的连接池。它的实现相当简单或 者说简陋,就是当执行Db.prepare的时候会 defer db.putConn(ci, err) ,也就是把这个连接放入连接池,每次调 用conn的时候会先判断freeConn的长度是否大于0,大于0说明有可以复用的conn,直接拿出来用就是了,如果不 大于0,则创建一个conn,然后再返回之。    




## mysql 使用    

~~~  
CREATE TABLE `userinfo` (
     `intro` TEXT NULL,

~~~   

~~~   
package main
第 5 章 5 访问数据库 | 180
~~~  

代码解释：  
`sql.Open()`: 用来打开一个注册过的数据库驱动    
`db.Prepare()`: 用来返回准备要执行的sql操作,然后返回准备完毕的执行状态。      
`db.Query()`: 用来直接执行Sql返回Rows结果。    
`stmt.Exec()`: 用来执行stmt准备好的SQL语句         


##PostgreSQL    
开源数据库    
 
~~~  
CREATE TABLE userinfo
 	uid integer,

~~~   

~~~  
import (
第 5 章 5 访问数据库 | 187
~~~      
使用的是`$1` 方式制定要传递的参数，不是使用的`?`    
sql.Open 中的dsn信息的格式与MySQL驱动中的dsn格式不一样     
pg 不支持 LastInsertId函数， 所以没有自增ID 返回    


--------------------------   
##beeDB ORM 开发    
* 安装   
	`go get github.com/astaxie/beedb`    
* 初始化   
	
	~~~   
	import (
	~~~   
* 打开数据库并创建beedb对象    

	~~~ 
	db, err := sql.Open("mymysql", "test/xiemengjun/123456")
	~~~   
	new方法其实有两个参数，第一个是标准接口db， 第二个是数据库引擎    
	`beedb.OnDebug=true`    

* 建立相应的struct    
	
	~~~   
	type Userinfo struct {
	~~~   
	> beedb针对驼峰命名会自动帮你转化成下划线字段,例如你定义了Struct名字为 UserInfo ,那么转化 成底层实现的时候是 user_info ,字段命名也遵循该规则。    

* 插入数据    
	
	~~~   
	var saveone Userinfo
	~~~   
	
	通过map 进行数据插入    
	
	~~~  
	add := make(map[string]interface{})
	~~~   
	
	插入多条数据   
	
	~~~  
	addslice := make([]map[string]interface{}, 0)
	~~~   

* 更新数据   
	
	~~~  
	saveone.Username = "Update Username"
	~~~   
	
	使用map更新数据   
	
	~~~  
	t := make(map[string]interface{})
	~~~   
	SetPK:显式的告诉ORM,数据库表 userinfo 的主键是 uid 。    
	Where:用来设置条件,支持多个参数,第一个参数如果为整数,相当于调用了Where("主键=?",值)。 Updata函数 接收map类型的数据,执行更新数据。     

* 查询   
	
	~~~  
	var user Userinfo //Where接受两个参数,支持整形参数 	orm.Where("uid=?", 27).Find(&user)
	
	var user2 Userinfo
	
	var user3 Userinfo //Where接受两个参数,支持字符型的参数 	orm.Where("name = ?", "john").Find(&user3)
	
	//条件查询  
	var user4 Userinfo
	
	//获取10条从 20 条开始
	var allusers []Userinfo
	
	//获取10 条
	var tenusers []Userinfo
	
	//获取全部
	var everyone []Userinfo
	~~~

	获取数据到map中   
	`a, _ := orm.SetTable("userinfo").SetPK("uid").Where(2).Select("uid,username").FindMap()`    
	

* 删除数据   
	
	~~~
	//saveone就是上面示例中的那个saveone 
	orm.Delete(&saveone)   
	
	//alluser就是上面定义的获取多条数据的slice 	orm.DeleteAll(&alluser)   
	
	rm.SetTable("userinfo").Where("uid>?", 3).DeleteRow()   
	~~~   
	
* 关联查询     
	
	~~~    
	a, _ := orm.SetTable("userinfo").Join("LEFT", "userdeatail", "userinfo.uid=userdeatail.uid").Where("userinfo.uid=?", 1)    
	第一个参数可以是INNER, LEFT, OUTER, CROSS等
	第二个表示连接的表  
	第三个参数表示连接的条件  
	~~~   
	
* Group By 和 Having     
`a, _ := orm.SetTable("userinfo").GroupBy("username").Having("username='astaxie'").FindMap()`    



##NOSQL 数据库    
* redis   

	~~~  
	package main
		第 5 章 5 访问数据库 | 196
	~~~

* mongDB   
	![](http://omy43wh36.bkt.clouddn.com/Snip20161211_20.png)   
	
	~~~  
	package main
	type Person struct {
	 	if err != nil {
	~~~