---
description: 本文是【浅析微信支付】系列文章的第十七篇，主要讲解在在微信平台中，如何创建优惠券，开通社交立减金，并为用户配置发送立减金。
---

# 开通社交立减金活动、创建立减金及领取使用的相关文档和源码

上篇文章已经为大家讲解了如何在微信公众平台创建优惠券并为用户发券，这片文章是优惠券的一个进阶，讲解微信平台上的`社交立减金`用法，希望可以帮助到大家。

#### 应用场景

小程序社交立减金是一款帮助商家快速生成具备裂变传播属性的小程序经营工具，用户通过支付、扫码等场景可以参与社交立减金活动，将社交立减金礼包分享至朋友后自己可获取一份，朋友在会话中可随机获取社交立减金，并直达商家小程序使用。

产品优势 1. 打通“小程序+支付+优惠”，支付成功页分发社交立减金，支付时抵扣使用； 2. 通过聊天分享和裂变，触达更多潜在用户，降低拉新成本； 3. 根据用户标签属性，可配置分发不同金额的社交立减金，如新老用户、是否会员等； 4. 结合用户偏好兴趣，可配置个性化商品推荐，提高转化。

官方文档地址：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=21515658940X5pIn
```

#### 接入流程

进入上面文档地址，文中有开通`公众号活动配置`权限，这个权限必须开启，开启指南如下：

```text
社交立减金可通过微信支付后向用户下发消息领取，在配置活动时需要登录发起支付的商户号开通权限。

Q:在微信支付平台开通“公众号活动配置”权限
A:使用管理员帐号（管理员为申请商户号时给用户分配的登录账号）登录微信支付商户平台pay.weixin.qq.com，进入“产品中心－我的产品－运营工具”，进入并开通“公众号活动配置”权限。
```

开通权限之后，还需要`完成免充值模式验收`，这个验收在我的前几篇文章中已经讲过，下面贴上对应的链接查看： [浅析微信支付：开通免充值产品功能及如何进行接口升级指引](https://mp.weixin.qq.com/s/78JA5-mdQ0I0g_ob5EgUBg)

完成免充值模式验收后，即可通过接口创建与支付打通的代金券，为配置立减金活动做准备。

若已完成免充值模式验收，可直接调用`创建代金券并设置跳转小程序`，下面是创建的流程和对应代码。

#### 创建社交立减金的步骤说明

首先，我们来理解一下社交立减金的构成和创建流程，这样才能不被微信给弄迷糊了，下面是几个步骤的说明： 1. 开通微信公众平台中的`微信卡券`功能 2. 开通微信商户平台中的`公众号活动配置`功能 3. 开通微信商户平台中的`免充值`相关产品功能 4. 完成免充值模式验收 5. 在`微信商户平台`创建或者通过接口创建微信代金券 6. 登陆`微信支付商户平台 -营销中心-管理代金券-草稿箱`中激活对应卡券 7. 通过`创建支付后领取立减金活动接口`创建活动 8. 登陆`微信支付商户平台 -营销中心-营销活动-满额送-管理-草稿箱`激活活动。 9. 下单购买某个商品后，在`服务通知`消息中可以领取或者转发社交立减金 10. 领取，结束

上面的步骤，缺一不可，需要小伙伴一步步的调试和试错，幸运的是，我已经都试过错，所以把经验总结一下，希望小伙伴们不会再次趟坑，创建社交立减金活动的源码也是有的哦～

好了，话不多说，对于上面的10步，前4步不需要讲，不清楚的小伙伴可以看看我的历史文章，有对应的讲解，下面从第5步开始说起。

#### 创建代金券并设置跳转小程序

下面先讲解接口的方式，完成配置社交立减金活动奖品准备环节，通过本接口开发者可以创建和支付打通的代金券，并设置代金券跳转小程序使用，在 base\_info 中新增设置字段"pay\_info"，详情参考创建卡券接口。

```text
协议：https  
http请求方式: POST  
请求URL：https://api.weixin.qq.com/card/create?access_token=ACCESS_TOKEN  
POST数据格式：JSON
```

对于POST的数据可以参考我的历史文章`微信卡券创建`，或者查看以下官方文档：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=21515658940X5pIn
```

