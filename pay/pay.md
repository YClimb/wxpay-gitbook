---
description: 本文是【浅析微信支付】系列文章的第五篇，主要讲解如何调用统一下单接口生成预支付单及调起支付页面。
---

# 统一下单接口

浅析微信支付系列已经更新四篇了哟～，没有看过的朋友们可以看一下哦。

[浅析微信支付：微信公众号网页授权](https://mp.weixin.qq.com/s/BQPn_r5pcwZO3tzqnoTiKg)

[浅析微信支付：开发前的准备](https://mp.weixin.qq.com/s/mkxukWU4aMw92NxnL91dng)

上面是本文的前置文章，有前面几篇文章的基础以后看会更加明了，如果已经看过的小伙伴可以忽略。

#### 1、什么是\[统一下单接口\]？

首先我们要明白这个问题，需要先行看一下微信的官方文档： [https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9\_1](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_1)

官方解释如下：

```text
除被扫支付场景以外，商户系统先调用该接口在微信支付服务后台生成预支付交易单，返回正确的预支付交易会话标识后再按扫码、JSAPI、APP等不同场景生成交易串调起支付。
```

什么意思？简单理解：就是说我们要在调起微信支付窗口之前，需要先生成一个 `预支付交易单`，这个单子相当于和我们自身系统的 `支付交易单` 一一对应，也就是我们每次支付需要记录的订单支付交易单。

从上面我们可以得到，在调用此接口之前，首先，我们系统中肯定已经需要有以下步骤：订单提交 -&gt; 生成订单 -&gt; 生成订单对应的支付单 -&gt; 调用统一下单接口

好了，假设系统现在已经生成支付交易单，准备调用统一下单接口，我们来看一下具体的实现方式。

PS：调用统一下单接口时，需要注意的是必须传入异步接收微信支付结果通知的回调地址，通知url必须为外网可访问的url，不能携带参数。示例如下：

`https://xxx.com/v1/weixin/pay/wxnotify`

#### 2、调用接口

示例代码：

```text
/**
 * [微信支付统一下单] - 保存调用的相关记录 <p>
 * [微信支付小程序] - 返回支付唤醒参数
 * @param payment 付款对象
 * @param user 当前用户
 * @return map
 * @throws Exception e
 *
 * @author yclimb
 * @date 2018/6/15
 */
public Map<String, String> saveWxPayUnifiedOrder(Payment payment, User user) throws Exception {
    if (payment == null || user == null) {
        return null;
    }

    // 1.调用微信统一下单接口
    WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance());
    Map<String, String> resultMap = wxPay.unifiedOrder(...);

    // 1.1.记录付款流水
    ...

    // 下单失败，进行处理
    if (WXPayConstants.FAIL.equals(resultMap.get(WXPayConstants.RETURN_CODE)) ||
            WXPayConstants.FAIL.equals(resultMap.get(WXPayConstants.RESULT_CODE))) {

        // 处理结果返回，无需继续执行
        resultMap.put(WXPayConstants.RESULT_CODE, WXPayConstants.FAIL);
        resultMap.put(WXPayConstants.ERR_CODE_DES, resultMap.get(WXPayConstants.RETURN_MSG));
        return resultMap;
    }

    // 1.2.获取prepay_id、nonce_str
    String prepay_id = resultMap.get("prepay_id");
    String nonce_str = resultMap.get("nonce_str");

    // 2.根据微信统一下单接口返回数据组装微信支付参数，返回结果
    return wxPay.chooseWXPayMap(prepay_id, nonce_str);
}
```

以上为一个调用统一下单的接口代码，主要在于以下两句：

```text
// 实例化一个微信支付对象，使用单例配置的方式
WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance());

// 直接调用统一下单接口
Map<String, String> resultMap = wxPay.unifiedOrder(...);
```

微信支付对象`WXPay`统一下单接口：

```text
/**
 * 作用：统一下单<br>
 * 场景：商户在小程序中先调用该接口在微信支付服务后台生成预支付交易单，返回正确的预支付交易后调起支付。
 * 接口链接：URL地址：https://api.mch.weixin.qq.com/pay/unifiedorder
 * 是否需要证书：否
 * 接口文档地址：https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_1
 *
 * @param notify_url       公众号用户openid
 * @param body             商品简单描述，该字段请按照规范传递，例：腾讯充值中心-QQ会员充值
 * @param out_trade_no     商户系统内部订单号，要求32个字符内，只能是数字、大小写字母_-|*且在同一个商户号下唯一
 * @param total_fee        订单总金额，传入参数单位为：元
 * @param spbill_create_ip APP和网页支付提交用户端ip，Native支付填调用微信支付API的机器IP
 * @param goods_tag        订单优惠标记，用于区分订单是否可以享受优惠
 * @param detail           商品详情    ，单品优惠活动该字段必传
 * @param timeStart        订单生成时间，格式为yyyyMMddHHmmss
 * @param timeExpire       订单失效时间，格式为yyyyMMddHHmmss，如2009年12月27日9点10分10秒表示为20091227091010
 * @return API返回数据
 * @throws Exception e
 */
public Map<String, String> unifiedOrder(String notify_url, String openid, String body, String out_trade_no, String total_fee, String spbill_create_ip, String goods_tag, String detail, Date timeStart, Date timeExpire) throws Exception {

    /** 构造请求参数数据 **/
    Map<String, String> data = new HashMap<>();

    // 字段名    变量名    必填    类型    示例值    描述
    // 标价币种    fee_type    否    String(16)    CNY    符合ISO 4217标准的三位字母代码，默认人民币：CNY，详细列表请参见货币类型
    data.put("fee_type", WXPayConstants.FEE_TYPE_CNY);
    // 通知地址    notify_url    是    String(256)    http://www.weixin.qq.com/wxpay/pay.php    异步接收微信支付结果通知的回调地址，通知url必须为外网可访问的url，不能携带参数。
    data.put("notify_url", notify_url);
    // 交易类型    trade_type    是    String(16)    JSAPI    小程序取值如下：JSAPI，详细说明见参数规定
    data.put("trade_type", WXPayConstants.TRADE_TYPE);
    // 用户标识    openid    否    String(128)    oUpF8uMuAJO_M2pxb1Q9zNjWeS6o    trade_type=JSAPI，此参数必传，用户在商户appid下的唯一标识。openid如何获取，可参考【获取openid】。
    data.put("openid", openid);
    // 商品描述    body    是    String(128)    腾讯充值中心-QQ会员充值 商品简单描述，该字段请按照规范传递，具体请见参数规定
    data.put("body", body);
    // 商户订单号    out_trade_no    是    String(32)    20150806125346    商户系统内部订单号，要求32个字符内，只能是数字、大小写字母_-|*且在同一个商户号下唯一。详见商户订单号
    data.put("out_trade_no", out_trade_no);
    // 标价金额    total_fee    是    Int    88    订单总金额，单位为分，详见支付金额
    // 默认单位为分，系统是元，所以需要*100
    data.put("total_fee", String.valueOf(new BigDecimal(total_fee).multiply(new BigDecimal(100)).setScale(2, BigDecimal.ROUND_HALF_UP).intValue()));
    // 终端IP    spbill_create_ip    是    String(16)    123.12.12.123    APP和网页支付提交用户端ip，Native支付填调用微信支付API的机器IP。
    data.put("spbill_create_ip", spbill_create_ip);

    /** 以下参数为非必填参数 **/
    // 订单优惠标记    goods_tag    否    String(32)    WXG    订单优惠标记，使用代金券或立减优惠功能时需要的参数，说明详见代金券或立减优惠
    if (StringUtils.isNotBlank(goods_tag)) {
        data.put("goods_tag", goods_tag);
    }
    // 商品详情    detail    否    String(6000)         商品详细描述，对于使用单品优惠的商户，改字段必须按照规范上传，详见“单品优惠参数说明”
    if (StringUtils.isNotBlank(detail)) {
        data.put("detail", detail);
        // 接口版本号 新增字段，接口版本号，区分原接口，默认填写1.0。入参新增version后，则支付通知接口也将返回单品优惠信息字段promotion_detail，请确保支付通知的签名验证能通过。
        data.put("version", "1.0");
    }
    // 设备号    device_info    否    String(32)    013467007045764    自定义参数，可以为终端设备号(门店号或收银设备ID)，PC网页或公众号内支付可以传"WEB"
    data.put("device_info", "WEB");

    // 交易起始时间    time_start    否    String(14)    20091225091010    订单生成时间，格式为yyyyMMddHHmmss，如2009年12月25日9点10分10秒表示为20091225091010。其他详见时间规则
    data.put("time_start", DateTimeUtil.getTimeShortString(timeStart));
    // 交易结束时间    time_expire    否    String(14)    20091227091010    订单失效时间，格式为yyyyMMddHHmmss，如2009年12月27日9点10分10秒表示为20091227091010。
    // 订单失效时间是针对订单号而言的，由于在请求支付的时候有一个必传参数prepay_id只有两小时的有效期，所以在重入时间超过2小时的时候需要重新请求下单接口获取新的prepay_id。其他详见时间规则,建议：最短失效时间间隔大于1分钟
    data.put("time_expire", DateTimeUtil.getTimeShortString(timeExpire));
    /*// 商品ID    product_id    否    String(32)    12235413214070356458058    trade_type=NATIVE时（即扫码支付），此参数必传。此参数为二维码中包含的商品ID，商户自行定义。
    data.put("product_id", null);
    // 指定支付方式    limit_pay    否    String(32)    no_credit    上传此参数no_credit--可限制用户不能使用信用卡支付
    data.put("limit_pay", null);
    // 附加数据    attach    否    String(127)    深圳分店    附加数据，在查询API和支付通知中原样返回，可作为自定义参数使用。
    data.put("attach", null);*/

    /** 以下五个参数，在 this.fillRequestData 方法中会自动赋值 **/
    /*// 小程序ID    appid    是    String(32)    wxd678efh567hg6787    微信分配的小程序ID
    data.put("appid", WXPayConstants.APP_ID);
    // 商户号    mch_id    是    String(32)    1230000109    微信支付分配的商户号
    data.put("mch_id", WXPayConstants.MCH_ID);
    // 随机字符串    nonce_str    是    String(32)    5K8264ILTKCH16CQ2502SI8ZNMTM67VS    随机字符串，长度要求在32位以内。推荐随机数生成算法
    data.put("nonce_str", nonce_str);
    // 签名类型    sign_type    否    String(32)    MD5    签名类型，默认为MD5，支持HMAC-SHA256和MD5。
    data.put("sign_type", WXPayConstants.MD5);
    // 签名    sign    是    String(32)    C380BEC2BFD727A4B6845133519F3AD6    通过签名算法计算得出的签名值，详见签名生成算法
    data.put("sign", sign);*/

    // 微信统一下单接口请求地址
    Map<String, String> resultMap = this.unifiedOrder(data);

    WXPayUtil.getLogger().info("wxPay.unifiedOrder:" + resultMap);

    return resultMap;
}
```

以上代码详细说明了每个字段的含义，具体的代码可以看一下作者的github，文末有对应的地址。

下面说一个特殊情况，在我们支付的时候，有时候用户会取消支付，等一段时间再重新调起，这时候，需要用到另外一个方法，那就是二次支付，所以，在我们数据库中，必须保存两个字段，用于二次支付时使用：预支付ID`prepay_id`、随机字符串`nonce_str`，此两个参数可以生成微信支付调起时需要的验证签名。

以下为二次调用时的代码：

```text
/**
 * 根据付款单号查询微信付款参数
 * @param relationId 交易编号
 * @param type 支付类型
 * @return payment
 *
 * @author yclimb
 * @date 2018/6/28
 */
public Map<String, String> queryChooseWXPayMapByRelationId(Integer relationId, String type) throws Exception {

        // 微信支付对象
        WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance());

        // 1.查询付款对象
        Payment payment = this.queryPaymentByRelationId(relationId, type);

        // 2.根据微信统一下单接口返回数据组装微信支付参数，返回结果
        return wxPay.chooseWXPayMap(payment.getPrepayId(), payment.getNonceStr());
    }
```

此时我们已经调用微信统一下单接口成功，并为我们返回了需要的参数，下一步需要组装为微信支付调起时前端需要的参数。

#### 生成支付签名

下面为组装支付签名的代码：

```text
/**
 * 作用：生成微信支付所需参数，微信支付二次签名<br>
 * 场景：根据微信统一下单接口返回的 prepay_id 生成微信支付所需的参数
 * 接口文档地址：https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6
 *
 * @param prepay_id 预支付id
 * @param nonce_str 随机字符串
 * @return 支付方法调用所需参数map
 * @throws Exception e
 */
public Map<String, String> chooseWXPayMap(String prepay_id, String nonce_str) throws Exception {

    // 支付方法调用所需参数map
    Map<String, String> chooseWXPayMap = new HashMap<>();
    chooseWXPayMap.put("appId", config.getAppID());
    chooseWXPayMap.put("timeStamp", String.valueOf(WXPayUtil.getCurrentTimestamp()));
    chooseWXPayMap.put("nonceStr", nonce_str);
    chooseWXPayMap.put("package", "prepay_id=" + prepay_id);
    chooseWXPayMap.put("signType", WXPayConstants.MD5);

    WXPayUtil.getLogger().info("wxPay.chooseWXPayMap:" + chooseWXPayMap.toString());

    // 生成支付签名
    String paySign = WXPayUtil.generateSignature(chooseWXPayMap, config.getKey());
    chooseWXPayMap.put("paySign", paySign);

    WXPayUtil.getLogger().info("wxPay.paySign:" + paySign);

    return chooseWXPayMap;
}
```

组装好需要的参数以后，就可以调起微信支付窗口了，如果是微信公众号支付，需要使用以下的方式调起微信支付：

```text
function weixinConfig(appid, timestamp, noncestr, signature) {
    wx.config({
        debug : false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
        // debug : true, 
        appId :appid, // 必填，公众号的唯一标识
        timestamp : timestamp, // 必填，生成签名的时间戳
        nonceStr : noncestr, // 必填，生成签名的随机串
        signature : signature,// 必填，签名，见附录1
        jsApiList: [
            'checkJsApi',
            'chooseWXPay' // 必须增加此参数，用户微信支付功能
        ]
        // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
    });
}

wx.chooseWXPay({
    appId: appId,
    timestamp: timeStamp, // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符
    nonceStr: nonceStr, // 支付签名随机串，不长于 32 位
    package: package, // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=\*\*\*）
    signType: signType, // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
    paySign: paySign, // 支付签名
    success: function (res) {
        // 支付成功后的回调函数
        alert('支付成功！');
    },
    cancel: function (res) {
        // 支付取消
    },
    fail: function (res) {
        // 支付失败
        alert("支付失败！");
    }
});
```

PS：小程序调用方法类似，参数一致。

#### 结语

以上就是微信支付统一下单接口的调用方式了，具体的源码可以参考作者github，最好在开发之前先通读一遍微信官方文档，此时再使用作者源码开发事半功倍，更易理解。

预告：下一篇文章，作者将讲 `支付结果通知`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： ​`https://github.com/YClimb/wxpay-sdk/blob/master/README.md`

加作者私人微信，作者微信号如下 `yclimb`，标明 `微信支付` 可拉入微信支付讨论群与小伙伴一起探讨哦，一定要标明 `微信支付` 哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x5FAE;&#x4FE1;&#x516C;&#x4F17;&#x53F7;](../.gitbook/assets/er-wei-ma.jpg)

