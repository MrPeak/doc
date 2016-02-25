# 图片上传接口约定

文件上传一律使用 `/Upload/` 或者 `/Upload.php`，默认返回数据格式如下：

```javascript
{
    "status": "success",
    "id": "3157137",
    // 后续表单提交时候所需要的文件 ID 标示
    "src": "http://www.domain.com/xxx/demo.jpg",
    // 文件的在线访问或者图片预览地址
    "filename": "demo.jpg",
    // 文件名
    "byte": 2341,
    // 文件大小
    "width": 200,
    "height": 200
    // 如果是图片会返回图片宽高尺寸
}
// 失败时返回
{
    "status": "error",
    "msg": "详细错误原因"
}
```

文件上传域的 name 默认为 `file`

不同类型的资源上传使用 `GET type`参数做区分。例如：

`/uploadify/?type=photo`
`/uploadify/?type=file`

```php
switch ($_GET['type']) {
    case "img":
    // 保存图片并返回信息
    break;
    case "file":
    // 保存文件并返回信息
    break;
    default:
    // 默认保存图片并返回信息
}
```
