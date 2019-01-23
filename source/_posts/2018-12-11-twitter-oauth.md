---
title: twitter接入之oauth验证
date: 2018-12-11 09:29:17
tags:
- oauth
- twitter
---

### 详细步骤说明

1. 获取consumer_key和consumer_key_secret。
    - 申请twitter开发者账号
    - 新建应用即可拿到

2. 请求oauth/request_token
该请求作为验证第一步，主要验证权限和获取权限token。请求没有body参数，只需将验证所需的参数按顺序放入Authorization头即可。实例如下：

{% asset_img twitter_oauth_1.png %}

+ oauth_callback：回调地址，一般是应用的主页
+ oauth_consumer_key：开发者账号获得
+ oauth_nonce：随机32位字符串，必须进行urlencode。
+ oauth_signature：签名(这个很重要，签名方法看最后)
+ oauth_signature_method：签名的加密方式，固定为HMAC-SHA1必须为大写
+ oauth_timestamp：请求时时间戳
+ oauth_version：oauth版本，twitter使用的是1.0版本

请求Response实例：

{% asset_img oauth_res_1.png %}

注：
+ oauth_nonce和oauth_timestamp每次请求时都必须重新生成且唯一。
+ 请求返回的oauth_callback_confirmed必须为true才代表验证通过

<!-- more -->

3. 重定向到用户授权页面。
前端将上述获取的oauth_token作为参数重定向页面到：
https://api.twitter.com/oauth/authenticate?oauth_token=<上一步获取的值>

4. 用户进行授权
用户授权成功后，会自动跳转回最初设定的回调地址，并带回两个参数，如下所示：

{% asset_img oauth_res_2.png %}

5. 获取access_token
将上述的两个参数加入验证头，并请求oauth/access_token获取最终的token和secret

{% asset_img twitter_oauth_2.png %}

请求Response实例：

{% asset_img oauth_res_3.png %}

6. 使用最终获取的token和secret进行其他接口请求

### 签名方法

1. 获取请求方式：POST或者GET

2. 获取请求地址, 必须为urlencode之后的数值，如：https%3A%2F%2Fapi.twitter.com%2Foauth%2Faccess_token

3. 生成加密字符串
    - 获取除oauth_signature之外的所有参数字典，并进行按字母顺序排序。将该参数字典进行urlencode。urlencode之后再对该值进行一次url编码（python2的方式是urlib.quote，python3中是urlib.parse.quote）
    - 拼接加密字符串，格式为<请求方式>&<请求地址>&<请求参数>，比如安装上方的例子就是：POST& https%3A%2F%2Fapi.twitter.com%2Foauth%2Faccess_token&<生成的字符串数据>

4. 生成加密key，格式为：consumer_secret&token_secret

+ 第一次请求/request_token时，token_secret是空字符串，所以key就为: consumer_secret&（注意&还是要写进去）

+ 之后请求获取到oauth_token_secret值就为：consumer_secret&oauth_token_secret。Oauth_token_secret始终使用最后获取到的值。
5. 使用sha1进行加密，注意最终还是要再进行两次encode，如下

{% asset_img signature.png %}

6. 将签名放入请求头的参数字典。

![avatar](https://phinf.pstatic.net/contact/20190108_71/1546937254419EXRI3_JPEG/image.JPEG)
