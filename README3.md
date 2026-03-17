# 全局公共参数

**全局Header参数**

| 参数名 | 示例值 | 参数类型 | 是否必填 | 参数描述 |
| --- | --- | ---- | ---- | ---- |
| X-API-KEY | bbbb | string | 是 | APIKEY |
| X-SIGN | aaaa | string | 是 | 签名 |


# 浏览器相关接口


### 签名与验签

#### 请求参数签名

##### Header请求参数

| 请求头参数名 | 描述说明 |
| --- | --- |
| X-API-KEY | APIKEY |
| X-SIGN | 参数签名 |

** 筛选
获取所有请求参数，不包括字节类型参数，如⽂件、字节流，剔除sign与sign_type参数。

** 排序
将筛选的参数按照第⼀个字符的键值ASCII码递增排序（字⺟升序排序），如果遇到相同字符则按照第⼆个字符的键值ASCII码递增排序，以此类推。

** 拼接
将排序后的参数与其对应值，组合成“参数=参数值”的格式，并且把这些参数⽤&字符连接起来，此时⽣
成的字符串为待签名字符串。MD5签名的商户需要将key的值拼接在字符串后⾯，调⽤MD5算法⽣成sign


##### 返回参数验证签名
** 筛选
获取所有请求参数，不包括字节类型参数，如⽂件、字节流，剔除sign与sign_type参数。

** 排序
将筛选的参数按照第⼀个字符的键值ASCII码递增排序（字⺟升序排序），如果遇到相同字符则按照第⼆个字符的键值ASCII码递增排序，以此类推。

** 拼接
将排序后的参数与其对应值，组合成“参数=参数值”的格式，并且把这些参数⽤&字符连接起来，此时⽣
成的字符串为待签名字符串。MD5签名的商户需要将key的值拼接在字符串后⾯，调⽤MD5算法⽣成sign

#### 签名参考代码

```java
import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.*;

import com.alibaba.fastjson2.JSON;

public class SignGenerator {

    /**
     * 生成签名
     * @param params 请求参数
     * @param key 商户密钥
     * @return 生成的签名
     */
    public static String generateSign(Map<String, Object> params, String key) {
        // 1. 筛选参数
        Map<String, Object> filteredParams = filterParams(params);

        // 2. 排序参数
        SortedMap<String, Object> sortedParams = new TreeMap<>(filteredParams);

        // 3. 拼接参数
        String signString = buildSignString(sortedParams);

        // 4. 拼接 key 并生成 MD5 签名
        return md5(signString + key);
    }

    /**
     * 筛选参数，排除字节类型参数、sign 与 sign_type 参数
     * @param params 原始参数
     * @return 筛选后的参数
     */
    private static Map<String, Object> filterParams(Map<String, Object> params) {
        Map<String, Object> filteredParams = new HashMap<>();
        for (Map.Entry<String, Object> entry : params.entrySet()) {
            String key = entry.getKey();
            if (!"sign".equals(key) && !"sign_type".equals(key)) {
                // 这里简单认为非字节类型参数，实际可根据具体情况调整
                filteredParams.put(key, entry.getValue());
            }
        }
        return filteredParams;
    }

    /**
     * 构建待签名字符串
     * @param sortedParams 排序后的参数
     * @return 待签名字符串
     */
    private static String buildSignString(SortedMap<String, Object> sortedParams) {
        StringBuilder sb = new StringBuilder();
        for (Map.Entry<String, Object> entry : sortedParams.entrySet()) {
            String key = entry.getKey();
            String value = convertToString(entry.getValue());
            if (value != null && !"".equals(value.trim())) {
                if (sb.length() > 0) {
                    sb.append("&");
                }
                sb.append(key).append("=").append(value);
            }
        }
        return sb.toString();
    }

    private static String convertToString(Object value) {
        if (value == null) {
            return "";
        }
        if (value instanceof String) {
            return (String) value;
        } else if (value instanceof Number) {
            return value.toString();
        } else if (value instanceof Boolean) {
            return Boolean.toString((Boolean) value);
        } else {
            return JSON.toJSONString(value);
        }
    }

    /**
     * 生成 MD5 签名
     * @param input 输入字符串
     * @return MD5 签名结果
     */
    private static String md5(String input) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] messageDigest = md.digest(input.getBytes("UTF-8"));
            StringBuilder hexString = new StringBuilder();
            for (byte b : messageDigest) {
                String hex = Integer.toHexString(0xFF & b);
                if (hex.length() == 1) {
                    hexString.append('0');
                }
                hexString.append(hex);
            }
            return hexString.toString().toUpperCase();
        } catch (NoSuchAlgorithmException | UnsupportedEncodingException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        Map<String, Object> params = new HashMap<>();
        params.put("merchantOrderNo", "111");
        params.put("amount", "20");
        params.put("remark", "test is test");
        params.put("sign", "old_sign");
        params.put("sign_type", "MD5");
        String key = "-OWQz0xOqtRtJTxbn5UzhQ3W4aMANY9mZFvRC2z6pNX2FcyVXkNsARsyfchLipB7";

        String sign = generateSign(params, key);
        System.out.println("Generated Sign: " + sign);
    }
}
```
```
a=a&b=b{apiKey}
```


