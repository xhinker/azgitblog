# 使用JSONP进行跨域Ajax 调用

## JSONP 是啥

JSONP 全称是JSON with Padding. 当需要进行跨域Ajax 调用的时候, 需要用到JSONP 协议. 

## 客户端

```javascript
$.ajax({
    url: 'http://xxx',
    type: "Get",
    data: { 
        user_name:user_name,
        password:password
    },
    dataType: "jsonp",
    success:function(data){
        console.log('send ok');
    },
    error:function(xhr, status, error){
        console.log(status + '; ' + error);
    }
});
```

## 服务端

下面用Nodejs 举例. 一个jsonp 请求来的时候, 服务端接收到url 大致是这样的: 
>/verify?callback=jQuery33108773940957973894_1519815876941&user_name=user4&password=1234&_=1519815876942

服务端需要做的就是,把callback 部分提取取出来,然后以下面这种方式返回

```javascript
res.end(query_data.callback+'('+ JSON.stringify(result_json) + ')');
```

## 相关链接
<http://www.cnblogs.com/lengyuhong/archive/2012/03/20/2370688.html>
<https://en.wikipedia.org/wiki/JSONP>
