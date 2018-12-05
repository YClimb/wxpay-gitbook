---
description: 本文是【浅析微信支付】系列文章的第十六篇，主要讲解如何使用微信公众平台的卡券功能、如何使用HTML5在网页展示用户领券以及微信卡券和商户平台代金券的关系。
---

# 公众平台卡券功能开通、HTML5线上发券（JS-SDK接口）、查看卡券详情

浅析微信支付系列已经更新十六篇了哟～，没有看过的朋友们可以看一下哦。

[浅析微信支付：开通免充值产品功能及如何进行接口升级指引](https://mp.weixin.qq.com/s/78JA5-mdQ0I0g_ob5EgUBg)

[浅析微信支付：商户平台代金券或立减优惠开通、指定用户代金券发放、查询等](https://mp.weixin.qq.com/s/RGLjNNxBW8ccQ4AVWjYEbA)

[浅析微信支付：商户平台开通现金红包、指定用户发放、红包记录查询](https://mp.weixin.qq.com/s/3fi9TjNMpZ9AbrcNZF1vKA)

[浅析微信支付：支付验收示例和验收指引](https://mp.weixin.qq.com/s/YESU2V5byxfM8z9YQXgnuA)

[浅析微信支付：如何使用沙箱环境测试](https://mp.weixin.qq.com/s/WmnsCnIrhN9STbvrNTQOiA)

前几篇文章主要介绍了如何在【微信商户平台】使用代金券和满减优惠折扣等产品功能，有不少小伙伴说到，【微信公众平台】也有一个卡券功能，那么他们有什么差别呢？这个卡券功能该如何使用？本文会给大家一个解释。

#### 两者的差别

首先，我们来解释商户平台和微信平台各自优惠券的区别，如果有人试过，那么应该知道，两者是不通用的，不通用的，不通用的！！！

至于这里要重点标识不通用？因为在开通微信卡券功能后，在商户平台也会出现微信卡券对应优惠券信息，虽然没有发券的功能，只是展示，但如果我们走接口发券，就会出现 `发券失败，不支持发送xxx类型的优惠券` 的错误，这时就尴尬了；

还没完，因为平台不同，所以微信卡券和支付优惠券发送、领取的方式（接口）也是不同的，包括用户领取时跳转到的页面也不相同，这个也请大家注意。

所以，我们需要将这两种券作为两种不同种类的券来处理就可以了。

#### 公众平台卡券功能开通

登陆公众平台 `https://mp.weixin.qq.com`，点击左侧\[功能-添加功能插件\]，进入插件库页面，进入\[卡券功能\]，可以开通卡券功能。

申请条件： 必须开通微信支付功能！！！！

功能介绍： 卡券功能，是提供给商户或第三方的一套派发优惠券，经营管理会员的工具，可在公众平台或通过接口创建卡券，多种渠道投放给用户，用户用券时需核销卡券，核销后可查看数据、进行对账。

主要能力： 

●朋友共享的优惠券——可利用社交链快速扩散传播，一人领券，本人和朋友皆可看到并使用。查看视频介绍 

●普通优惠券——传统优惠券电子版，领取后仅本人可见可用，支持多种类型：折扣券、代金券、兑换券、团购券、优惠券。 

●会员卡——支持折扣、积分等玩法，并提供会员管理、数据报表等丰富工具，便于商户高效运营会员。 

●微信买单——无需进行微信支付开发，同时与会员卡，代金券，折扣券打通，为你积累用户消费数据，用于经营参考 

●储值功能——会员卡商户无需申请，可直接通过API接口，使用“余额展示”功能，将会员余额显示在微信会员卡首页。具有预付卡资质的商家可申请“储值”功能，申请成功后，可通过API接口设置此入口，帮助会员通过微信支付为会员卡充值。 

●第三方代制模式——经商户授权后，可代子商户快速接入并使用卡券功能，支持通过公众平台或API接口实现该功能。

官方开发文档地址：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141229
```

这里就已知小伙伴们已经开通卡券功能，如何手动创建优惠券可以在公众平台上操作，很简单，这里就不说了，本文主要讲如何使用接口的方式来创建卡券、用户领取、查询卡券详情。

#### 微信卡券指引

官方文档地址：

```text
https://mp.weixin.qq.com/cgi-bin/readtemplate?t=cardticket/faq_tmpl&type=info&token=&lang=zh_CN#0
```

下面是`卡券功能开通指引`对应的申请渠道、申请条件：

![&#x5361;&#x5238;&#x529F;&#x80FD;&#x5F00;&#x901A;&#x6307;&#x5F15;](https://img-blog.csdnimg.cn/20181204202143255.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lDbGltYg==,size_16,color_FFFFFF,t_70)

有需要的小伙伴可以通读一下上面官方文档，读完之后就会对微信卡券有所认识了，差不多基础业务都能做到心中有数咯。

如果是开发者，直接看下面`微信卡券接口文档`就行，毕竟我们更关心接口相关的信息，官方文档如下：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141229
```

下载卡券资料包：

```text
https://mp.weixin.qq.com/zh_CN/htmledition/comm_htmledition/res/cardticket/wx_card_document.zip?token=&lang=zh_CN
```

根据官方文档的步骤，需要整整七步，下面为具体步骤： 1. 获取access\_token 2. 上传卡券logo 3. 创建卡券 4. 创建二维码投放 5. 显示二维码 6. 设置测试白名单 7. 核销卡劵

官方的文档主要是使用了`沙箱测试账号`来测试并验证，关于接口测试号申请可以通过以下链接来取得：

```text
https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login
```

下面我们来一步步分析接口。

#### 获取access\_token

页面地址：`http://mp.weixin.qq.com/debug/`

接口类型：基础支持

接口列表：获取access\_token接口

注意事项：参数填写开发者的appid和secret

点击检查问题，即可返回access\_token，access\_token的有效期是两小时，两小时之后须重新获取

接口地址：获取access\_token接口 `https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140183`

这里的获取微信全局access\_token接口就不讲了，能看到这篇文章的小伙伴应该早就写过了，哈哈哈。

#### 上传卡券logo

页面地址：`http://mp.weixin.qq.com/debug/`

接口类型：基础支持

接口列表：上传图片素材接口

access\_token: 上一步获得的access\_token

buffer：你选择的图片

点击检查问题，即可获取图片url，在下一步创建卡劵的参数中需要

接口地址：上传图片接口 `https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1451025056`

此接口就是上传商户logo，可以获得一个卡券logo链接，基本上就用一次，如果不想使用此接口来上传logo，可以在公众平台手动创建优惠券时上传logo，上传后可以在logo图片上鼠标右键-复制图片路径，也是一样的（不想调接口的可以用这个歪招哈哈哈哈）。

#### 创建卡券

页面地址：`http://mp.weixin.qq.com/debug/`

接口类型：卡劵接口

接口列表：创建卡劵接口

access\_token:第一步获得的access\_token

创建卡券接口地址：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1451025056
```

以下为调用接口示例：

```text
https://api.weixin.qq.com/card/create?access_token=xxx
```

JSON示例：

```text
{
    "card": {
        "card_type": "CASH", 
        "cash": {
            "base_info": {
                "logo_url": "https://mmbiz.qlogo.cn/mmbiz_png/E5Q3G4ku9nf7UiafOetcAPLyfia7kdWWWauHukNN7ZXnggtZcTzEPGa8IUDiaLIv14EkNdPvmbCFyHibh0G8tia7Eibw/0?wx_fmt=png", // 此处是上传logo的图片地址
                "pay_info": {
                    "swipe_card": {
                        "use_mid_list": [
                            "xxx" // 商户号
                        ], 
                        "create_mid": "xxx", // 商户号
                        "is_swipe_card": true
                    }
                }, 
                "brand_name": "测试代金券", 
                "code_type": "CODE_TYPE_NONE", 
                "title": "111", 
                "color": "Color090",  // 主题颜色
                "service_phone": "18888888888", 
                "description": "不可与其他优惠同享如需团购券发票，请在消费时向商户提出", 
                "date_info": {
                    "type": "DATE_TYPE_FIX_TIME_RANGE", 
                    "begin_timestamp": 1536768000,  // 开始时间
                    "end_timestamp": 1536940800 // 结束时间
                }, 
                "can_share": false, 
                "center_title": "立即使用", 
                "center_app_brand_user_name": "gh_7195ea80d2e6@app", 
                "center_app_brand_pass": "pages/index/index", 
                "can_give_friend": false, 
                "sku": {
                    "quantity": 500000
                }, 
                "get_limit": 30, 
                "custom_url_name": "立即使用", 
                "custom_url": "http://www.qq.com", 
                "custom_url_sub_title": "6个汉字tips", 
                "promotion_url_name": "更多优惠", 
                "promotion_url": "http://www.qq.com"
            }, 
            "advanced_info": {
                "use_condition": {
                    "accept_category": "鞋类", 
                    "reject_category": "阿迪达斯", 
                    "can_use_with_other_discount": true, 
                    "least_cost": "51"
                }, 
                "abstract": {
                    "abstract": "微信餐厅推出多种新季菜品，期待您的光临", 
                    "icon_url_list": [
                        "http://mmbiz.qpic.cn/mmbiz/p98FjXy8LacgHxp3sJ3vn97bGLz0ib0Sfz1bjiaoOYA027iasqSG0sjpiby4vce3AtaPu6cIhBHkt6IjlkY9YnDsfw/0"
                    ]
                }, 
                "text_image_list": [
                    {
                        "image_url": "http://mmbiz.qpic.cn/mmbiz/p98FjXy8LacgHxp3sJ3vn97bGLz0ib0Sfz1bjiaoOYA027iasqSG0sjpiby4vce3AtaPu6cIhBHkt6IjlkY9YnDsfw/0", 
                        "text": "此菜品精选食材，以独特的烹饪方法，最大程度地刺激食 客的味蕾"
                    }, 
                    {
                        "image_url": "http://mmbiz.qpic.cn/mmbiz/p98FjXy8LacgHxp3sJ3vn97bGLz0ib0Sfz1bjiaoOYA027iasqSG0sj piby4vce3AtaPu6cIhBHkt6IjlkY9YnDsfw/0", 
                        "text": "此菜品迎合大众口味，老少皆宜，营养均衡"
                    }
                ], 
                "time_limit": [
                    {
                        "type": "MONDAY", 
                        "begin_hour": 0, 
                        "end_hour": 10, 
                        "begin_minute": 10, 
                        "end_minute": 59
                    }, 
                    {
                        "type": "HOLIDAY"
                    }
                ], 
                "business_service": [
                    "BIZ_SERVICE_FREE_WIFI", 
                    "BIZ_SERVICE_WITH_PET", 
                    "BIZ_SERVICE_FREE_PARK", 
                    "BIZ_SERVICE_DELIVER"
                ]
            }, 
            "reduce_cost": 5
        }
    }
}
```

通过以上接口和参数可以创建一个优惠券信息，json示例也可以使用官方的，这里有几个问题需要重点说一下： 1. use\_mid\_list 商户号需要填写本商户的 2. color 颜色可以根据文档中选择 3. begin\_timestamp、end\_timestamp 这两个时间非常重要，首先结束时间必须大于开始时间，并且需要大于一定的限度，测试时最好跨度大于一天；还需要注意的是时间格式必须是10位数值，例如：1536768000，其他格式会报错。 4. time\_limit 下的值根据文档中的要求填写 5. sku、get\_limit 参数按要求填写 6. logo\_url 使用微信官方的logo图片地址

注意事项：date\_info中用的是Unix时间戳，注意把begin\_timestamp修改小于当前时间，end\_timestamp修改成今天之后的时间，这样在后面核销卡劵测试才能成功

#### 创建二维码投放

页面地址：[http://mp.weixin.qq.com/debug](http://mp.weixin.qq.com/debug)

接口类型：卡劵接口

接口列表：创建二维码ticket接口

access\_token：第一步获得的access\_token

接口文档地址：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1451025062
```

接口调用示例：

```text
https://api.weixin.qq.com/card/qrcode/create?access_token=TOKEN
```

json参数：

```text
{
    "action_name":"QR_CARD",
    "expire_seconds":1800,
    "action_info":{
        "card":{
            "card_id":"pFS7Fjg8kV1IdDz01r4SQwMkuCKc",
            "code":"198374613512",
            "openid":"oFS7Fjl0WsZ9AMZqrI80nbIq8xrA",
            "is_unique_code":false,
            "outer_str":"12b"
        }
    }
}
```

#### 显示二维码

在上一步的返回中点击字段show\_qrcode\_url字段中的链接，即可显示卡券领取二维码。 打开微信扫一扫，然后领取卡劵，如果显示卡劵未通过审核，那么需要下一步设置测试白名单，如果可以领取就忽略第六步。

#### 设置测试白名单

文档地址：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1451025062
```

#### 核销卡劵

文档地址：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1451025239
```

注意事项：仅支持审核通过且在有效期内的卡劵

#### HTML5网页发券（JS-SDK）

上面通过二维码发券的方式，我觉得官方文档已经解释清楚了，且接口也比较简单，所以这里就不赘述，主要需要讲的是这里的 `HTML5网页发券（JS-SDK）`，官方文档链接如下：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1451025062
```

![HTML5&#x7EBF;&#x4E0A;&#x53D1;&#x5238;&#xFF08;JS-SDK&#x63A5;&#x53E3;&#xFF09;](https://img-blog.csdnimg.cn/20181204202210256.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lDbGltYg==,size_16,color_FFFFFF,t_70)

看了上面的图，小伙伴大概知道是什么意思了吧，简单点说，就是用户在咋们系统H5中点击按钮，可以弹出微信官方的领取优惠券界面，官方的页面是微信提供的，我们无需开发，只需要关注如何调用官方的方法即可。

官方解释如下：微信提供的addCard接口供商户前端网页调用，用于将一张或多张卡券添加到用户卡包

接口地址如下\(在以下页面搜索addCard即可直达\)：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115
```

![&#x6279;&#x91CF;&#x6DFB;&#x52A0;&#x5361;&#x5238;&#x63A5;&#x53E3;](https://img-blog.csdnimg.cn/20181204202223449.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lDbGltYg==,size_16,color_FFFFFF,t_70)

上面是添加卡券的接口，我们重点关注这几个参数: 1. cardId：卡券ID，如果是商户平台，则是批次ID 2. cardExt：卡券的扩展参数。需进行 JSON 序列化为字符串传入 3. cardExt&gt;openid：指定领取者的 openid，只有该用户能领取。 bind\_openid 字段为 true 的卡券必须填写，bind\_openid 字段为 false 不可填写。 4. cardExt&gt;code：用户领取的 code，仅自定义 code 模式的卡券须填写，非自定义 code 模式卡券不可填写 5. cardExt&gt;nonce\_str：随机字符串，由开发者设置传入，加强安全性（若不填写可能被重放请求）。随机字符串，不长于 32 位。推荐使用大小写字母和数字，不同添加请求的 nonce\_str 须动态生成，若重复将会导致领取失败。 6. cardExt&gt;signature：签名，商户将接口列表中的参数按照指定方式进行签名,签名方式使用 SHA1，具体签名方案参见：卡券签名 7. cardExt&gt;timestamp：时间戳，东八区时间,UTC+8，单位为秒 8. cardExt&gt;outer\_str：领取渠道参数，用于标识本次领取的渠道值。 9. cardExt&gt;fixed\_begintimestamp：卡券在第三方系统的实际领取时间，为东八区时间戳（UTC+8,精确到秒）。当卡券的有效期类为 DATE\_TYPE\_FIX\_TERM 时专用，标识卡券的实际生效时间，用于解决商户系统内起始时间和领取微信卡券时间不同步的问题。

**根据代金券批次ID得到组合的cardList**

首先，获取cardList接口需要先获取access\_token，再获取api\_ticket，最后组装成想要的集合，下面是示例代码。

根据代金券批次ID得到组合的cardList:

```text
/**
 * 根据代金券批次ID得到组合的cardList
 *
 * @param cardId 卡包ID
 * @return cardList
 * @author yclimb
 * @date 2018/9/21
 */
public JSONArray getCardList(String cardId) {
    if (StringUtils.isBlank(cardId)) {
        return null;
    }
    try {

        // 获取[商户名称]公众号的 access_token
        String accessToken = this.getAccessToken(WXConstants.WX_MINI_PROGRAM_CODE);
        String timestamp = String.valueOf(WXPayUtil.getCurrentTimestamp());
        String nonce_str = WXPayUtil.generateNonceStr();

        // 卡券的扩展参数。需进行 JSON 序列化为字符串传入
        JSONObject cardExt = new JSONObject();
        //cardExt.put("code", "");
        //cardExt.put("openid", "");
        //cardExt.put("fixed_begintimestamp", "");
        //cardExt.put("outer_str", "");
        cardExt.put("timestamp", timestamp);
        cardExt.put("nonce_str", nonce_str);

        /**
         * 1.将 api_ticket、timestamp、card_id、code、openid、nonce_str的value值进行字符串的字典序排序。
         * 2.将所有参数字符串拼接成一个字符串进行sha1加密，得到signature。
         * 3.signature中的timestamp，nonce字段和card_ext中的timestamp，nonce_str字段必须保持一致。
         */
        Map<String, String> map = new HashMap<>(8);
        //map.put("code", "");
        //map.put("openid", "");
        map.put("api_ticket", this.getWxCardApiTicket(accessToken));
        map.put("timestamp", timestamp);
        map.put("card_id", cardId);
        map.put("nonce_str", nonce_str);
        cardExt.put("signature", WXPayUtil.SHA1(WXPayUtil.dictionaryOrder(map, 2)));

        // 卡券对象
        JSONObject cardInfo = new JSONObject();
        cardInfo.put("cardId", cardId);
        cardInfo.put("cardExt", cardExt.toJSONString());

        // 需要添加的卡券列表
        JSONArray cardList = new JSONArray(1);
        cardList.add(cardInfo);

        return cardList;
    } catch (Exception e) {
        WXPayUtil.getLogger().error(e.getMessage(), e);
    }
    return null;
}
```

获取微信全局accessToken：

```text
/**
 * 获取微信全局accessToken
 *
 * @param code 标识
 * @return accessToken
 */
public String getAccessToken(String code) {

    // 取redis数据
    String key = WXConstants.WECHAT_ACCESSTOKEN + code;
    String accessToken = (String) redisTemplate.opsForValue().get(key);
    if (accessToken != null) {
        return accessToken;
    }

    // 通过接口取得access_token
    JSONObject jsonObject = restTemplate.getForObject(MessageFormat.format(WXURL.BASE_ACCESS_TOKEN, WXPayConstants.APP_ID, WXPayConstants.SECRET), JSONObject.class);
    String token = (String) jsonObject.get("access_token");
    if (StringUtils.isNotBlank(token)) {
        // 存储redis
        redisTemplate.opsForValue().set(key, token, 7000, TimeUnit.SECONDS);
        return token;
    } else {
        log.error("获取微信accessToken出错，微信返回信息为：[{}]", jsonObject.toString());
    }
    return null;
}
```

获取卡券 api\_ticket 的 api：

```text
/**
 * 获取卡券 api_ticket 的 api
 * 请求路径：https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token={0}&type=wx_card
 *
 * @param access_token token
 * @return api_ticket json obj
 * @author yclimb
 * @date 2018/9/21
 */
public String getWxCardApiTicket(String access_token) {
    if (StringUtils.isBlank(access_token)) {
        return null;
    }
    try {

        // redis key
        String redisKey = RedisKeyUtil.keyBuilder(RedisKeyEnum.IMALL_WXCARD_APITICKET, access_token);

        // 从redis中获取缓存
        Object obj = redisTemplate.opsForValue().get(redisKey);
        if (obj != null) {
            return obj.toString();
        }

        // 获取卡券 api_ticket
        String api_ticket = restTemplate.getForObject(WXURL.BASE_API_TICKET, String.class, access_token);
        WXPayUtil.getLogger().info("getWxCardApiTicket:api_ticket:{}", api_ticket);
        if (StringUtils.isBlank(api_ticket)) {
            return null;
        }
        JSONObject jsonObject = JSON.parseObject(api_ticket);
        if (0 != jsonObject.getIntValue("errcode")) {
            return null;
        }

        // 设置到redis中，下次取直接拿缓存即可，防止多次生成
        String ticket = jsonObject.getString("ticket");
        redisTemplate.opsForValue().set(redisKey, ticket, jsonObject.getIntValue("expires_in"), TimeUnit.SECONDS);

        return ticket;
    } catch (Exception e) {
        WXPayUtil.getLogger().error(e.getMessage(), e);
    }
    return null;
}
```

以上代码可以获取cardList，将此参数替换到微信官方方法中即可唤起领券页面；需要注意的是，公众平台和商户平台的券领取的方式不同，这里是以公众平台为例子，如果小伙伴要用商户平台这样为用户发券，是无法成功的哟。

#### 查看卡券详情

开发者可以调用该接口查询某个card\_id的创建信息、审核状态以及库存数量。

接口调用：

```text
HTTP请求方式：POST
URL：https://api.weixin.qq.com/card/get?access_token=TOKEN
参数：card_id，卡券ID
```

接口文档\(进入链接后查询 `查看卡券详情` 可快速定位\)：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1451025272
```

此接口官方介绍已经很详细，这里就不细讲了，大家参考官方文档即可。

#### 结语

以上为`微信公众平台-卡券功能`相关的解释和源码，小伙伴们一定要注意看看官方文档哦，具体的源码可以看作者的github，里面对每个方法有详细的注释。

如果小伙伴有遇到解决不了的问题，可以关注作者微信公众号，加入讨论群中发出疑问，和小伙伴们一起解决哦～

预告：下一篇文章会讲发放奖励的另一种方式 `公众平台-社交立减金活动`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： ​ ​`​https://github.com/YClimb/wxpay-sdk/blob/master/README.md ​`

关注作者微信公众号，点击下方`讨论群`，扫码即可加入`微信支付讨论群`与小伙伴一起探讨哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x516C;&#x4F17;&#x53F7;](https://img-blog.csdn.net/20180130111432962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvWUNsaW1i/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

