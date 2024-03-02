# 开发例程

## 整体流程
1. 获取输入参数，命令不正确则输入提示信息，并退出；
1. 将第二个参数作为 C 文件的路径，并初始化 `Prompt`；
1. 打开文件，成功则将读出的内容追加到 `Prompt` 中，错误则输出错误信息（未退出，可以直接退出）；
1. 创建 GLM 实例，将 `API key` 和 `Prompt` 发送给开放平台，并等待回复；
1. 最后输出平台返回的内容。

## 与 AI 平台的交互

1. 根据 `API key` 创建 `JWT`；
1. 将 `JWT` 和 `Prompt` 发送给开放平台，并等待回复；

## API Key
1. 智谱 AI 开放平台的 `API key`，同时包含 “用户标识 id” 和 “签名密钥 secret”，即格式为 `{id}.{secret}`；
2. 用户 ID 用来标识用户，密钥用来加密鉴权。

## JWT
详细内容见智谱AI官网说明文档 <https://maas.aminer.cn/dev/api#nosdk>

1. Json Web Token，用于网络传输中数据鉴权、防篡改；
1. 这里的 JWT 是平台自定义类型，与标准JWT 有一点区别；
1. JWT 由三部分组成：头部、负载、签名，字符串值格式为：`{header}.{payload}.{signature}`，其中 `header`、`payload`和`signature`，都是经过 `Base64URL` 编码后的字符串。

### JWT Header
```json
{
    "alg": "HS256",
    "sign-type": "SIGN"
}
```
`alg`：表示签名使用的算法，默认为 HMAC SHA256（写为HS256）；
`sign_type`：表示令牌的类型，JWT 令牌统一写为 SIGN 。

### JWT Payload
```json
{
    "api_key": "{ApiKey.id}",
    "exp": 1682503829130,
    "timestamp":1682503820130
}
```
`api_key`：表示用户标识 id，即用户API Key的 `{id}` 部分；
`exp`：表示生成的JWT的过期时间，客户端控制，单位为毫秒；
`timestamp`：表示当前时间戳，单位为毫秒。

### JWT Signature
要创建签名部分，必须获取头部、负载、密钥，使用头部中的指定算法进行签名。

例如，如果使用 HMAC SHA256 算法，将按以下方式创建签名：
```
HMACSHA256(
    base64UrlEncode(JWT.header) + "." + base64UrlEncode(JWT.payload),
    ApiKey.id
)
```

## 开放平台 http 接口
1. URL：`https://open.bigmodel.cn/api/paas/v4/chat/completions`
1. 对该 URL 发起 `post` 请求，详细内容见官方文档 <https://maas.aminer.cn/dev/api#glm-4>

    以下为示例代码中的内容，`head` 内容如下
    ```
    "Cache-Control":"no-cache"
    "Connection", "keep-alive"
    "Accept", "text/event-stream"
    "Content-Type", "application/json;charset=UTF-8"
    "Authorization", "Bearer {token}"
    ```
    其中 `token` 就是 JWT 的 base64URL 字符串值。

    `body` 内容如下：
    ```json
    {
        "model":"{val}",
        "messages":[
            {
                "role":"system",
                "content":"{SYSTEM_CONTENT}"
            }
            {
                "role":"user",
                "content":"{prompt}"
            }
        ],
        "stream":"{val}",
        "do_sample":"{val}",
        "temperature":"{val}",
        "top_p":"{val}"
    }
    ```
    其中：

    `prompt`即 C 语言代码内容，即输入的 C 文件内容；

    `SYSTEM_CONTENT`：是提前写好的身份描述信息，让 AI 模型扮演 c/c++ 和 rust 专家。
