---
description: 本文是【浅析微信支付】系列文章的第十四篇，主要讲解在如何开通商户平台的代金券或立减优惠功能，商家向指定用户发送代金券，查询发送记录，代金券信息等。
---

# 商户平台代金券或立减优惠开通、指定用户代金券发放、查询等

浅析微信支付系列已经更新十四篇了哟～，没有看过的朋友们可以看一下哦。

[浅析微信支付：商户平台开通现金红包、指定用户发放、红包记录查询](https://mp.weixin.qq.com/s/3fi9TjNMpZ9AbrcNZF1vKA)

[浅析微信支付：\(余额提现\)企业付款到微信用户零钱或银行卡账户](https://mp.weixin.qq.com/s/YZkrbYm5t8HJPT_S4W9zrQ)

[浅析微信支付：支付验收示例和验收指引](https://mp.weixin.qq.com/s/YESU2V5byxfM8z9YQXgnuA)

[浅析微信支付：如何使用沙箱环境测试](https://mp.weixin.qq.com/s/WmnsCnIrhN9STbvrNTQOiA)

首先我们需要了解一下什么是代金券和立减优惠？

代金券是微信支付为商家提供的一个营销工具，他的主要功能可以简单理解为商家的满减券，比如常见的“满十减一”、“满x减x”这类，需要用户主动领取或者平台主动为用户发放，核销时会在微信支付调起界面显示优惠券信息。

立减优惠是微信支付为商家提供的另一种自主核销优惠，为何叫自主核销？因为此优惠是一个门槛，不需要用户领取，商家设置一个用户群里，比如全员优惠“满十减一”，那么所有人都可以享受这个优惠，直接在购买商品时自动扣减金额。

以上为简单的解释，下面我会结合官方文档来解释这两个优惠方式。

#### 代金券

微信支付代金券业务是基于微信支付，为了协助商户方便地实现营销优惠措施。针对部分有开发能力的商户，微信支付提供通过API接口实现运营代金券的功能

官方文档地址：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/sp_coupon.php?chapter=12_2&index=2
```

首先，这里我们讲接口发放代金券的方式，下面是代金券的三个接口： 

![&#x4EE3;&#x91D1;&#x5238;&#x63A5;&#x53E3;&#x4ECB;&#x7ECD;](https://img-blog.csdnimg.cn/20181120163710717.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lDbGltYg==,size_16,color_FFFFFF,t_70)

操作代金券开通和如何手动创建的官方文档如下：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/sp_coupon.php?chapter=12_7&index=3
```

这里说一下重点需要注意的地方，首先，代金券分为单品券和全场券，简单理解： 1. 单品券：指定某几个商品ID的商品可以使用的代金券 2. 全场券：所有商品都可以使用的代金券

PS：通过高级接口发放的代金券不能插入卡包，并且用户是没有感知的，这是一个缺点，大家一定要记住。

微信支付的代金券核销时都是在微信的确认支付窗口，如果有多个代金券，可以选择或者合并代金券支付，支付后在支付通知中会显示记录代金券使用的记录。

还需要注意一点，单品代金券核销时需要验证商品ID，这个商品ID在`预支付单`中`单品优惠活动detail字段`传入，json格式必填参数，字段中`goods_id`就是我们在商户后台创建代金券时填入的商品ID，具体的代码可以看我的`统一下单接口`文章和GitHub源码。

[浅析微信支付：统一下单接口](https://mp.weixin.qq.com/s/lYPiad1pyZ6ZdqH-sg66eg)

show me the code:

```text
/**
 * [单品优惠券] - 根据订单VO拼接统一下单需要的 detail 参数，此参数用于[单品优惠券]时自动抵扣 <br>
 * 统一下单API(支持单品优惠参数) - 享受了单品优惠的订单不支持部分退款，对账单与普通支付保持一致 <br>
 * 接口地址：https://pay.weixin.qq.com/wiki/doc/api/danpin.php?chapter=9_102&index=2
 * @param orderList 订单list
 * @return 微信支付统一下单 detail 参数
 *
 * @author yclimb
 * @date 2018/9/14
 */
public JSONObject setWxPayUnifiedOrderDetail(List<Order> orderList) {
    if (orderList == null || orderList.isEmpty()) {
        return null;
    }
    // 单品优惠活动detail字段列表说明：
    JSONObject detail = new JSONObject();
    /* 订单原价    cost_price    否    int    608800
       1.商户侧一张小票订单可能被分多次支付，订单原价用于记录整张小票的交易金额。
       2.当订单原价与支付金额不相等，则不享受优惠。
       3.该字段主要用于防止同一张小票分多次支付，以享受多次优惠的情况，正常支付订单不必上传此参数。*/
    // detail.put("cost_price", createTradeVo.getTrade().getTotalPayMoney());
    // 商品小票ID    receipt_id    否    String(32)    wx123    商家小票ID
    // detail.put("receipt_id", "");

    // 单品优惠活动goods_detail字段说明：
    JSONArray goodsDetailList = new JSONArray();

    for (Order order : orderList) {
        JSONObject goodsDetail = new JSONObject();
        // 商品编码    goods_id    是    String(32)    商品编码    由半角的大小写字母、数字、中划线、下划线中的一种或几种组成
        goodsDetail.put("goods_id", order.getProductId());
        // 微信侧商品编码    wxpay_goods_id    否    String(32)    1001    微信支付定义的统一商品编号（没有可不传）
        // goodsDetail.put("wxpay_goods_id", "");
        // 商品名称    goods_name    否    String(256)    iPhone6s 16G    商品的实际名称
        goodsDetail.put("goods_name", order.getProductName());
        // 商品数量    quantity    是    int    1    用户购买的数量
        goodsDetail.put("quantity", order.getItemNum());
        // 商品单价    price    是    int    528800    单位为：分。如果商户有优惠，需传输商户优惠后的单价(例如：用户对一笔100元的订单使用了商场发的纸质优惠券100-50，则活动商品的单价应为原单价-50)
        goodsDetail.put("price", NumberUtil.mul(order.getPayMoney(), 100));

        // 加入单品优惠集合
        goodsDetailList.add(goodsDetail);
    }

    // 单品列表    goods_detail    是    String    示例见下文    单品信息，使用Json数组格式提交
    detail.put("goods_detail", goodsDetailList);

    return detail;
}
```

**发放代金券接口链接**

```text
https://api.mch.weixin.qq.com/mmpaymkttransfers/send_coupon
```

**是否需要证书**

请求需要双向证书。

**调用接口**

用于商户主动调用接口给用户发放代金券的场景，已做防小号处理，给小号发放代金券将返回错误码。

注意：通过接口发放的代金券不会进入微信卡包

接口很简单，需要代金券批次ID和用户openid，代金券批次ID在哪里？每个代金券创建后就会有一个代金券批次ID，在商户平台-营销管理-代金券管理中可以看到。

下面为调用方式：

```text
// 微信支付对象
WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance());

// 调用发送代金券接口
Map<String, String> resultMap = wxPay.sendCoupon(coupon_stock_id, partner_trade_no, openid);
```

微信接口调用：

```text
/**
 * 作用：商户平台-代金券或立减优惠-发放代金券<br>
 * 场景：用于商户主动调用接口给用户发放代金券的场景，已做防小号处理，给小号发放代金券将返回错误码。
 * 注意：通过接口发放的代金券不会进入微信卡包
 * 接口文档地址：https://pay.weixin.qq.com/wiki/doc/api/tools/sp_coupon.php?chapter=12_3&index=4
 *
 * @param coupon_stock_id 代金券批次id
 * @param partner_trade_no 商户单据号
 * @param openid 用户openid
 * @return API返回数据
 * @throws Exception e
 *
 * @author yclimb
 * @date 2018/9/14
 */
public Map<String, String> sendCoupon(String coupon_stock_id, String partner_trade_no, String openid) throws Exception {

    /** 构造请求参数数据 **/
    Map<String, String> data = new HashMap<>();

    // 代金券批次id    coupon_stock_id    是    1757    String    代金券批次id
    data.put("coupon_stock_id", coupon_stock_id);
    // openid记录数    openid_count    是    1    int    openid记录数（目前支持num=1）
    data.put("openid_count", "1");
    // 商户单据号    partner_trade_no    是    1000009820141203515766    String    商户此次发放凭据号（格式：商户id+日期+流水号），商户侧需保持唯一性
    data.put("partner_trade_no", partner_trade_no);
    // 用户openid    openid    是    onqOjjrXT-776SpHnfexGm1_P7iE    String    Openid信息，用户在appid下的唯一标识
    data.put("openid", openid);

    /** 以下参数为非必填参数 **/
    // 操作员    op_user_id    否    10000098    String(32)    操作员帐号, 默认为商户号 可在商户平台配置操作员对应的api权限
    // 设备号    device_info    否         String(32)    微信支付分配的终端设备号
    // 协议版本    version    否    1.0    String(32)    默认1.0
    // 协议类型    type    否    XML    String(32)    XML【目前仅支持默认XML】


    /** 以下四个参数，在 this.fillRequestData 方法中会自动赋值 **/
    // 公众账号ID appid    是    wx5edab3bdfba3dc1c    String(32)    微信为发券方商户分配的公众账号ID，接口传入的所有appid应该为公众号的appid（在mp.weixin.qq.com申请的），不能为APP的appid（在open.weixin.qq.com申请的）。
    // 商户号    mch_id    是    10000098    String(32)    微信为发券方商户分配的商户号
    // 随机字符串    nonce_str    是    1417574675    String(32)    随机字符串，不长于32位
    // 签名    sign    是    841B3002FE2220C87A2D08ABD8A8F791    String(32)    签名参数，详见签名生成算法

    // 微信调用接口
    Map<String, String> resultMap = this.sendCoupon(data);

    WXPayUtil.getLogger().info("wxPay.sendCoupon:" + resultMap);

    return resultMap;
}
```

以上为发放代金券相关代码，下面是查询代金券批次和代金券领取记录接口。 解释下什么叫做代金券批次和代金券记录： 1. 代金券批次：商户平台创建的一个批次代金券，包含x张代金券 2. 代金券：代金券批次下的一张代金券，代金券ID在用户领取代金券后由领取接口获取 3. 代金券记录：用户领券的代金券记录，与代金券1:1，一个批次下有多个领取记录

#### 代金券批次查询

官方文档如下：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/sp_coupon.php?chapter=12_4&index=5
```

是否需要证书：否

请求参数主要为代金券批次id`coupon_stock_id`，下面是调用接口代码：

```text
/**
 * 作用：商户平台-代金券或立减优惠-查询代金券批次<br>
 * 场景：查询代金券批次信息
 * 接口文档地址：https://pay.weixin.qq.com/wiki/doc/api/tools/sp_coupon.php?chapter=12_4&index=5
 *
 * @param coupon_stock_id 代金券批次id
 * @return API返回数据
 * @throws Exception e
 *
 * @author yclimb
 * @date 2018/9/14
 */
public Map<String, String> queryCouponStock(String coupon_stock_id) throws Exception {

    /** 构造请求参数数据 **/
    Map<String, String> data = new HashMap<>();

    // 代金券批次id    coupon_stock_id    是    1757    String    代金券批次id
    data.put("coupon_stock_id", coupon_stock_id);

    /** 以下参数为非必填参数 **/
    // 操作员    op_user_id    否    10000098    String(32)    操作员帐号, 默认为商户号 可在商户平台配置操作员对应的api权限
    // 设备号    device_info    否         String(32)    微信支付分配的终端设备号
    // 协议版本    version    否    1.0    String(32)    默认1.0
    // 协议类型    type    否    XML    String(32)    XML【目前仅支持默认XML】


    /** 以下四个参数，在 this.fillRequestData 方法中会自动赋值 **/
    // 公众账号ID appid    是    wx5edab3bdfba3dc1c    String(32)    微信为发券方商户分配的公众账号ID，接口传入的所有appid应该为公众号的appid（在mp.weixin.qq.com申请的），不能为APP的appid（在open.weixin.qq.com申请的）。
    // 商户号    mch_id    是    10000098    String(32)    微信为发券方商户分配的商户号
    // 随机字符串    nonce_str    是    1417574675    String(32)    随机字符串，不长于32位
    // 签名    sign    是    841B3002FE2220C87A2D08ABD8A8F791    String(32)    签名参数，详见签名生成算法

    // 微信调用接口
    Map<String, String> resultMap = this.queryCouponStock(data);

    WXPayUtil.getLogger().info("wxPay.queryCouponStock:" + resultMap);

    return resultMap;
}
```

此接口主要用于在商家系统主动查询代金券时使用，如果需要实时同步领券数量等，需要定时任务来同步；推荐做法，如果商家自身系统已经发券，就不要使用微信商户平台的发券方式，自身系统发券即可；或者可以做一个手动同步的口子，某一个时间点手动触发同步机制。

#### 查询代金券信息

官方文档如下：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/sp_coupon.php?chapter=12_5&index=6
```

此接口主要作用是查询某个用户的领券状态，代金券状态。 需要三个主要参数：`coupon_id` 代金券id、`stock_id` 批次号、`openid` 用户openid。

调用接口代码如下：

```text
/**
 * 作用：商户平台-代金券或立减优惠-查询代金券信息<br>
 * 场景：查询代金券信息
 * 接口文档地址：https://pay.weixin.qq.com/wiki/doc/api/tools/sp_coupon.php?chapter=12_5&index=6
 *
 * @param coupon_id 代金券id
 * @param stock_id 批次号
 * @param openid 用户openid
 * @return API返回数据
 * @throws Exception e
 *
 * @author yclimb
 * @date 2018/9/14
 */
public Map<String, String> queryCouponsInfo(String coupon_id, String stock_id, String openid) throws Exception {

    /** 构造请求参数数据 **/
    Map<String, String> data = new HashMap<>();

    // 代金券id    coupon_id    是    1565    String    代金券id
    data.put("coupon_id", coupon_id);
    // 用户openid    openid    是    onqOjjrXT-776SpHnfexGm1_P7iE    String    Openid信息，用户在appid下的唯一标识
    data.put("openid", openid);
    // 批次号    stock_id    是    58818    String(32)    代金劵对应的批次号
    data.put("stock_id", stock_id);

    /** 以下参数为非必填参数 **/
    // 操作员    op_user_id    否    10000098    String(32)    操作员帐号, 默认为商户号 可在商户平台配置操作员对应的api权限
    // 设备号    device_info    否         String(32)    微信支付分配的终端设备号
    // 协议版本    version    否    1.0    String(32)    默认1.0
    // 协议类型    type    否    XML    String(32)    XML【目前仅支持默认XML】


    /** 以下四个参数，在 this.fillRequestData 方法中会自动赋值 **/
    // 公众账号ID appid    是    wx5edab3bdfba3dc1c    String(32)    微信为发券方商户分配的公众账号ID，接口传入的所有appid应该为公众号的appid（在mp.weixin.qq.com申请的），不能为APP的appid（在open.weixin.qq.com申请的）。
    // 商户号    mch_id    是    10000098    String(32)    微信为发券方商户分配的商户号
    // 随机字符串    nonce_str    是    1417574675    String(32)    随机字符串，不长于32位
    // 签名    sign    是    841B3002FE2220C87A2D08ABD8A8F791    String(32)    签名参数，详见签名生成算法

    // 微信调用接口
    Map<String, String> resultMap = this.queryCouponsInfo(data);

    WXPayUtil.getLogger().info("wxPay.queryCouponsInfo:" + resultMap);

    return resultMap;
}
```

#### 立减优惠折扣

在商户平台 - 产品中心 - 预充值立减与折扣 中开通功能即可，预充值立减与折扣是微信支付为商户提供的基础营销工具之一，商户可以在商户平台-营销中心配置预充值型立减或折扣，开展营销活动。

可自定义活动标题、减价面额、减价门槛、可用商户、预算、用户领取次数限制，也可以配置指定会员可用、指定某些商品享受优惠等。

此功能不需要开发，创建活动审核开通即生效，在微信支付时自动扣减。

关于立减功能的使用，这里就不多说了，很简单，小伙伴们可以在微信商户平台上阅读一下官方解释，进入产品详情创建一个活动测试一下即可。

#### 结语

这一篇讲解了如何开通代金券和立减优惠折扣，并贴上如何发送代金券、查询代金券等接口的源码，小伙伴需要仔细阅读官方文档，对照本篇文章，应该不会有什么问题。

这里主要是使用了`预充值代金券`、`预充值立减和折扣`，必须先充值足够的`预算金额`才可以使用功能，如果想要`免充值`即可使用，需要开通`免充值代金券`、`免充值立减和折扣`，开通该两项功能需要走`免充值产品功能使用指引`，该功能还需要接口升级，下一篇文章为大家介绍如何`接口升级及开通免充值产品功能`。

如果小伙伴有遇到解决不了的问题，可以关注作者微信公众号，加入讨论群中发出疑问，和小伙伴们一起解决哦～

预告：下一篇文章会讲 `接口升级及开通免充值产品功能`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： ​ ​`​https://github.com/YClimb/wxpay-sdk/blob/master/README.md ​`

关注作者微信公众号，点击下方`讨论群`，扫码即可加入`微信支付讨论群`与小伙伴一起探讨哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x516C;&#x4F17;&#x53F7;](https://img-blog.csdn.net/20180130111432962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvWUNsaW1i/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