注意： 1. 卡券提交前需检查是否有传："pay\_info"、"is\_swipe\_card": true 字段； 2. 卡券的起止时间应大于当前时间；如当前时间为：2018/1/17 0:0:0，起止时间需大于当前时间：2018/1/17 13:30:03 （时间精确到秒）； 3. 需设置最低消费门槛，如：5元代金券，满10元可用。 "reduce\_cost": 5 ,"least\_cost":10 。

推荐查看我的文章，里面有一些字段的注意事项，这里就不细讲了，下面说一下第二种方式，就是直接通过微信商户平台创建代金券，创建完成后，需要在“微信支付商户平台 -营销中心-管理代金券-草稿箱”中激活对应卡券，获取卡券id后，可继续创建立减金活动。

#### 创建支付后领取立减金活动接口

重头戏来了，可以通过此接口创建立减金活动。将已创建的代金券cardid、跳转小程序appid、发起支付的商户号等信息通过此接口创建立减金活动，成功返回活动id即为创建成功。

```text
协议：https  
http请求方式: POST  
请求URL：https://api.weixin.qq.com/card/mkt/activity/create?access_token=ACCESS_TOKEN 
POST数据格式：JSON
```

大白话解释一下，上面的代金券cardid就是通过接口返回的卡券ID，或者是通过手动创建后的卡包ID，就是乱七八糟一串英文的那一个，例如：`pX2-vjpU_MT1gFDsP8lNl15PdaFS`，官方的POST示例可以查看文档：

```text
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=21515658940X5pIn
```

好消息来了，这个接口我已经封装在sdk中了，封装为方法便于大家使用，下面是调用方法和具体方法的代码。

调用方法：

```text
/**
 * 创建支付后领取立减金活动接口
 *
 * @author yclimb
 * @date 2018/9/18
 */
private void createCardActivity() {
    WXUtils wxUtils = new WXUtils();
    wxUtils.createCardActivity("2018-09-18 18:00:00", "2018-09-18 19:59:59", 3, 1,
            1, "pX2-vjpU_MT1gFDsP8lNl15PdaZE", "100",
            null, false);
}
```

方法源码：

