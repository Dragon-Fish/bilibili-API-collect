# 二维码登录

流程&逻辑：

1. 获取「二维码url」以及「秘钥」，以「二维码url」作为内容生成二维码，等待手机端扫描
2. 以「秘钥」作为参数进行POST
3. if "code"==true goto 6                   else goto 4（是否已经确认）
4. if "data"==-4   goto 2                   else goto 5（是否已经扫描）
5. if "data"==-5   goto 3 && 提示「已扫描」 else goto 1&提示二维码超时或错误（秘钥是否有效）
6. 

## 获取二维码内容url以及秘钥 (秘钥超时为180秒)
passport.bilibili.com/qrcode/getLoginUrl

*方式:GET*


**json回复**
| 字段    | 类型  | 内容      | 备注               |
| ------- | ----- | --------- | ------------------ |
| code    | num   | 返回值    | 0成功              |
| status  | bool  | true      | 作用尚不明确       |
| ts      | num   | 请求时间  | 时间戳             |
| data    | obj   | 信息本体  |                    |

data 对象：
| 字段     | 类型  | 内容          | 备注       |
| -------- | ----- | ------------- | ---------- |
| url      | str   | 二维码内容url | 恒为87字符 |
| oauthKey | str   | 扫码登录秘钥  | 恒为32字符 |

示例：

http://passport.bilibili.com/qrcode/getLoginUrl
```
{
	"code": 0,
	"status": true,
	"ts": 1583314311,
	"data": {
		"url": "https://passport.bilibili.com/qrcode/h5/login?oauthKey=c3bd5286a2b40a822f5f60e9bf3f602e",
		"oauthKey": "c3bd5286a2b40a822f5f60e9bf3f602e"
	}
}
```

## 验证二维码登录
passport.bilibili.com/qrcode/getLoginInfo

正确时会进行设置以下cookie项：
DedeUserID DedeUserID__ckMd5 SESSDATA bili_jct


*方式:POST*

参数：
| 参数名   | 内容         | 必要性 | 备注                          |
| ---------| ------------ | ------ |------------------------------ |
| oauthKey | 扫码登录秘钥 | 必要   |                               |
| gourl    | 跳转url      | 非必要 | 默认为http://www.bilibili.com |


**json回复**
| 字段    | 类型                | 内容                                 | 备注                                            |
| ------- | ------------------- | ------------------------------------ | ----------------------------------------------- |
| status  | bool                | 扫码是否成功                         |                                                 |
| code    | num                 | 返回值                               | 0成功                                           |
| message | str                 | 错误信息                             | 正确无                                          |
| ts      | num                 | 扫码时间                             | 错误无                                          |
| data    | 正确时obj 错误时num | 正确时：游戏分站url 错误时：错误代码 | 错误时：-1秘钥错误 -2秘钥超时 -4未扫描 -5未确认 |

data 对象：
| 字段 | 类型 | 内容            | 备注 |
| ---- | ---- | --------------- | ---- |
| url  | str  | 游戏分站登录url |      |

示例：

curl -d "oauthKey=xxx" "http://passport.bilibili.com/qrcode/getLoginInfo"
```
{
	"code": 0,
	"status": true,
	"ts": 1583315474,
	"data": {
		"url": "https://passport.biligame.com/crossDomain?DedeUserID=293793435&DedeUserID__ckMd5=ab24cb40578a85b0&Expires=15551000&SESSDATA=7816ef30%2C1598867474%2C500ce%2A31&bili_jct=cf13274801a8ff69bdbc4a1c0f09688a&gourl=http%3A%2F%2Fwww.bilibili.com"
	}
}
```

抓包信息（回复头部）：

可明显看见设置了几个cookie（本人手打cookie，成功登录B站）（重要token已河蟹处理）

```
HTTP/1.1 200 OK
Date: Wed, 04 Mar 2020 10:36:37 GMT
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Server: Apache-Coyote/1.1
Set-Cookie: sid=***; Domain=.bilibili.com; Expires=Thu, 04-Mar-2021 10:36:37 GMT; Path=/
Set-Cookie: DedeUserID=***; Domain=.bilibili.com; Expires=Mon, 31-Aug-2020 10:19:57 GMT; Path=/
Set-Cookie: DedeUserID__ckMd5=***; Domain=.bilibili.com; Expires=Mon, 31-Aug-2020 10:19:57 GMT; Path=/
Set-Cookie: SESSDATA=***; Domain=.bilibili.com; Expires=Mon, 31-Aug-2020 10:19:57 GMT; Path=/; HttpOnly
Set-Cookie: bili_jct=***; Domain=.bilibili.com; Expires=Mon, 31-Aug-2020 10:19:57 GMT; Path=/
Expires: Wed, 04 Mar 2020 10:36:36 GMT
Cache-Control: no-cache
X-Cache-Webcdn: BYPASS from ks-sxhz-dx-w-01
```

**游戏分站登录url**
https://passport.biligame.com/crossDomain?
DedeUserID=(登录UID)&
DedeUserID__ckMd5=(DedeUserID__ckMd5)&
Expires=(过期时间 秒)&
SESSDATA=(SESSDATA)&
bili_jct=(bili_jct)&
gourl=(跳转网址 默认为主页)

