#Go Web  国际化    

## 设置默认区域  
####Locale 
Locale是一组描述世界上某一特定区域文本格式和语言习惯的设置的集合。  
1. 是一个强制性的,表示语言的缩写   
2. 跟在一个下划线之 后,是一个可选的国家说明符,用于区分讲同一种语言的不同国家   
3. 跟在一个句点之后,是可选的字符集说明符.例如"zh_CN.gb2312"表示中国使用gb2312 字符集。   

* 设置Locale   
	1. 通过域名设置Locale    
		• 通过URL就可以很明显的识别     
		• 有利于搜索引擎抓取,能够提高站点的SEO     
		
		~~~  
		 if r.Host == "www.asta.com" {
		  //通过子域名实现  
		  prefix := strings.Split(r.Host,".")
		~~~
	2. 通过参数实现   
	3. 客户端设置   
		`Accept-Language`  
		
		~~~  
		AL := r.Header.Get("Accept-Language")
		~~~
##本地化资源   
Go语言中将本地化格式信息存储子啊json中， 然后通过核实的方式展现出来    
* 本地化文本消息   
	建立需要的语言的map来维护一个key-value的关系   
	
	~~~  
	import "fmt"
		
		func msg(locale, key string) string {
			}
			return ""
		}
	~~~
* 本地化日期和时间   
	本地化日期时间需要考虑两个问题：  
		- 时区问题  
		- 格式问题   
	在go语言中，`$GOROOT/lib/time` 中的timeinfo.zip 包含有locale对应的时区定义   
	为了获得对应于当前locale的时间,我们 应首先使用 `time.LoadLocation(name string)` 获取相应于地区的locale  ，然后再利用此信息与调用 `time.Now` 获得的Time对象协作来获得最终的时间     
	
	~~~  
	en["time_zone"]="America/Chicago"
	~~~   

* 本地化视图资源  
	
	将文件进行locale区分    
	
	~~~  
	s1, _ := template.ParseFiles("views"+lang+"index.tpl")   
	VV.Lang=lang   
	s1.Execute(os.Stdout, VV)    
	
	// js文件
	<link href="views/{{.VV.Lang}}/css/bootstrap-responsive.min.css" rel="stylesheet">     
	<img src="views/{{.VV.Lang}}/images/btn.png">    
	~~~      
##国际化站点  

为了支持国际化,在此我们使用了一个国际化相关的包——go-i18n   
首先我们向go-i18n包注册config/locales 这个目录,以加载所有的locale文件   

~~~  
Tr:=i18n.NewLocale()

//测试
fmt.Println(Tr.Translate("submit")) //输出Submit
fmt.Println(Tr.Translate("submit")) //输出“递交”

~~~



* 自动加载本地包    

~~~  
//加载默认配置文件,这些文件都放在go-i18n/locales下面
}
 if fi.IsDir() {
			return err 
			}
 	}
}
~~~

通过上面的方法加载配置信息到默认的文件,这样我们就可以在我们没有自定义时间信息的时候执行如下的代码   

~~~  
//locale=zh的情况下,执行如下代码:
~~~


* template mapfunc   
Go语言的模板支持自定义模板函数,下面是我们实现的方便操作的mapfunc:    

	- 文本信息  

	~~~  
	func I18nT(args ...interface{}) string {
	return Tr.Translate(s)
~~~

	注册函数 ：  
	`t.Funcs(template.FuncMap{"T": I18nT})   `  
	模板使用：  
	`{{.V.Submit | T}}`    
	
	- 时间日期  
	
	~~~  
	 func I18nTimeDate(args ...interface{}) string {
	~~~
	
	注册函数：  
	`t.Funcs(template.FuncMap{"TD": I18nTimeDate})`    
	模板使用：  
	`{{.V.Now | TD}}`    
	
	- 货币信息  
		
	~~~  
	 func I18nMoney(args ...interface{}) string {
         }
	~~~    
	注册函数：  
	`t.Funcs(template.FuncMap{"M": I18nMoney})`   
	模板使用：  
	`{{.V.Money | M}}`   
	