```text
/**
 * 创建支付后领取立减金活动接口
 * 通过此接口创建立减金活动。
 * 将已创建的代金券cardid、跳转小程序appid、发起支付的商户号等信息通过此接口创建立减金活动，成功返回活动id即为创建成功。
 * 接口地址：https://mp.weixin.qq.com/wiki?t=resource/res_main&id=21515658940X5pIn
 *
 * @param begin_time               活动开始时间，精确到秒
 * @param end_time                 活动结束时间，精确到秒
 * @param gift_num                 单个礼包社交立减金数量（3-15个）
 * @param max_partic_times_act     每个用户活动期间最大领取次数,最大为50，默认为1
 * @param max_partic_times_one_day 每个用户活动期间单日最大领取次数,最大为50，默认为1
 * @param card_id                  卡券ID
 * @param min_amt                  最少支付金额，单位是元
 * @param membership_appid         奖品指定的会员卡appid。如用户标签有选择商户会员，则需要填写会员卡appid，该appid需要跟所有发放商户号有绑定关系。
 * @param new_tinyapp_user         可以指定为是否小程序新用户（membership_appid为空、new_tinyapp_user为false时，指定为所有用户）
 * @return json
 * @author yclimb
 * @date 2018/9/18
 */
public JSONObject createCardActivity(String begin_time, String end_time, int gift_num, int max_partic_times_act,
                                     int max_partic_times_one_day, String card_id, String min_amt,
                                     String membership_appid, boolean new_tinyapp_user) {
    try {

        // 创建活动接口之前的验证
        String msg = checkCardActivity(begin_time, end_time, gift_num, max_partic_times_act, max_partic_times_one_day, min_amt);
        if (null != msg) {
            JSONObject resultJson = new JSONObject(2);
            resultJson.put("errcode", "1");
            resultJson.put("errmsg", msg);
            return resultJson;
        }

        // 获取[商户名称]公众号的 access_token
        String accessToken = this.getAccessToken(WXConstants.WX_MINI_PROGRAM_CODE);

        // 调用接口传入参数
        JSONObject paramJson = new JSONObject(1);

        // info 包含 basic_info、card_info_list、custom_info
        JSONObject info = new JSONObject(3);

        // 基础信息对象
        JSONObject basic_info = new JSONObject(8);
        // activity_bg_color    是    活动封面的背景颜色，可参考：选取卡券背景颜色
        basic_info.put("activity_bg_color", CardBgColorEnum.COLOR_090.getBgName());
        // activity_tinyappid    是    用户点击链接后可静默添加到列表的小程序appid；
        basic_info.put("activity_tinyappid", WXPayConstants.APP_ID);
        // mch_code    是    支付商户号
        basic_info.put("mch_code", WXPayConstants.MCH_ID);
        // begin_time    是    活动开始时间，精确到秒（unix时间戳）
        basic_info.put("begin_time", DateTimeUtil.getTenTimeByDate(begin_time));
        // end_time    是    活动结束时间，精确到秒（unix时间戳）
        basic_info.put("end_time", DateTimeUtil.getTenTimeByDate(end_time));
        // gift_num    是    单个礼包社交立减金数量（3-15个）
        basic_info.put("gift_num", gift_num);
        // max_partic_times_act    否    每个用户活动期间最大领取次数,最大为50，不填默认为1
        basic_info.put("max_partic_times_act", max_partic_times_act);
        // max_partic_times_one_day    否    每个用户活动期间单日最大领取次数,最大为50，不填默认为1
        basic_info.put("max_partic_times_one_day", max_partic_times_one_day);

        // card_info_list    是    可以配置两种发放规则：小程序新老用户、新老会员
        JSONArray card_info_list = new JSONArray(1);
        JSONObject card_info = new JSONObject(3);
        // card_id    是    卡券ID
        card_info.put("card_id", card_id);
        // min_amt    是    最少支付金额，单位是分
        card_info.put("min_amt", String.valueOf(new BigDecimal(min_amt).multiply(new BigDecimal(100)).setScale(2, BigDecimal.ROUND_HALF_UP).intValue()));
        /*
         * membership_appid    是    奖品指定的会员卡appid。如用户标签有选择商户会员，则需要填写会员卡appid，该appid需要跟所有发放商户号有绑定关系。
         * new_tinyapp_user    是    可以指定为是否小程序新用户
         * total_user    是    可以指定为所有用户
         * membership_appid、new_tinyapp_user、total_user以上字段3选1，未选择请勿填，不必故意填写false
         */
        if (StringUtils.isNotBlank(membership_appid)) {
            card_info.put("membership_appid", membership_appid);
        } else {
            if (new_tinyapp_user) {
                card_info.put("new_tinyapp_user", true);
            } else {
                card_info.put("total_user", true);
            }
        }
        card_info_list.add(card_info);

        // 自定义字段，表示支付后领券
        JSONObject custom_info = new JSONObject(1);
        custom_info.put("type", "AFTER_PAY_PACKAGE");

        // 拼装json对象
        info.put("basic_info", basic_info);
        info.put("card_info_list", card_info_list);
        info.put("custom_info", custom_info);
        paramJson.put("info", info);

        // 请求微信接口，得到返回结果[json]
        HttpEntity<JSONObject> entity = new HttpEntity<>(paramJson, this.getHttpHeadersUTF8JSON());
        JSONObject resultJson = restTemplate.postForObject(WXURL.WX_CARD_ACTIVITY_CREATE_URL, entity, JSONObject.class, accessToken);

        // {"errcode":0,"errmsg":"ok","activity_id":"4728935"}
        System.out.println(resultJson.toJSONString());

        return resultJson;
    } catch (Exception e) {
        WXPayUtil.getLogger().error(e.getMessage(), e);
    }
    return null;
}

/**
 * 创建活动接口之前的验证
 *
 * @param begin_time               活动开始时间，精确到秒
 * @param end_time                 活动结束时间，精确到秒
 * @param gift_num                 单个礼包社交立减金数量（3-15个）
 * @param max_partic_times_act     每个用户活动期间最大领取次数,最大为50，默认为1
 * @param max_partic_times_one_day 每个用户活动期间单日最大领取次数,最大为50，默认为1
 * @param min_amt                  最少支付金额，单位是元
 * @return msg str
 * @author yclimb
 * @date 2018/9/18
 */
public String checkCardActivity(String begin_time, String end_time, int gift_num, int max_partic_times_act,
                                int max_partic_times_one_day, String min_amt) {

    // 开始时间不能小于结束时间
    if (DateTimeUtil.latterThan(end_time, begin_time, DateTimeUtil.TIME_FORMAT_NORMAL)) {
        return "活动开始时间不能小于活动结束时间";
    }

    // 单个礼包社交立减金数量（3-15个）
    if (gift_num < 3 || gift_num > 15) {
        return "单个礼包社交立减金数量（3-15个）";
    }

    // 每个用户活动期间最大领取次数,最大为50，默认为1
    if (max_partic_times_act <= 0 || max_partic_times_act > 50) {
        return "每个用户活动期间最大领取次数,最大为50，默认为1";
    }

    // 每个用户活动期间单日最大领取次数,最大为50，默认为1
    if (max_partic_times_one_day <= 0 || max_partic_times_one_day > 50) {
        return "每个用户活动期间单日最大领取次数,最大为50，默认为1";
    }

    // 最少支付金额，单位是元
    if (BigDecimal.ONE.compareTo(new BigDecimal(min_amt)) > 0) {
        return "最少支付金额必须大于1元";
    }

    return null;
}
```

