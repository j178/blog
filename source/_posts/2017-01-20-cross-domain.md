---
 title: 跨域相关
 date: 2017-01-20 23:55:43
 tags:  [同源策略, 跨域]
 categories: 前端
---

##  同源策略

origin = 协议 + 域名 + 端口号

同源策略的限制：

1. Cookie, LocalStorage 和 IndexDB 无法读取
2. DOM 无法获得
3. AJAX 无法发送

可跨域的标签 script，img，iframe, link等

JSONP 实质是一种自己发起的XSS

无法在 https 页面发起对非https的请求，会出现 Mixed content 错误

## CORS方案

发起跨域请求时，浏览器会自动设置`Origin`首部，无法手动改变
在服务端的响应中设置`Access-Control-Allow-Origin`首部
但是这个首部不支持多个 host，所以如果需要支持多个 domain，需要动态地生成这个首部。比如说：

```php
< ?php
$allowedOrigins = array("http://example.com", "http://localhost:63342");
$headers = getallheaders();
if ($headers["Origin"] && in_array($headers["Origin"], $allowedOrigins)) {
    header("Access-Control-Allow-Origin: " . $headers["Origin"]);
}
?>
```
### 是否发送凭证

客户端：

```javascript
      var xhr = new XMLHttpRequest();
      xhr.withCredentials = true;
```
服务端添加 `Access-Control-Allow-Credentials : true`首部，如果服务器端没有返回这个首部，即使当前domain是允许的，客户端也得不到响应。会有类似 *XMLHttpRequest cannot load http://example.com/index.php. Credentials flag is ‘true’, but the ‘Access-Control-Allow-Credentials’ header is ”. It must be ‘true’ to allow credentials. Origin ‘http://localhost:63342‘ is therefore not allowed access.* 的提示。

当开启 withCredential 属性时，服务器返回的`Access-Control-Allow-Origin`不允许为`*`

### Preflight

当跨域请求不是简单请求时，浏览器会先发送一个预检请求(preflight)

浏览器先通过OPTIONS方法询问 request的方法和首部是否支持:

`Access-Control-Request-Method`

`Access-Control-Request-Headers`

如果服务端允许跨域请求，则返回几个特殊的首部：

`Access-Control-Allow-Methods`:必需。返回所有支持的方法，而不单是浏览器请求的那个方法，这是为了避免多次预检请求。

`Access-Control-Allow-Headers`：如果请求中有`Access-Control-Request-Headers`，则必需。返回所有支持的首部信息。

`Access-Control-Allow-Credentials`

`Access-Control-Allow-Max-Age`:可选，指定本次预检的有效期，单位为秒。

一旦服务器通过了预检，以后每次浏览器正常的CORS请求都跟简单请求一样，包含一个`Origin`首部，服务器的相应也会有一个`Access-Control-Allow-Origin`。

### `crossorigin`属性

通常 `<link> <script> <img>`这些本身就可以跨域的标签，在请求时不会发送`Origin`首部，所以我们也无法使用 JS 获取他们的内容(会报错)。如果为这些标签设置`crossorigin`属性，则请求时就会使用`CORS`请求，并且设置了`Origin`首部。

`crossorigin`有两个可能的值：`anonymouse`：不会发送credentials，`use-credentials`：发送credentials



# 参考链接

1. [浏览器同源政策及其规避方法](http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html)
2. [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)

