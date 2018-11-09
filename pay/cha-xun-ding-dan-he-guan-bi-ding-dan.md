---
description: 本文是【浅析微信支付】系列文章的第七篇，主要讲解微信商户平台的订单查询和关闭接口的使用。
---

# 查询订单和关闭订单

浅析微信支付系列已经更新六篇了哟～，没有看过的朋友们可以看一下哦。

[浅析微信支付：支付结果通知](https://mp.weixin.qq.com/s/Zr3ldgsMIg_cBrtuS2c7_g)

[浅析微信支付：统一下单接口](https://mp.weixin.qq.com/s/lYPiad1pyZ6ZdqH-sg66eg)

[浅析微信支付：微信公众号网页授权](https://mp.weixin.qq.com/s/BQPn_r5pcwZO3tzqnoTiKg)

声明：这里的`查询订单`、`关闭订单`接口仅适用于 `小程序支付、公共号支付、扫码支付、APP支付`，`刷卡支付`方式此处并不适用。

#### 1、查询订单

以下为微信官方的`查询订单`文档：

```text
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_2
```

**1.1. 应用场景**

该接口提供所有微信支付订单的查询，商户可以通过查询订单接口主动查询订单状态，完成下一步的业务逻辑。

```text
需要调用查询接口的情况：
◆ 当商户后台、网络、服务器等出现异常，商户系统最终未接收到支付通知；
◆ 调用支付接口后，返回系统错误或未知交易状态情况；
◆ 调用刷卡支付API，返回USERPAYING的状态；
◆ 调用关单或撤销接口API之前，需确认支付状态；
```

**1.2. 接口链接**

```text
https://api.mch.weixin.qq.com/pay/orderquery
```

**1.3. 是否需要证书**

不需要

**1.4. 调用接口**

查询订单接口需要使用`微信订单号`或者`商户订单号`来查询，其他参数为商户平台信息的公共参数，为常量，此处省略解释。

```text
微信订单号：transaction_id（微信的订单号，建议优先使用）
商户订单号：out_trade_no（商户系统内部订单号）
```

此两个参数必填其中之一，微信推荐使用`微信订单号`来查询，下面为实现代码：

```text
private void doOrderQuery() {
    System.out.println("查询订单");
    HashMap<String, String> data = new HashMap<String, String>();
    // data.put("out_trade_no", out_trade_no);
    data.put("transaction_id", "4008852001201608221962061594");
    try {
        WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance());
        Map<String, String> r = wxPay.orderQuery(data);
        System.out.println(r);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

`wxPay.orderQuery`方法为封装的sdk方法，具体实现请参考作者github源码。

对于商户关键信息的写入，公共方法为`wxPay.fillRequestData`，实现如下：

```text
/**
 * 向 Map 中添加 appid、mch_id、nonce_str、sign_type、sign <br>
 * 该函数适用于商户适用于统一下单等接口，不适用于红包、代金券接口
 *
 * @param reqData r
 * @return map
 * @throws Exception e
 */
public Map<String, String> fillRequestData(Map<String, String> reqData) throws Exception {
    reqData.put("appid", config.getAppID());
    reqData.put("mch_id", config.getMchID());
    reqData.put("nonce_str", WXPayUtil.generateNonceStr());
    if (SignType.MD5.equals(this.signType)) {
        reqData.put("sign_type", WXPayConstants.MD5);
    } else if (SignType.HMACSHA256.equals(this.signType)) {
        reqData.put("sign_type", WXPayConstants.HMACSHA256);
    }
    reqData.put("sign", WXPayUtil.generateSignature(reqData, config.getKey(), this.signType));
    return reqData;
}
```

以上为查询微信订单的使用方式，具体的返回参数请参考官方文档。

#### 2、关闭订单

以下为微信官方的`关闭订单`文档：

```text
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_3
```

**2.1. 应用场景**

以下情况需要调用关单接口：

```text
商户订单支付失败需要生成新单号重新发起支付，要对原订单号调用关单，避免重复支付；
系统下单后，用户支付超时，系统退出不再受理，避免用户继续，请调用关单接口。
```

注意：订单生成后不能马上调用关单接口，最短调用时间间隔为5分钟。

**2.2. 接口链接**

```text
https://api.mch.weixin.qq.com/pay/closeorder
```

**2.3. 是否需要证书**

不需要

**2.4. 调用接口**

关闭订单接口需要使用`商户订单号`来查询，其他参数为商户平台信息的公共参数，为常量，此处省略解释。

```text
商户订单号：out_trade_no（商户系统内部订单号）
```

PS：关单接口只能使用`微信订单号`来查询，和查询接口不同，下面为实现代码：

```text
private void doOrderClose() {
    System.out.println("关闭订单");
    HashMap<String, String> data = new HashMap<String, String>();
    data.put("out_trade_no", out_trade_no);
    try {
        WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance());
        Map<String, String> r = wxPay.closeOrder(data);
        System.out.println(r);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

关单接口的公共参数设置和查询订单一致，这里就不重复解释了，具体的返回参数请参考微信官方文档。

PS：关单接口可能会调用失败，已支付、已关闭等场景，所以需要开发者注意官方文档中的错误码，对异常情况进行处理。

#### 结语

以上为`查询订单`、`关闭订单`的调用方式，如果是`刷卡支付`方式，他的关闭订单接口为`撤销订单:reverse`，在作者sdk源码中也有具体的实现方式。

预告：下一篇文章 `申请退款和退款回调接口`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下：

```text
​https://github.com/YClimb/wxpay-sdk/blob/master/README.md
```

加作者私人微信，作者微信号如下 `yclimb`，标明 `微信支付` 可拉入微信支付讨论群与小伙伴一起探讨哦，一定要标明 `微信支付` 哦～

到此本文就结束了，关注公众号查看更多推送！！！

![](../.gitbook/assets/er-wei-ma.jpg)

