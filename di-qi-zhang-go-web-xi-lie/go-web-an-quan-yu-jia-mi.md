#Go Web 安全与加密   
__对于用户的输入数据,在对其进行 验证之前都应该将其视为不安全的数据__    
如果直接把这些不安全的数据输出到客户端,就可能造成跨站脚本攻 击(XSS)的问题    


##预防CSRF攻击  
CSRF(Cross-site request forgery),中文名称:跨站请求伪造,也被称为:one click attack/session ridi ng,缩写为:CSRF/XSRF。

完成此攻击需要满足的条件：  
1. 登录受信任网站A,并在本地生成Cookie 。   
2. 在不退出A的情况下,访问危险网站B。   

防止方式：  

1. 正确使用GET,POST和Cookie;     
	get 只用来读取， post用来操作数据  	
2. 在非GET请求中增加伪随机数;    
	* 为每个用户生成一个唯一的cookie token， 所有表单都包含同一个伪随机值  
	* 每个请求使用验证码,这个方案是完美的,因为要多次输入验证码,所以用户友好性很差     
	* 不同的表单包含一个不同的伪随机值    

~~~  

 h := md5.New()
 
 
  r.ParseForm()
~~~    
##确保输入过滤  
通过对所有的输入数据进行过滤,可以避免恶 意数据在程序中被误信或误用     

1. 识别数据,搞清楚需要过滤的数据来自于哪里    
Go中对于用户输入的数据很好识别：Go通过`r.ParseForm`之后,把用户POST和GET的数据全部放在了`r.Form` 里面     
`r.Header` 中的而很多数据是客户端操作的   
最好的方式是将所有的数据看成是用户输入的   

2. 过滤数据,弄明白我们需要什么样的数据     
过滤数据的一些库：  
	* `strconv`zi符串转化相关函数  
	* string  
	* regexp

区分过滤数据  
把所有经过过滤的数据放入一个 叫全局的Map变量中(CleanMap)。这时需要用两个重要的步骤来防止被污染数据的注入:  

 - 每个请求都要初始化C leanMap为一个空Map。 
 - 加入检查及阻止来自外部数据源的变量命名为CleanMap。    

~~~
<form action="/whoami" method="POST"> 我是谁:

//白名单验证
r.ParseForm()

r.ParseForm()

~~~


3. 区分已过滤及被污染数据,如果存在攻击数据那么保证过滤之后可以让我们使用更安全的数据


##避免XSS攻击   
跨站脚本攻击(Cross-Site Scripting)    
XSS是一种常见的web安全漏洞,它允许攻击者将恶意代码植入到提供给其它 用户使用的页面中。   
XSS的攻击目标是为了盗取存储在客户端的cookie或者其他网站用于识别客户端身份的敏感信息。     

* 一类是存储型XSS,主要出现在让用户输入数据,供其他浏览此页的用户进行查看的地 方,包括留言、评论、博客日志和各类表单等    
* 另一类是反射型XSS,主要做法是将脚本代码加入URL地址的 请求参数里,请求参数进入程序后在页面直接输出,用户点击类似的恶意链接就可能受到攻击。    

__攻击目的和手段__：  

__防止手段__：   
 


##避免SQL注入   
可以用它来从数据库获取 敏感信息,或者利用数据库的特性执行添加用户,导出文件等一系列恶意操作,甚至有可能获取数据库乃至系统 用户最高 限。
__原因__： 是因为程序没有有效过滤用户的输入,使攻击者成功的向服务器提交恶意的SQL查询代 码,程序在接收后错误的将攻击者的输入作为查询语句的一部分执行,导致原始的查询逻辑被改变,额外的执行 了攻击者精心构造的恶意代码。    


~~~   
username:=r.Form.Get("username")

如果用户输入的是一下用户名  
myuser' or 'foo' = 'foo' --   
SELECT * FROM user WHERE username='myuser' or 'foo'=='foo' --'' AND password='xxx'  

 -- 是注释标记,所以查询语句会在此中断
~~~


__预防SQL注入__   

1. 严格限制Web应用的数据库的操作 限,给此用户提供仅仅能够满足其工作的最低 限,从而最大限度的减少 注入攻击对数据库的危害。



##存储密码      

1. 单向hash算法    
常用的有HA-256, SHA-1, MD5等   

~~~  
//import "crypto/sha256"
~~~   

加盐hash算法  

~~~  
//import "crypto/md5" //假设用户名abc,密码123456
~~~   

2. 故意增加密码计算所需耗费的资 源和时间
故意增加密码计算所需耗费的资 源和时间,使得任何人都不可获得足够的资源建立所需的 rainbow table 。     
`scrypt`:scrypt是由著名的FreeBSD黑客Colin Percival为他的备份服务Tarsnap开发的   

`http://code.google.com/p/go/source/browse?repo=crypto#hg%2Fscrypt`    

`dk := scrypt.Key([]byte("some password"), []byte(salt), 16384, 8, 1, 32)`    


3. 简单数据 ---- base64 加密   

~~~  
package main
	fmt.Println(string(enbyte))
}
	
~~~

4. 高级加密    

Go语言的 crypto 里面支持对称加密的高级加解密包有:   

1. crypto/aes 包:AES(Advanced Encryption Standard),又称Rijndael加密法,是美国联邦政府采用的一种 区块加密标准。     
2. crypto/des 包:DES(Data Encryption Standard),是一种对称加密标准,是目前使用最广泛的密钥系 统,特别是在保护金融数据的安全中。曾是美国联邦政府的加密标准,但现已被AES所替代。     


~~~   
package main
	//aes的加密字符串

  if err != nil {
~~~
通过调用函数 aes.NewCipher (参数key必须是16、24或者32位的[]byte,分别对应AES-128, AES-192或AES-2

~~~
type Block interface {
~~~