对应的全部代码，可以查看我的github，地址如下：

```text
https://github.com/YClimb/wxpay-sdk/blob/master/src/main/java/com/weixin/pay/util/WXUtils.java
```

创建成功后，会返回一个 "activity\_id": "123456" 的json数据，也就是立减金活动id咯，如果创建失败，也不要着急，查看上面说到的官方文档，创建接口下有对应的返回码说明， 已知是足够解决问题了。

活动创建成功后，需要登陆“微信支付商户平台 -营销中心-营销活动-满额送-管理-草稿箱”激活活动，激活后，社交立减金投放成功。可以通过支付行为进行验证。

PS：激活活动后，活动是通过满额送行为发送给用户的，而且是自动发放，如果你在平台上购买的商品价格满足规则，则会在`服务通知`中显示社交立减金的信息，此活动需要用户主动领取才行，用户领取的券最终就是咋们最初创建的`代金券`，此代金券使用规则和正常代金券一样。

PPS：社交立减金可以分享转发给朋友，这个是他的特点，对于每个活动的代金券规则和份数，大家一定要注意，根据场景投放。

#### 结语

以上为`社交立减金`相关的解释和源码，小伙伴们一定要注意看看官方文档哦，具体的源码可以看我的github，里面对每个方法有详细的注释。

如果小伙伴有遇到解决不了的问题，可以关注作者微信公众号，加入讨论群中发出疑问，和小伙伴们一起解决哦～

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： ​ ​`​https://github.com/YClimb/wxpay-sdk/blob/master/README.md ​`

关注作者微信公众号，点击下方`讨论群`，扫码即可加入`微信支付讨论群`与小伙伴一起探讨哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x516C;&#x4F17;&#x53F7;](https://img-blog.csdn.net/20180130111432962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvWUNsaW1i/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

