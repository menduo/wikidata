---
categories: weibo,oauth,repost
...

    本文为转载，略加排版和修改。

[新浪微博开放平台](http://open.weibo.com/wiki/API%E6%96%87%E6%A1%A3_V2)提供了丰富的API接口，利用这些接口，开发者能够开发出独具特色的微博应用。但是，大部分接口都需要用户授权给应用，应用利用授权得到的Access Token来调用相应的接口来获取内容。

新浪微博的授权机制目前主要有3种应用场景：

1.  Web应用
2.  移动应用
3.  站内应用

本文主要介绍 Web 应用如何授权、获取 Access Token。

## **步骤一：添加网站**

进入[新浪微博开放平台](http://open.weibo.com)，进入“管理中心“，点击”创建应用”，选择“网页应用”，填写相应的信息后提交。

## **步骤二：Oauth2.0授权设置**

应用创建完后可以在“管理中心”-“我的应用”中查看信息，在“应用信息”--“高级信息”中可以设置网站的`授权回调页`和`取消授权回调页`。

授权回调页非常重要，一定要填写正确，当用户授权成功后会回调到此页面，传回一个 `code` 参数，开发者可以用 code 换取 `Access Token` 值。

## **步骤三：引导用户授权**

引导需要授权的用户到如下页面：

    https://api.weibo.com/oauth2/authorize?client_id=**YOUR_CLIENT_ID**&response_type=code&redirect_uri=**YOUR_REGISTERED_REDIRECT_URI**

* `YOUR_CLIENT_ID` :即应用的AppKey，可以在应用基本信息里查看到。
* `YOUR_REGISTERED_REDIRECT_URI` :即之前填写的授权回调页，注意一定要完全相同。

__如果用户授权成功后，会跳转到回调页，开发者此时需要得到url参数中的code值，注意code只能使用一次。__

## **步骤四：换取Access Token**

开发者可以访问如下页面得到 Access Token：

    https://api.weibo.com/oauth2/access_token?client_id=**YOUR_CLIENT_ID**&client_secret=**YOUR_CLIENT_SECRET**&grant_type=authorization_code&redirect_uri=**YOUR_REGISTERED_REDIRECT_URI**&code=**CODE**

这些参数就不一一介绍了。

如果都没有问题，就可以得到 Access Token 了，返回示例：

```json
{
   "access_token": "ACCESS_TOKEN"，
   "expires_in": 1234，
   "remind_in": "798114"，
   "uid": "12341234"
 }
```

开发者可以通过两种方式计算access_token的实效时间：

1. 用户授权时，oauth2/access_token接口返回的expires_in值就是access_token的生命周期；
2. 从上述对应表中，找到应用所对应的授权有效期，过期时间 = 用户授权时间 + 授权有效期；

## **步骤五：调用API**

获取到Access Token后，开发者可以保存它的值，调用API的时候直接用就可以了。Access Token 有一定的**有效期**，过期后需要重新授权。

## Links
* 原文地址：[http://www.cnblogs.com/e241138/archive/2013/03/15/sina-weibo-oauth-access_token.html]()