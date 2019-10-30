# 第三方系统接入

校验日期：

---

## 使用场景

* 外部系统需要访问petrel体系内服务。

---

## 接入流程

1. 向管理员申请固定开发者ID，开发者密码。
2. 根据开发者ID和开发者密码，获取访问令牌：

   ```
   GET https://dev-petrel.belle.net.cn/petrel/cgi-bin/create/token?appId={开发者ID}&appSecret={开发者密码}
   ```

   ```
   RETURN
   {
       "data":{
           "token":"eyJjdHkiOiJKV1QiLCJlbmMiOiJBMjU2R0NNIiwiYWxnIjoiZGlyIn0..kAm3tgg9tDc7FT6M.Fol4T5SWb3kNHol7BwHxxhkV6NKAhjl-6vji4whvtKrcJZC1yiyo8Kz82uGuoIyCt3987RvrfF41hF56vg4Qp4ydS66d5JD9TlzFcES4RKuQ2O3tcfHSl50HCvDvn0ioXxJ9Eb9zZuIt-UXTNUE7-Ab-LH7MOt_iOGeeCOg41StSgo1bTn5VtctJuGBR5RbpbAxTilxwwUqUFe3OhTmVQTPXGak5BpQJF0uke4yKTZbIKn07Y3PPAv_BXvnfPt3mw4oxFH43kdZPibNfBbIQqaV8ADXWXMI-EV2EKAv0apXnYQW2wBRXbKsrJm4WmT85pMSCPQt-NLAqX4Xb_muQFyH9w67KPJF2AjGRxOv4CRudx8TdOxZKvUg3fALmytJgibrUHZWpHbBCYLD1hqtMqQwPDuhbLFYQS6mIAq44pzCENnK3nHhX2G44_qBZkcUc4CS91vMFI0-A0erLAn7Evxm67gyfs6qJFFMEeVyf7NxeuGuU5SVbI2tVktKb86-0dSflt2UqGJZ0lgHrd_KITzpQJtskC47dp9d1SQFg8eiw4Zpj7J-e10u-XvbtgSx8xIhi09093Ov0RoLbgEhwUjT1wHglI20MUq7ZefpJ-n7YnW85b0jk57UdTSCtU5Z2qWSbUfwtXyY.p2_nyE1QB3T2UqSYk2MxjQ",
           "expire":"7200"
       },
       "flag":{
           "retCode":"0",
           "retMsg":"success",
           "retDetail":null
       }
   }
   ```

3. 带上令牌访问体系内服务：  
   `https://dev-petrel.belle.net.cn/petrel/petrel-uc-api/sysUserApi/page?token={接口返回的token}`

> 接入者通过接口申请token,有效期为7200秒
>
> 每个接口调用都需要带上token，可在http的header或者url?token=令牌

---

## 说明

* 安全认证从两方面保证系统安全

  * 系统角度

    * 对每个接入者进行身份安全认证，不合法者直接拒绝访问

    * 对每个接入者访问资源进行控制，防止权限外溢

  * 协议角度

    * 采用https协议，防止数据传输被窃取，仿冒接入偷取令牌

* 系统角度解决方案

  * 后端系统增加网关服务，外部系统访问经过网关认证，认证成功后才允许访问内部系统接口

  * 内部接口应用不允许直接暴露外网访问，需经过网关服务

  * 第三方接入者申请开发者ID、授于访问权限

* 说明

  * 网关服务：拦截外部请求，保护内部服务安全，认证授权，只允许合法者访问内部接口

  * 外部访问内部系统，都必须带上AccessToken令牌

* Petrel平台维护第三方系统

  * 维护接入者基本信息\(提供信息，由基础应用组维护\)

  * 配置外部接入者访问资源权限

* 用户操作流程

  * 接入者申请固定开发者ID，开发者密码

  * 接入者通过接口申请token,有效期为7200秒

  * 每个接口调用带上token，可在http的header或者url?token=令牌



