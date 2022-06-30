# UnionID 机制说明

如果开发者拥有多个移动应用、网站应用、和公众帐号（包括小程序），可通过 UnionID 来区分用户的唯一性，因为只要是同一个微信开放平台帐号下的移动应用、网站应用和公众帐号（包括小程序），用户的 UnionID 是唯一的。换句话说，同一用户，对同一个微信开放平台下的不同应用，UnionID是相同的。

![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/0.png)



# 微信开放平台绑定流程

登录[微信开放平台](https://open.weixin.qq.com/) — 管理中心 

![img](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/union_bind.e13e592b.png)



# UnionID获取途径

## 小程序

绑定了开发者帐号的小程序，可以通过以下途径获取 UnionID。

1. 开发者可以直接通过 [wx.login](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/login/wx.login.html) + `code2Session` 获取到该用户 UnionID，无须用户授权。
2. 小程序端调用云函数时，可在云函数中通过 [Cloud.getWXContext](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/reference-sdk-api/utils/Cloud.getWXContext.html) 获取 UnionID。
3. 用户在小程序（暂不支持小游戏）中支付完成后，开发者可以直接通过`getPaidUnionId`接口获取该用户的 UnionID，无需用户授权。注意：本接口仅在用户支付完成后的5分钟内有效，请开发者妥善处理。



## 公众号

绑定为公众号网页开发者的公众号，可以通过以下途径获取UnionID。

1.基于微信开发的网页中主动授权用户信息后（静默授权得不到），调用**拉取用户信息(需 scope 为 snsapi_userinfo)接口**可以得到UnionID。

2.当用户关注公众号后通过关注事件或取到openid调用**获取用户基本信息（包括 UnionID 机制）**接口可以得到UnionID。



### 拉取用户信息(需 scope 为 snsapi_userinfo)

**请求方法**

> http：GET（请使用 https 协议）：

> https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN

| 参数         | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| access_token | 网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同 |
| openid       | 用户的唯一标识                                               |
| lang         | 返回国家地区语言版本，zh_CN 简体，zh_TW 繁体，en 英语        |

**返回说明**

正确时返回的 JSON 数据包如下：

```json
{   
  "openid": "OPENID",
  "nickname": NICKNAME,
  "sex": 1,
  "province":"PROVINCE",
  "city":"CITY",
  "country":"COUNTRY",
  "headimgurl":"https://thirdwx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ4eMsv84eavHiaiceqxibJxCfHe/46",
  "privilege":[ "PRIVILEGE1" "PRIVILEGE2"     ],
  "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
}
```

| 参数       | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| openid     | 用户的唯一标识                                               |
| nickname   | 用户昵称                                                     |
| sex        | 用户的性别，值为1时是男性，值为2时是女性，值为0时是未知      |
| province   | 用户个人资料填写的省份                                       |
| city       | 普通用户个人资料填写的城市                                   |
| country    | 国家，如中国为CN                                             |
| headimgurl | 用户头像，最后一个数值代表正方形头像大小（有0、46、64、96、132数值可选，0代表640*640正方形头像），用户没有头像时该项为空。若用户更换头像，原有头像 URL 将失效。 |
| privilege  | 用户特权信息，json 数组，如微信沃卡用户为（chinaunicom）     |
| unionid    | 只有在用户将公众号绑定到微信开放平台帐号后，才会出现该字段。 |







### **获取用户基本信息（包括 UnionID 机制）**

开发者可通过 OpenID 来获取用户基本信息。请使用 https 协议。

> 接口调用请求说明 http请求方式: GET https://api.weixin.qq.com/cgi-bin/user/info?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN

参数说明

| 参数         | 是否必须 | 说明                                                  |
| :----------- | :------- | :---------------------------------------------------- |
| access_token | 是       | 调用接口凭证                                          |
| openid       | 是       | 普通用户的标识，对当前公众号唯一                      |
| lang         | 否       | 返回国家地区语言版本，zh_CN 简体，zh_TW 繁体，en 英语 |

返回说明

正常情况下，微信会返回下述 JSON 数据包给公众号：

```json
{
    "subscribe": 1, 
    "openid": "o6_bmjrPTlm6_2sgVt7hMZOPfL2M", 
    "language": "zh_CN", 
    "subscribe_time": 1382694957,
    "unionid": " o6_bmasdasdsad6_2sgVt7hMZOPfL",
    "remark": "",
    "groupid": 0,
    "tagid_list":[128,2],
    "subscribe_scene": "ADD_SCENE_QR_CODE",
    "qr_scene": 98765,
    "qr_scene_str": ""
}
```

参数说明

| 参数            | 说明                                                         |
| :-------------- | :----------------------------------------------------------- |
| subscribe       | 用户是否订阅该公众号标识，值为0时，代表此用户没有关注该公众号，拉取不到其余信息。 |
| openid          | 用户的标识，对当前公众号唯一                                 |
| language        | 用户的语言，简体中文为zh_CN                                  |
| subscribe_time  | 用户关注时间，为时间戳。如果用户曾多次关注，则取最后关注时间 |
| unionid         | 只有在用户将公众号绑定到微信开放平台帐号后，才会出现该字段。 |
| remark          | 公众号运营者对粉丝的备注，公众号运营者可在微信公众平台用户管理界面对粉丝添加备注 |
| groupid         | 用户所在的分组ID（兼容旧的用户分组接口）                     |
| tagid_list      | 用户被打上的标签 ID 列表                                     |
| subscribe_scene | 返回用户关注的渠道来源，ADD_SCENE_SEARCH 公众号搜索，ADD_SCENE_ACCOUNT_MIGRATION 公众号迁移，ADD_SCENE_PROFILE_CARD 名片分享，ADD_SCENE_QR_CODE 扫描二维码，ADD_SCENE_PROFILE_LINK 图文页内名称点击，ADD_SCENE_PROFILE_ITEM 图文页右上角菜单，ADD_SCENE_PAID 支付后关注，ADD_SCENE_WECHAT_ADVERTISEMENT 微信广告，ADD_SCENE_REPRINT 他人转载 ,ADD_SCENE_LIVESTREAM 视频号直播，ADD_SCENE_CHANNELS 视频号 , ADD_SCENE_OTHERS 其他 |
| qr_scene        | 二维码扫码场景（开发者自定义）                               |
| qr_scene_str    | 二维码扫码场景描述（开发者自定义）                           |



# UnionID互通流程

**模型图**

![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/%E6%A8%A1%E5%9E%8B%E5%9B%BEV2.png)





**时序图**

![](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/UnionID%E4%BA%92%E9%80%9A%E6%97%B6%E5%BA%8F%E5%9B%BE.png)



**关系图**

![image-20220620164132263](https://qxguide.oss-cn-beijing.aliyuncs.com/blog/images/image-20220620164132263.png)