## 浏览器列表

**接口URL**

> /api/open/browser/list

**请求方式**

> POST

**Content-Type**

> json

**请求Body参数**

```javascript
{
    "pageSize": 20,
    "page": 1
}
```


**响应示例**

* 成功(200)

```javascript
{
	"code": 1,
	"message": "success",
	"data": {
		"pageNum": 1,
		"pageSize": 20,
		"total": 1,
		"pages": 1,
		"data": [
			{
				"_id": "69b26cd5d69183445512430d",
				"userID": "admin",
				"serverId": "69b22c516f0cf46cb5b6df2b",
				"serverName": "ubuntu205",
				"dockerId": "e624580d3066",
				"deviceId": "node-docker-cli-ubuntu205-192.168.10.205",
				"ipAddress": "192.168.10.205",
				"port": "13946",
				"name": "node-docker-cli-ubuntu205-192.168.10.205brms1-browser-yuntu-13946",
				"useUserID": "zhangsan",
				"type": "yuntu",
				"image": "registry.tnt-pub.com/nyy/nyy/reddit-sandbox:latest",
				"status": "used",
				"status2": "online",
				"jumpServerId": "14a93f24-4d1a-4592-a9c0-89f1606d75f7",
				"vncPort": "15762",
				"createTime": "2026-03-12T07:35:49.351Z",
				"updateTime": "2026-03-12T07:35:51.258Z",
				"usedTime": "2026-03-12T07:45:38.128Z"
			}
		]
	}
}
```

* 失败(404)

```javascript
暂无数据
```



## 浏览器创建

**接口URL**

> /api/open/browser/create

**请求方式**

> POST

**Content-Type**

> json

**请求Body参数**

```javascript
{
    "type": "yuntu", // 浏览器类型 yuntu或browserless
    "proxyUrl": "" // yuntu的代理
}
```


**响应示例**

* 成功(200)

```javascript
{
	"code": 1,
	"message": "success",
	"data": {
		"_id": "69b26cd5d69183445512430d",
		"userID": "admin",
		"serverId": "69b22c516f0cf46cb5b6df2b",
		"serverName": "ubuntu205",
		"dockerId": "e624580d3066",
		"deviceId": "node-docker-cli-ubuntu205-192.168.10.205",
		"ipAddress": "192.168.10.205",
		"port": "13946",
		"name": "node-docker-cli-ubuntu205-192.168.10.205brms1-browser-yuntu-13946",
		"useUserID": "zhangsan",
		"type": "yuntu",
		"image": "registry.tnt-pub.com/nyy/nyy/reddit-sandbox:latest",
		"status": "used",
		"status2": "online",
		"jumpServerId": "14a93f24-4d1a-4592-a9c0-89f1606d75f7",
		"vncPort": "15762",
		"createTime": "2026-03-12T07:35:49.351Z",
		"updateTime": "2026-03-12T07:35:51.258Z",
		"usedTime": "2026-03-12T07:45:38.128Z"
	}
}
```

* 失败(404)

```javascript
暂无数据
```


## 浏览器释放

**接口URL**

> /api/open/browser/release/:id

**请求方式**

> POST

**Content-Type**

> json

**请求Body参数**

```javascript
{
}
```


**响应示例**

* 成功(200)

```javascript
{
	"code": 1,
	"message": "success"
}
```

* 失败(404)

```javascript
暂无数据
```


## 浏览器代理列表

**接口URL**

> /api/open/proxy/list

**请求方式**

> POST

**Content-Type**

> json

**请求Body参数**

