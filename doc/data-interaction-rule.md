前后端开发约定
============

数据交互一律使用 JSON 格式

目录：

1. [URL 约定](#hash_url1)
2. [项目迁移解决方案](#hash_change_url2)
3. [常用 Request 参数约定](#hash_request3)
4. [AJAX 数据格式约定](#hash_ajaxdata4)
    4.1 [JSON常用参数约定](#hash_ajaxdata4_1)
    4.2 [文件上传](#hash_ajaxdata4_2)
5. [数据接口约定](#hash_dataapi5)
    5.1 [数据精简](#hash_dataapi5_1)
    5.2 [状态标识](#hash_dataapi5_2)
    5.3 [静态数据和动态数据](#hash_dataapi5_3)
6. [技术选型](#hash_technical_options6)


<a name="hash_url1" title="URL约定"></a>

URL 约定
-------

前端同事有本地模拟后端数据的环境，所以交付给后端的 Demo 中会写上真实路径。比如：

```html
<!-- 传统的 Demo -->
请<a href="login.html">点击此处</a>登录

<!-- 真实路径 Demo -->
请<a href="/login/">点击此处</a>登录
```

所以，需要在项目开发前约定好URL的格式。

**默认约定：**

一、使用下划线作为URL分隔符

```
√ /change_password/
x /change-password/
```


二、如需兼容老版本的 URL 以优化 SEO，则通过服务器的 Rewrite 提供支持.

三、AJAX一律使用 /ajax/ 前缀
> 某些POST请求优先使用当前页面，例如： 页面或AJAX存在`GET /news/?id=2` AJAX则使用`POST /news/ {id:1,body:"新闻内容"}`

```
√ /ajax/get_user/
x /ajax_get_user/
```

<a name="hash_change_url2" title="项目迁移解决方案"></a>

### 项目迁移解决方案
考虑如下场景：
```html
<!-- 首页代码 -->
您好，请<a href="/login/">登录</a>
```
项目是一个博客系统，域名是 `http://www.domain.com` 登录地址是 `http://www.domian.com/login/` 。

上线后需求方要求将博客迁移至 `http://www.domain.com/blog/` 。

迁移后访问首页，点击登录`(/login/)`。打开 `/login/` 页面后出现404。因为博客的登录页面变成了 `/blog/login/`，而页面中的链接没有修改。

此时需要将所有页面中的 URL 都加上 `/blog/` 前缀才可以确保所有 URL 正确，`/login/` 改为 `/blog/login/` 等。

------------

当项目迁移至子目录时，因为 URL 前缀固定导致所有页面需要同时修改。我们通过前缀变量的方式解决这个问题。

PHP代码
```html
define("APP_PATH","");
您好，请<a href="<?php echo $APP_PATH ?>/login/">登录</a>
```

**渲染结果:**
> 您好，请`<a href="/login/">登录</a>`

此处是原生 PHP 渲染页面示例，不同后端框架渲染页面方式不同。大致都是定义一个常量，每个 URL 都加上此常量。

使用此方案后，可通过修改常量完成所有页面 URL 的迁移。

```html
define("APP_PATH","/blog");
您好，请<a href="<?php echo $APP_PATH ?>/login/">登录</a>
```

**渲染结果:**
> 您好，请`<a href="/blog/login/">登录</a>`

---

**前端注意：**  
AJAX 路径也需要加上项目路径前缀，防止项目迁移 AJAX 路径错误。参考如下示例：
```php
<script>
var APP_PATH = "<?php echo APP_PATH ?>";
</script>
<script>
$.get(APP_PATH + '/url/', function () {
    // ...
})
</script>
```

--------

<a name="hash_request3" title="常用 Request 参数约定"></a>

常用 Request 参数约定
-------------------
约定 Request 中常见的提交参数。

1. `p` 页数（默认第一页）
2. `pagesize` 每页显示数(默认10页)
3. `keyword` 搜索全局字符
4. `keyword_name` 搜索指定数据字符

**keyword** 示例：

keyword 表示全局搜索

```html
<form action="submit.php" method="get">
<input type="text" name="keyword" value="qq" />
<input type="submit" value="search"/>
</form>
```

```javascript
// 数据库中的数据
[
    {
        "id": 1,
        "email": "test@163.com",
        "note": "163邮箱，不是qq邮箱"
    },
    {
        "id": 2,
        "email": "demo@qq.com",
        "note": "内容"
    },
    {
        "id": 3,
        "email": "mail@gmail.com",
        "note": "备注"
    }
]
```

根据 `<form>` 提交参数: `submit.php?keyword=qq` 将会返回：

```javascript
{
    status: "success",
    data: [   
        // note 包含了 qq 
        {
            "email": "test@163.com",
            "note": "163邮箱，不是qq邮箱"
        },
        // email 包含了 qq
        {
            "email": "demo@qq.com",
            "note": "内容"
        }
    ]
}
```

如果只想搜索 email 则修改 name 为 keyword_email

`<input type="text" name="keyword_email" value="qq" />`

根据 `<form>` 提交参数: `submit.php?keyword_email=qq` 将会返回：

```javascript
{
    status: "success",
    data: [   
        // email 包含了 qq
        {
            "email": "demo@qq.com",
            "note": "内容"
        }
    ]
}
```

### json

前端会将一些格式复杂的数据通过 JSON 提交给后端，这些数据以 JSON 作为 `key` ，`value` 是 JSON 字符串。
```
GET
/ajax/add_news/
?id=1&json={%22name%22:[1,2,3,4]}
// ?id=1&json={"name":[1,2,3,4]}
```

<a name="hash_ajaxdata4" title="ajax数据格式约定"></a>

AJAX 数据交互约定
---------------

使用 JSON 格式

默认成功返回：

```javascript
{
    "status": "success"
}
```

默认失败返回：

```javascript
{
    "status": "error",
    "msg": "error detail"
}
```
注意需要用户登录状态下才能请求成功的 AJAX 在用户没有登录时应当返回：  

```javascript
{
    "status": "error",
    "msg": "nologin"
}

// 或通过状态值明确表示未登录（约定的作用在于前端可以通过 nologin 的标识判断错误类型，从而自由的控制错误UI）

{
    "status": "nologin",
}
```

建议后端通过如下代码判断页面请求是否为AJAX并根据结果返回内容：

```php
if ($_SERVER['HTTP_X_REQUESTED_WITH'] === 'XMLHttpRequest' || $_GET['X-Requested-With'] === 'XMLHttpRequest' || $_POST['X-Requested-With'] === 'XMLHttpRequest') {
    // AJAX 请求此页面时返回JSON
    echo '{"status":"error","msg":"您没有权限访问此页面，请确认您是否登录。"}';
} else {
    // 浏览器直接访问此页面返回登录页面
    echo <<<EOD    
    
    <html>
    <body>
    Username：<input type="text" />
    Password：<input type="password" />
    </body>
    </html>
    
    EOD;  
}
```

------------
**前端注意：**   
`Request Headers` 的 `X-Requested-With` 和 `GET/POST X-Requested-With` 都可以通过组件 API 添加，比如 jQuery等类库AJAX请求时自带了 `X-Requested-With: XMLHttpRequest`。

不带 `X-Requested-With: XMLHttpRequest`的组件可通过设置GET和POST参数 X-Requested-With 为 XMLHttpRequest 的方式标识给后端这是AJAX。

检测方法：  
Chrome developer tool > Network >  XHR > 左侧列表 > Headers > Request Headers 

存在 `X-Requested-With`  
![Request Headers](https://cloud.githubusercontent.com/assets/3949015/6520889/dd3df26c-c408-11e4-8758-1f9aa86f9439.png)

不存在 `X-Requested-With`  
![Not X-Requested-With](https://cloud.githubusercontent.com/assets/3949015/6521883/eefc0458-c417-11e4-9134-7d2fd442e4f6.gif)


------------

<a title="JSON常用参数约定" name="hash_ajaxdata4_1"></a>

### JSON常用参数约定

#### msg


```javascript
{   
    "status:": "error",
    "msg": "您已经赞过人"
}
```
`msg` 是 message 的缩写，用于存放消息


#### src

```javascript
{   
    "status:": "success",
    "src": "http://www.domain.com/xxx/demo.jpg"
}
```
`src` 是图片视频等资源的在线地址



#### url

```javascript
{   
    "status:": "success",
    "url": "/article/?id=3"
}
```
`url` 是页面路径信息，列如：  
前端提交一个发布文章的 AJAX ，后端返回数据后前端在页面弹窗提示:

```javascript
您已经成提交，请<a href="/article/?id=3">点击此处</a>查看文章
```

注意：不要将url 与 src 混淆，src 用于存放图片视频等资源，url 用于存放页面地址.


#### data

`data` 存放 `msg` `src` `url` 之外的数据，例如：  
前端发送AJAX获取新闻列表，后端将列表数据存放在 data.lists 中，前端将数据在浏览器渲染：

```javascript
// AJAX 返回的 JSON
{
    status: "success",
    data: {
        lists: [
            {
                title: '第87届奥斯卡金像奖红毯秀',
                time: '2015-02-23  06:14:00',
                link: '/news/?id=3'
            },
            {
                title: '美国民众暴雪天气中跳楼取乐',
                time: '2015-02-22 04:12:37',
                link: '/news/?id=2'
            },
            {
                title: '金正恩主持朝鲜党中央军委扩大会议',
                time: '2015-02-21 10:32:11',
                link: '/news/?id=1'
            }
        ]
    }
}

```


```html
<!-- 前端模板 -->
<ul>
    {{#data.lists}}
    <li>
        <a href="{{link}}">{{title}}</a>
        发布于：{{time}}
    </li>
    {{/data.lists}}
</ul>
```

```html
<!-- 渲染结果 -->
    <ul>
        <li>
            <a href="/news/?id=3">第87届奥斯卡金像奖红毯秀</a>
            发布于：2015-02-23  06:14:00
        </li>
        <li>
            <a href="/news/?id=2">美国民众暴雪天气中跳楼取乐</a>
            发布于：2015-02-22 04:12:37
        </li>
        <li>
            <a href="/news/?id=1">金正恩主持朝鲜党中央军委扩大会议</a>
            发布于：2015-02-21 10:32:11
        </li>
    </ul>
```

#### data.id

```javascript
{
    "status":"success",
    "data":{
        id: "pic_739515",
    },
    "src":"http://www.domain.com/xxx/demo.jpg"
}
```
`val` 保存会提交给后端的数据，列如：  

用户上传头像，前端通过 AJAX 向 `/ajax/uploader/` 上传图片后。

前端将后端返回JSON中的 src 属性转换为图片在浏览器展示出来。

并将 data.id 中的 "pic_739515" 保存在 `<input type="hidden" name="pic_value" value="pic_739515">` 中当用户保存信息时候将 `pic_739515` 一起提交给后端。

#### data.pagecount

`pagecount` 总页数，例如：

```javascript
// AJAX 返回的 JSON
{
    data: {
        lists: [
            {
                title: '第87届奥斯卡金像奖红毯秀',
                time: '2015-02-23  06:14:00',
                link: '/news/?id=3'
            },
            {
                title: '美国民众暴雪天气中跳楼取乐',
                time: '2015-02-22 04:12:37',
                link: '/news/?id=2'
            },
            {
                title: '金正恩主持朝鲜党中央军委扩大会议',
                time: '2015-02-21 10:32:11',
                link: '/news/?id=1'
            }
        ],
        pagecount: 20
    }
}
```

#### data.count

`count` 数据总数，例如：

> 每页显示 `3` 条，页数有 `20` 页，共有 `612` 条数据

```javascript
// AJAX 返回的 JSON
{
    data: {
        lists: [
            {
                title: '第87届奥斯卡金像奖红毯秀',
                time: '2015-02-23  06:14:00',
                link: '/news/?id=3'
            },
            {
                title: '美国民众暴雪天气中跳楼取乐',
                time: '2015-02-22 04:12:37',
                link: '/news/?id=2'
            },
            {
                title: '金正恩主持朝鲜党中央军委扩大会议',
                time: '2015-02-21 10:32:11',
                link: '/news/?id=1'
            }
        ],
        pagecount: 20,
        count: 612
    }
}
```

<a title="文件上传" name="hash_ajaxdata4_2"></a>

### 文件上传

[/uploadify/](uploadify.md)

<a name="hash_dataapi5" title="数据接口约定"></a>

数据接口约定
-------------
此数据接口约定的最大作用是统一方案，防止前端所需数据与后端提供数据不一致，导致前后端返工联调。可根据不同项目自由调整，如果要做调整一定要前后端达成一致，并形成文档。**无特殊情况则使用这套方案**。

<a title="数据精简" name="hash_dataapi5_1"></a>

### 数据精简
数据尽量精简，只提供关键的动态数据。不提供可由前端扩展的数据。

例如一个新闻列表功能开发：  
当后端确定新闻页面的 URL 不会变动并通过 `/news/?news_id=` 加新闻ID 可以 GET 传参访问页面时。后端只需返回新闻的ID列表，无需返回完整 URL。由前端二次封装出完成数据。
```javascript
// 后端提供的数据
[
    {
        id: 1
    },
    {
        id: 2
    },
]
```

```javascript
// 前端封装成完整 URL 列表
$.getJSON('/ajax/news/', function (data) {
    $.each(data, function () {
        // APP_PATH 是项目顶级目录，比如此处 APP_PATH = '/'；
        this.url = APP_PATH + 'news/?news_id=' + this.id;
    })
})
```

<a title="状态标识" name="hash_dataapi5_2"></a>

### 状态标识
状态信息一律使用固定的数字和小写英文字符，不要使用中文字符或者动态文字。

```javascript
// Good
{
    "task_status": "done"
}
// Good
{
    "task_status": 1,
}

// Bad
{
    "task_status": "成功"
}
```

#### 小写英文标识示例

例如 `task_status` 会有 `success` `wait` `fail` 三个状态。则只能新增 `delete`状态，不能修改 `success` 为 `done`。因为前端会根据固定的值编写匹配规则，如下代码：

```javascript
var status_msg;
switch (task_status) {
    case "success":
    status_msg = "成功";
    break;
    case "wait":
    status_msg = "等待";
    break;
    case "fail":
    status_msg = "失败";
    break;
    default:
    status_msg = "未知";
}
```

一旦 `success` 被修改为 `done`，前端必须修改 JavaScript 中的匹配规则代码，而有时前端匹配代码散落在很多地方，不一定能找到所有代码的 `success` 修改为 `done`。故此，状态信息（标识信息）只许增加不许修改。

例如不要将 `task_status` 与管理后台可控制的 `task_status_msg` 状态文字共用。

**可通过管理后台修改状态文字：**

```html
<form>
    修改文章发布状态：<br>
    成功状态文字：<input type="text" name="status_success_msg" value="OK" /> <br>
    等待状态文字：<input type="text" name="status_wait_msg" value="正在审核" /> <br>
    失败状态文字：<input type="text" name="status_fail_msg" value="审核失败" /> <br>
    <input type="submit" value="确定" />
</form>
```

```javascript
// Bad
{
    "status": "OK"
}

// Good
// 标识(success)确定后，永远不要变
{
    "status": "success",
    "status_msg": "OK"
}
```

`task_status` 只是个示例，实际开发中任何用于做匹配规则的属性都属于匹配规则。

<a title="静态数据和动态数据" name="hash_dataapi5_3"></a>

### 静态数据和动态数据

后端接口只提供动态数据。静态数据全部写在 View 层(在模板中编写)。

#### 静态数据示例

一、URL 示例

```javascript
// 已确定 用户中心的 URL 是固定不变的，所以直接写在模板里面
<a href="{{$APP_PATH}}/user/">用户中心</a>
```

二、固定文字信息（网站管理员不可通过管理后台修改的信息）

```html
// Smarty 模板引擎语法：当任务状态是 done 时，显示文字：完成
{{if $task_status == "done"}}完成{{/if}}
```

#### 动态数据示例

一、ID（文章ID、其他用户ID等）

```javascript
// 动静结合的示例（URL + ID）
<a href="{{$APP_PATH}}/news/?id={{$new_id}}">新闻：oxoxoxxo</a>
```

二、动态文字信息(管理后台可控制的信息)

```html
状态：{{$task_status_msg}}
// 渲染结果
// 1. 状态：成功
// 2. 状态：等待
// 3. 状态：失败
```

<a name="hash_technical_options6" title="技术选型"></a>

技术选型
-------
### 页面渲染方式

要解决的痛点：**后端将前端 Demo “翻译”成后端模板引擎**

如下：

```html
<!-- 渲染结果：免费用户状态 -->
<html>
<head>
    <title>页面标题</title>
</head>
<body>
    <ul>
        <li>欢迎你， Nimo</li>
    </ul>
</body>
</html>
```

```html
<!-- 渲染结果：付费用户状态 -->
<html>
<head>
    <title>页面标题</title>
</head>
<body>
    <ul>
        <li>欢迎你，会员 Nimo</li>
        <li><a href="/email/">收取邮件</a></li>
    </ul>
</body>
</html>

```



```php
<!-- 渲染方式：PHP 直接解析 -->
<?php include "header_1.php" ?>
<title>页面标题</title>
<?php include "header_2.php" ?>
<ul>
    <li>
    欢迎你，
    <?php echo $status === "free"? "普通会员": "会员"; ?>
    <?php echo $username ?>
    </li>
    <?php echo $status === 'free'? '': '<li><a href="/email/">收取邮件</a></li>'; ?>
</ul>
<?php include "footer.php" ?>
```

```php
<!-- 渲染方式：Smart 模板渲染 -->
{include file="header_1.tpl"}
<title>页面标题</title>
{include file="header_2.tpl"}
<ul>
    <li>
    欢迎你，
    {if $status=='free'}
        普通会员
    {else}
        会员
    {/if}
    {username}
    </li>
    {if $status=='free'}
        <li><a href="/email/">收取邮件</a></li>
    {/if}
    {username}
</ul>
{include file="footer.tpl"}
```

以上是一个简单的前端 Demo “翻译”工作。复杂的模板翻译工作因为 HTML 输出逻辑复杂，经常会出现最终结果与前端 Demo 输出不一致。增加大量的联调和沟通的工作量。

**考虑三种解决方案：**

1. SPA(Single-page application) 所有用到的展现数据都是后端通过异步接口(AJAX/JSONP)的方式提供的。由后端提供JSON数据，前端渲染HTML。
2. 后端使用 [smarty 模板引擎](http://www.smarty.net/)，由前端使用 [FMS](http://github.com/nimojs/fms) 编写后端模板。在 [MVC框架](http://baike.baidu.com/view/5432454.htm) 中后端专注在 Controller 向 View 输出数据，后端不参与PHP模板的开发工作。
3. 前端只维护 HTML ，但需要约定 AJAX 。第一次前端只交付HTML，后端将HTML『翻译』成模板。维护阶段前端也只维护 HTML，后端同事通过 SVN 或 GIT 查看代码修改记录找到 HTML 修改部分，将修改应用到模板引擎中。


如果选择前两种方案则务必记住以下两点：
1. **前后端在代码中只通过 JSON GET POST 沟通**
2. 以后端为主导事先约定数据交互文档（利用 [FMS](http://github.com/nimojs/fms) 生成文档和模拟数据）
