# 分享JS安全接口域名

# **网页授权**

> 如果用户在 Connect App 中访问第三方网页，可以通过 Connect 网页授权机制，来获取用户基本信息，进而实现业务逻辑。
> 

## **授权回调api地址**

api 地址可在开发者后台应用资料里设置，必须精确到授权方法，需精确到具体的授权回调方法,用户使用Connect账号登录后，只能回调至该链接。

参考示例: [https://example.com/callback](https://example.com/callback)

二级目录: [https://example.com/api/callback](https://example.com/api/callback)

# **开发指南**

网页授权流程分为四步：

1. 引导用户进入授权页面同意授权，获取code
2. 通过code换取网页授权access_token（与基础支持中的access_token不同）
3. 如果需要，开发者可以刷新网页授权access_token，避免过期,默认有效期24小时
4. 通过网页授权access_token和openid获取用户基本信息

![Untitled](%E5%88%86%E4%BA%ABJS%E5%AE%89%E5%85%A8%E6%8E%A5%E5%8F%A3%E5%9F%9F%E5%90%8D%2055e20c471da0404d8b35e20ba28ab191/Untitled.png)

### **一、获取code**

```bash
// 获取授权码
```

### **请求方式**

```html
https://open.nb-connect.cn/oauth/authorize?client_id=86OozHLhPl13NGYKJQVTpUkd&redirect_uri=http://test6666.com/callback&response_type=code
```

### **请求Query参数**

| 参数名 | 示例值 | 参数类型 | 是否必填 | 参数描述 |
| --- | --- | --- | --- | --- |
| client_id |  | String | 是 | appID |
| redirect_uri | http://test6666.com/callback | String | 是 | 授权回调URL |
| response_type |  | String | 是 | 授权类型 |

在确保该账号拥有授权的权限的前提下（已认证开发者账号，默认拥有权限），引导关注者打开如下页面：

```html
https://open.nb-connect.cn/oauth/authorize?client_id=appID&redirect_uri=redirect_uri&response_type=code
```

若提示“该链接无法访问”，请检查参数是否填写错误，是否拥有授权权限。

<aside>
💡 尤其注意：由于授权操作安全等级较高，所以在发起授权请求时，Connect会对授权链接做正则强匹配校验，如果链接的参数顺序不对，授权页面将无法正常访问

</aside>

<aside>
💡 尤其注意：跳转回调redirect_uri，Connect强制效验https链接来确保授权code的安全性。

</aside>

**用户同意授权后**

如果用户同意授权，页面将跳转至 redirect_uri/?code=CODE。

> code说明：
> 
> 
> code作为换取access_token的票据，每次用户授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期。
> 

## **二、通过code换取网页授权access_token**

<aside>
💡 首先注意：这里通过code换取的是一个特殊的网页授权access_token,本步骤中获取到网页授权access_token的同时，也获取到了openid，网页授权流程即到此为止。

</aside>

<aside>
💡 尤其注意：由于开发者账号的secret和获取到的access_token安全级别都非常高，必须只保存在服务器，不允许传给客户端。后续刷新access_token、通过access_token获取用户信息等步骤，也必须从服务器发起。

</aside>

### **请求方法**

获取code后，POST请求以下链接获取access_token：

```html

https://open.nb-connect.cn/oauth/token
```

| 参数 | 是否必须 | 说明 |
| --- | --- | --- |
| client_id | 是 | appId |
| client_secret | 是 | secret |
| code | 是 | 填写第一步获取的code参数 |
| grant_type | 是 | 填写为authorization_code |
| redirect_uri | 是 | 授权回调完整链接 |

### **返回示例**

正确时返回的JSON数据包如下：

```json
{
	"access_token": "eyJ0eXAiOiJKV1QiLCJhbGdqwdqdqRDiJ9.eyJhdWQiOiI0NU9vekhMaFBsOTNOR1lLSlFWVHBVa2QiLCJqdGkiOiJhODhjNTU2YzY2ZWQ5YmZhYmM0ODNkNjdiNmU0MzUxNDg4MzU1YzY3NWMzY2ViYzVjZmRjMTRhZDIyYTA1ZTc2NDNhOWY5OTUxYTllZmQxNSIsImlhdCI6MTcxODk4NDgxNiwibmJmIjoxNzE4OTg0ODE2LCJleHAiOjE3NTA1MjA4MTYsInN1YiI6IjExMTExMTExMSIsInNjb3BlcyI6W119.Sv7N7wldW8kqrooV5ROUjFm64KugbjpWb4mNuS7MU63WHf9ayyUR3Yw2x0su45jIJHc73Yg6zdB2yAEBC_h53JBx1lOUxstRgajhGQZQCYVXk_fvGZKiq8o_WEGwpSV3iB-Fj0SRKsFwg60YZm90ctAxf9AaWF4imHCjyVfSiv-ffVhfWkR6qiSj5Y4kmWCeT4L9n5Isxl1I5rAsR_re33Amt4_ceFulcgeZFcOqhZ5003TVnmVCAsNVyWrvJ5DZNGdIrfrUoGvEuPliJGibGJ-ZvN-0N432nC5gkA7UOUEpE9kaYTzJ3GfCD6ISjEsf9LEJxCWVxjNt9VugccGfdQ",
	"open_id": "ct_hjuOUfiQzWDesadotHpR4mbxKSLk6XyX23rQAwD5YFJ7NI1PecE9qz8sgVBM"
}
```

| 参数 | 描述 |
| --- | --- |
| access_token | 网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同 |
| openid | 用户唯一标识，用户和开发者应用唯一的OpenID |

错误时Connect会返回JSON数据包如下（示例为Code无效错误）:

```json
{
	"error": "invalid_request",
	"error_description": "The request is missing a required parameter, includes an invalid parameter value, includes a parameter more than once, or is otherwise malformed.",
	"hint": "Authorization code has been revoked",
	"message": "The request is missing a required parameter, includes an invalid parameter value, includes a parameter more than once, or is otherwise malformed."
}
```

## **三、拉取用户信息**

### **请求方法**

http：Post（请使用https协议）：

```html
https://open.nb-connect.cn/cgibin/connect/getUsers
```

| 参数 | 描述 |
| --- | --- |
| access_token | 网页授权接口调用凭证（需带在头部，参数名为Authorization，token拼接Bearer token） |
| open_id | 用户的唯一标识 |

### **Header 请求参数**

| 参数名 | 参数值 | 是否必填 | 参数类型 | 描述说明 |
| --- | --- | --- | --- | --- |
| Authorization | Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhqdQWdq6wOiI0NU9vekhMaFBsOTNOR1lLSlFWVHBVa2QiLCJqdGkiOiIxMTRjMDk4YWFiOTA5MDNmZjE2MDdhZmEyYmNmYzRhODViY2I2MGJmMzVkM2VjOTg0NzZjN2M5ZjczMjlmOTUzZGQyMjc0YWI3YWQ1MDUxMyIsImlhdCI6MTcxODk4Nzk2MywibmJmIjoxNzE4OTg3OTYzLCJleHAiOjE3MTkwNzQzNjMsInN1YiI6IjE4MDMzNDIxODExMDA3ODU2NjYiLCJzY29wZXMiOltdfQ.lezUejONGrVdsfToSYTr1jk5Zi9toMMU2lsZmMAyuKf07OjeSfNCPa01lJ-0ki4HZ2-CyHUaxwN8IMkpGBXEXcsqSKH-qotXzJNEU7OobCBjhKOdSO9VkPqY_J0hR-aGjrRmh_e0OAlmsZp4MBayl0Zf3TXOTvCODG2RFbRCQpcBNCcL_E0aVqdWsa67yrgmfUIgqurCFvHxg3Ji_7A6Iu3HAkNn7et-8Dt-WCgOYpSSpbN5vkA7OjsePgO1WDIF-c3gJQOxCko0pvjB0iqwq7pnJzA11aM4d9WiGCu2_E_LZcvo3Wbmq5rA4vHoBKDOBnYeZBiCZxeub03Iob5StQ | 是 | String |  |

### **Body 请求参数**

| 参数名 | 参数值 | 是否必填 | 参数类型 | 描述说明 |
| --- | --- | --- | --- | --- |
| open_id | ct_vAkPQ0bdIM8KX2UTgjz5GtExaO1nlZiCrDL6wp4VW7J9m31BSsFoNYhyHcqu | 是 | String |  |

**返回说明**

正确时返回的JSON数据包如下：

```json
{
	"code": 1,
	"msg": "success",
	"time": "1718987988",
	"data": {
		"userName": "Tommy",
		"avatar": "http://connect-oss.oss-cn-hangzhou.aliyuncs.com/upload/20240619/464cc01ab148f2546b4dc4611fee10c9.png"
	}
}
```

| 参数 | 描述 |
| --- | --- |
| code | 状态码 |
| msg | 描述 |
| time | 时间戳 |
| data.userName | 用户昵称 |
| data.avatar | 用户头像 |

错误时Connect会返回JSON数据包如下（示例为openid无效）:

```json
{
	"code": 0,
	"msg": "Currently, no users  has been found. Please check if the openID is correct",
	"time": "1718988217",
	"data": null
}
```
