layout: restful
title: "RESTful API 最佳实践"
date: 2015-10-22 16:22:11
tags:
---

###协议
API与用户的通信协议，总是使用HTTPs协议。

###域名
应该尽量将API部署在专用域名之下。

https://api.example.com
如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下。

https://example.org/api/

###版本（Versioning）
应该将API的版本号放入URL。

https://api.example.com/v1/
另一种做法是，将版本号放在HTTP头信息中，但不如放入URL方便和直观。Github采用这种做法。

###路径（Endpoint）
路径又称"终点"（endpoint），表示API的具体网址。
在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的"集合"（collection），所以API中的名词也应该使用复数。
举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。
<pre>
https://api.example.com/v1/zoos
https://api.example.com/v1/animals
https://api.example.com/v1/employees
</pre>

###HTTP动词
对于资源的具体操作类型，由HTTP动词表示。
常用的HTTP动词有下面五个（括号里是对应的SQL命令）。
GET（SELECT）：从服务器取出资源（一项或多项）。
POST（CREATE）：在服务器新建一个资源。
PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）。
PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性）。
DELETE（DELETE）：从服务器删除资源。

还有两个不常用的HTTP动词。
HEAD：获取资源的元数据。
OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的。

下面是一些例子。
GET /zoos：列出所有动物园
POST /zoos：新建一个动物园
GET /zoos/ID：获取某个指定动物园的信息
PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
DELETE /zoos/ID：删除某个动物园
GET /zoos/ID/animals：列出某个指定动物园的所有动物
DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

###使用复数名词
不要混淆名词单数和复数，为了保持简单，只对所有资源使用复数。

/cars 而不是 /car
/users 而不是 /user
/products 而不是 /product
/settings 而部署 /setting

###Get方法和查询参数不应该涉及状态改变
使用PUT, POST 和DELETE 方法 而不是 GET 方法来改变状态，不要使用GET 进行状态改变:

###在post,put,patch上使用json作为输入
使用json作为所有的API输出格式，接下来考虑API的输入数据格式。
很多的API使用url编码格式：就像是url查询参数的格式一样：单纯的键值对。这种方法简单有效，但是也有自己的问题：它没有数据类型的概念。这使得程序不得不根据字符串解析出布尔和整数,而且还没有层次结构–虽然有一些关于层次结构信息的约定存在可是和本身就支持层次结构的json比较一下还是不很好用。

当然如果API本身就很简单，那么使用url格式的输入没什么问题。但对于复杂的API你应该使用json。或者干脆统一使用json。
注意使用json传输的时候，要求请求头里面加入：Content-Type：application/json，否则抛出415异常（unsupported media type）。

###更新和创建操作应该返回资源
PUT、POST、PATCH 操作在对资源进行操作的时候常常有一些副作用：例如created_at,updated_at 时间戳。为了防止用户多次的API调用（为了进行此次的更新操作），我们应该会返回更新的资源（updated representation.）例如：在POST操作以后，返回201 created 状态码，并且包含一个指向新资源的url作为返回头

###不符合CURD的操作
对这个令人困惑的问题，下面是一些解决方法：
重构你的行为action。当你的行为不需要参数的时候，你可以把active对应到activated这个资源，（更新使用patch）.
以子资源对待。例如:github上，对一个gists加星操作：PUT /gists/:id/star 并且取消星操作：DELETE /gists/:id/star.
有时候action实在没有难以和某个资源对应上例如search。那就这么办吧。我认为API的使用者对于/search这种url也不会有太大意见的（毕竟他很容易理解）。只要注意在文档中写清楚就可以了。

###使用子资源表达关系
如果一个资源与另外一个资源有关系，使用子资源：

GET /cars/711/drivers/ 返回 car 711的所有司机
GET /cars/711/drivers/4 返回 car 711的4号司机

###过滤信息
如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。
下面是一些常见的参数。
?limit=10：指定返回记录的数量
?offset=10：指定返回记录的开始位置。
?page=2&per_page=100：指定第几页，以及每页的记录数。
?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
?animal_type_id=1：指定筛选条件
参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，GET /zoo/ID/animals 与 GET /animals?zoo_id=ID 的含义是相同的。

###状态码（Status Codes）
200 OK － 服务器代码按预期执行

###设计错误消息
所有结果的结构一致相同，结构如下：
<pre>
{
  code:0,
  message:'',
  errors:[
    {
      field:'',
      message:''
    }
  ]
  data:[]|{}
}
</pre>
code =0:  操作成功
     >0:  操作不成
message:  提示信息

通过code字段来表明这个请求的成功或失败；如果成功，则可以读取data数据进行后续处理；如果失败，可以读取message或errors进行错误的处理。
这样的好处就是，既不用关心HTTP状态码，也不用对成功或失败来处理不同的JSON结构；并且容易扩展，如加入总记录数。

###返回结果
针对不同操作，服务器向用户返回的结果应该符合以下规范。
GET /collection：返回资源对象的列表（数组）
GET /collection/resource：返回单个资源对象
POST /collection：返回新生成的资源对象
PUT /collection/resource：返回完整的资源对象
PATCH /collection/resource：返回完整的资源对象
DELETE /collection/resource：返回一个空文档

###Hypermedia API
RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。
比如，当用户向api.example.com的根目录发出请求，会得到这样一个文档。

<pre>
{"link": {
  "rel":   "collection https://www.example.com/zoos",
  "href":  "https://api.example.com/zoos",
  "title": "List of zoos",
  "type":  "application/vnd.yourformat+json"
}}
</pre>

上面代码表示，文档中有一个link属性，用户读取这个属性就知道下一步该调用什么API了。rel表示这个API与当前网址的关系（collection关系，并给出该collection的网址），href表示API的路径，title表示API的标题，type表示返回类型。
Hypermedia API的设计被称为HATEOAS。Github的API就是这种设计，访问api.github.com会得到一个所有可用API的网址列表。

<pre>
{
  "current_user_url": "https://api.github.com/user",
  "authorizations_url": "https://api.github.com/authorizations",
  // ...
}
</pre>

从上面可以看到，如果想获取当前用户的信息，应该去访问api.github.com/user，然后就得到了下面结果。

<pre>
{
  "message": "Requires authentication",
  "documentation_url": "https://developer.github.com/v3"
}
</pre>

上面代码表示，服务器给出了提示信息，以及文档的网址。

###其他
（1）API的身份认证应该使用OAuth 2.0框架。
（2）服务器返回的数据格式，应该尽量使用JSON，避免使用XML。