```javascript
{
    "pageSize": 20,
    "page": 1
}
```


**响应示例**

* 成功(200)

```javascript
{
	"code": 1,
	"message": "success",
	"data": {
		"pageNum": 1,
		"pageSize": 20,
		"total": 1,
		"pages": 1,
		"data": [
			{
				"_id": "69b2734815db702b5390af81",
				"userID": "zhangsan",
				"serverId": "69b22c516f0cf46cb5b6df2b",
				"serverName": "ubuntu205",
				"dockerId": "cb243cf9e1e1",
				"port": "23032",
				"name": "node-docker-cli-ubuntu205-192.168.10.205brms1-browser-yuntu-14420_proxy",
				"useUserID": "zhangsan",
				"type": "gost",
				"image": "ginuerzh/gost:latest",
				"status": "used",
				"browserId": "69b26cddd691834455124310",
				"browserName": "node-docker-cli-ubuntu205-192.168.10.205brms1-browser-yuntu-14420",
				"proxyUrl": "socks5://a:b@12.233.138.199:1240",
				"createTime": "2026-03-12T08:03:20.397Z",
				"updateTime": "2026-03-12T08:03:20.397Z"
			}
		]
	}
}
```

* 失败(404)

```javascript
暂无数据
```



## 浏览器代理创建

**接口URL**

> /api/open/proxy/create

**请求方式**

> POST

**Content-Type**

> json

**请求Body参数**

```javascript
{
    "browserId":"69b26cd5d69183445512430d",
    "proxyUrl": "socks5://a:b@12.233.138.199:1240",
    "imageType": "gost"
}
```


**响应示例**

* 成功(200)

```javascript
{
	"code": 1,
	"message": "success",
	"data": {
		"_id": "69b2734815db702b5390af81",
		"userID": "zhangsan",
		"serverId": "69b22c516f0cf46cb5b6df2b",
		"serverName": "ubuntu205",
		"dockerId": "cb243cf9e1e1",
		"port": "23032",
		"name": "node-docker-cli-ubuntu205-192.168.10.205brms1-browser-yuntu-14420_proxy",
		"useUserID": "zhangsan",
		"type": "gost",
		"image": "ginuerzh/gost:latest",
		"status": "used",
		"browserId": "69b26cddd691834455124310",
		"browserName": "node-docker-cli-ubuntu205-192.168.10.205brms1-browser-yuntu-14420",
		"proxyUrl": "socks5://a:b@12.233.138.199:1240",
		"createTime": "2026-03-12T08:03:20.397Z",
		"updateTime": "2026-03-12T08:03:20.397Z"
	}
}
```

* 失败(404)

```javascript
暂无数据
```


## 浏览器代理释放

**接口URL**

> /api/open/proxy/release/:id

**请求方式**

> POST

**Content-Type**

> json

**请求Body参数**

```javascript
{
}
```


**响应示例**

* 成功(200)

```javascript
{
	"code": 1,
	"message": "success"
}
```

* 失败(404)

```javascript
暂无数据
```



## 创建子用户

**接口URL**

> /api/open/createUser

**请求方式**

> POST

**Content-Type**

> json

**请求Body参数**

```javascript
{
    "userID":"wangwu"
}
```


**响应示例**

* 成功(200)

```javascript
{
	"code": 1,
	"message": "success"
}
```

* 失败(404)

```javascript
暂无数据
```

## 授权子用户打开浏览器

**接口URL**

> /api/open/getOpenBrowserUrl

**请求方式**

> POST

**Content-Type**

> json

**请求Body参数**

```javascript
{
    "userID":"wangwu",
    "browserId": "69b3fd297f4c577903cc652d"
}
```


**响应示例**

* 成功(200)

```javascript
{
	"code": 1,
	"message": "success",
	"data": "?ticketId=23254759-d00f-4b5b-9247-f88b2cc333aa&userID=wangwu" // 拼接 https://xx.com/jump?ticketId=23254759-d00f-4b5b-9247-f88b2cc333aa&userID=wangwu
}
```

* 失败(404)

```javascript
暂无数据
```

## 登出子用户

**接口URL**

> /api/open/logout

**请求方式**

> POST

**Content-Type**

> json

**请求Body参数**

```javascript
{
    "userID":"wangwu"
}
```


**响应示例**

* 成功(200)

```javascript
{
	"code": 1,
	"message": "success"
}
```

* 失败(404)

```javascript
暂无数据
```
