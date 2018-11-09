---
description: 本文是【浅析微信支付】系列文章的第八篇，主要讲解商户如何处理微信申请退款、退款回调、查询退款接口，其中有一些坑的地方，会着重强调。
---

# 申请退款、退款回调接口、查询退款

浅析微信支付系列已经更新七篇了哟～，没有看过的朋友们可以看一下哦。

[浅析微信支付：查询订单和关闭订单](https://mp.weixin.qq.com/s/SG4sTHsUKKJF-_Qgpjh0jA)

[浅析微信支付：支付结果通知](https://mp.weixin.qq.com/s/Zr3ldgsMIg_cBrtuS2c7_g)

[浅析微信支付：统一下单接口](https://mp.weixin.qq.com/s/lYPiad1pyZ6ZdqH-sg66eg)

在实际场景中，申请退款和退款回调接口是比较常用到的微信支付接口，这里我们会讲`原路返回`方式的退款，还有的是使用直接为用户`付款到零钱`、`现金红包`等方式来退款，此种情况主要会出现在客服退款时，不是全部退款的情况，也有的会出现在使用了`微信代金券-单品券`的时候，因为单品券不能部分退款，所以只能走企业付款用户的方式，以下我们主要讲`原路返回`退款。

PS：原路返回的意思就是，从你支付时的关联支付单中扣款，微信会记录相关数据，可以在客户端通知中展示。

#### 1、申请退款接口

以下为微信官方的`申请退款`文档：

```text
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_4
```

**1.1. 应用场景**

当交易发生之后一段时间内，由于买家或者卖家的原因需要退款时，卖家可以通过退款接口将支付款退还给买家，微信支付将在收到退款请求并且验证成功之后，按照退款规则将支付款按原路退到买家帐号上。

```text
注意：
1、交易时间超过一年的订单无法提交退款
2、微信支付退款支持单笔交易分多次退款，多次退款需要提交原支付订单的商户订单号和设置不同的退款单号。申请退款总金额不能超过订单金额。 一笔退款失败后重新提交，请不要更换退款单号，请使用原商户退款单号
3、请求频率限制：150qps，即每秒钟正常的申请退款请求次数不超过150次
错误或无效请求频率限制：6qps，即每秒钟异常或错误的退款申请请求不超过6次
4、每个支付订单的部分退款次数不能超过50次
```

PS：以上限制一般情况下不会出现，但我们也必须写入系统异常场景处理中，请求频率可以使用队列或增加延迟等方式来处理，部分退款此时不要超过微信的限制。

**1.2. 接口链接**

```text
https://api.mch.weixin.qq.com/secapi/pay/refund
```

**1.3. 是否需要证书**

请求需要双向证书。

PS：关于微信证书，可以在 \[商户平台-账户中心-API安全\] 去下载，此证书很多支付接口均需要使用，请将证书地址配置为常量，具体实现可以参考作者github源码。

**1.4. 调用接口**

先看源码，如下：

```text
/**
 * [微信退款接口] - 保存调用的相关记录
 * @param refundPayment 退款订单的支付记录
 * @param tradePayment 历史付款单
 * @return map
 * @throws Exception e
 *
 * @author yclimb
 * @date 2018/6/21
 */
public Map<String,String> saveWxPayRefund(Payment refundPayment, Payment tradePayment) throws Exception {
    if (refundPayment == null || tradePayment == null) {
        return null;
    }

    // 微信订单号/商户订单号，必须传入其中一个，此处默认传入商户订单号
    // 微信订单号，微信生成的订单号，在支付通知中有返回
    // String transaction_id = null;
    // 商户订单号，商户系统内部订单号，要求32个字符内，只能是数字、大小写字母_-|*@ ，且在同一个商户号下唯一。
    String out_trade_no = tradePayment.getFlowNumer();
    // 商户退款单号，商户系统内部的退款单号，商户系统内部唯一，只能是数字、大小写字母_-|*@ ，同一退款单号多次请求只退一笔。
    String out_refund_no = refundPayment.getFlowNumer();
    // 订单总金额，传入参数单位为：元
    String total_fee = String.valueOf(tradePayment.getAmount());
    // 退款总金额，订单总金额，传入参数单位为：元
    String refund_fee = String.valueOf(refundPayment.getAmount());
    // 退款原因，若商户传入，会在下发给用户的退款消息中体现退款原因
    String refund_desc = refundPayment.getBody();

    // 微信支付对象
    WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance());

    // 微信退款接口
    Map<String, String> resultMap = wxPay.refund(refundUrl, null, out_trade_no, out_refund_no, total_fee, refund_fee, refund_desc);
    logger.info("saveWxPayRefund:resultMap:" + resultMap.toString());

    // 记录付款流水


    // 下单失败，进行处理
    if (WXPayConstants.FAIL.equals(resultMap.get(WXPayConstants.RETURN_CODE)) ||
            WXPayConstants.FAIL.equals(resultMap.get(WXPayConstants.RESULT_CODE))) {

        // 处理结果返回，无需继续执行
        resultMap.put(WXPayConstants.RESULT_CODE, WXPayConstants.FAIL);
        resultMap.put(WXPayConstants.ERR_CODE_DES, resultMap.get(WXPayConstants.RETURN_MSG));
        return resultMap;
    }

    return resultMap;
}
```

以上为sdk退款调用示例代码，有几个参数需要我们注意：

| 字段名 | 变量名 | 必填 | 类型 | 描述 |  |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 微信订单号 | transaction\_id | 是 | String\(32\) | 微信生成的订单号，在支付通知中有返回 |  |
| 商户订单号 | out\_trade\_no | 是 | String\(32\) | 商户系统内部订单号，要求32个字符内，只能是数字、大小写字母\_- | \*@ ，且在同一个商户号下唯一。 |
| 商户退款单号 | out\_refund\_no | 是 | String\(64\) | 商户系统内部的退款单号，商户系统内部唯一，只能是数字、大小写字母\_- | \*@ ，同一退款单号多次请求只退一笔。 |
| 退款金额 | refund\_fee | 是 | Int | 退款总金额，订单总金额，单位为分，只能为整数 |  |
| 退款结果通知url | notify\_url | 否 | String\(256\) | 异步接收微信支付退款结果通知的回调地址，通知URL必须为外网可访问的url，不允许带参数，如果参数中传了notify\_url，则商户平台上配置的回调地址将不会生效。 |  |

PS：推荐以上的参数都必填，`notify_url`参数可配置为环境常量，根据环境的不同配置调用不会的回调地址。

下面为具体的实际sdk`wxPay.refund`调用代码：

```text
/**
 * 作用：申请退款<br>
 * 场景：当交易发生之后一段时间内，由于买家或者卖家的原因需要退款时，卖家可以通过退款接口将支付款退还给买家，
 * 微信支付将在收到退款请求并且验证成功之后，按照退款规则将支付款按原路退到买家帐号上。
 * 接口文档地址：https://pay.weixin.qq.com/wiki/doc/api/wxa/wxa_api.php?chapter=9_4
 *
 * @param notify_url     回调地址
 * @param transaction_id 微信生成的订单号，在支付通知中有返回
 * @param out_trade_no   商户系统内部订单号，要求32个字符内，只能是数字、大小写字母_-|*@ ，且在同一个商户号下唯一。
 * @param out_refund_no  商户系统内部的退款单号，商户系统内部唯一，只能是数字、大小写字母_-|*@ ，同一退款单号多次请求只退一笔。
 * @param total_fee      订单总金额，传入参数单位为：元
 * @param refund_fee     退款总金额，订单总金额，传入参数单位为：元
 * @param refund_desc    退款原因，若商户传入，会在下发给用户的退款消息中体现退款原因
 * @return API返回数据
 * @throws Exception e
 */
public Map<String, String> refund(String notify_url, String transaction_id, String out_trade_no, String out_refund_no,
                                  String total_fee, String refund_fee, String refund_desc) throws Exception {

    /** 构造请求参数数据 **/
    Map<String, String> data = new HashMap<>();

    // 变量名        字段名    必填    类型    示例值    描述
    // 微信订单号    二选一    String(32)    1.21775E+27    微信生成的订单号，在支付通知中有返回
    if (transaction_id != null) {
        data.put("transaction_id", transaction_id);
    }
    // 商户订单号    String(32)    1.21775E+27    商户系统内部订单号，要求32个字符内，只能是数字、大小写字母_-|*@ ，且在同一个商户号下唯一。
    data.put("out_trade_no", out_trade_no);
    // 商户退款单号    是    String(64)    1.21775E+27    商户系统内部的退款单号，商户系统内部唯一，只能是数字、大小写字母_-|*@ ，同一退款单号多次请求只退一笔。
    data.put("out_refund_no", out_refund_no);
    // 订单金额    是    Int    100    订单总金额，单位为分，只能为整数，详见支付金额
    data.put("total_fee", String.valueOf(new BigDecimal(total_fee).multiply(new BigDecimal(100)).setScale(2, BigDecimal.ROUND_HALF_UP).intValue()));
    // 退款金额    是    Int    100    退款总金额，订单总金额，单位为分，只能为整数，详见支付金额
    // 默认单位为分，系统是元，所以需要*100
    data.put("refund_fee", String.valueOf(new BigDecimal(refund_fee).multiply(new BigDecimal(100)).setScale(2, BigDecimal.ROUND_HALF_UP).intValue()));
    // 退款原因    否    String(80)    商品已售完    若商户传入，会在下发给用户的退款消息中体现退款原因
    data.put("refund_desc", refund_desc);
    // 货币种类    否    String(8)    CNY    货币类型，符合ISO 4217标准的三位字母代码，默认人民币：CNY，其他值列表详见货币类型
    data.put("refund_fee_type", WXPayConstants.FEE_TYPE_CNY);
    // 退款结果通知url    否    String(256)    https://weixin.qq.com/notify/    异步接收微信支付退款结果通知的回调地址，通知URL必须为外网可访问的url，不允许带参数,如果参数中传了notify_url，则商户平台上配置的回调地址将不会生效。
    data.put("notify_url", notify_url);

    /** 以下参数为非必填参数 **/
    // 退款资金来源    否    String(30)    REFUND_SOURCE_RECHARGE_FUNDS    仅针对老资金流商户使用;REFUND_SOURCE_UNSETTLED_FUNDS---未结算资金退款（默认使用未结算资金退款）;REFUND_SOURCE_RECHARGE_FUNDS---可用余额退款
    // data.put("refund_account", null);


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

    // 微信退款接口
    Map<String, String> resultMap = this.refund(data);

    WXPayUtil.getLogger().info("wxPay.refund:" + resultMap);

    return resultMap;
}
```

以上已经详细说明的具体的字段含义，有不明白的同学可以查看微信的官方文档，具体的源码可以查看作者的github。

这里有一个比较需要注意的点，在我们调用退款之后，会返回一些异常处理情况，官方文档中收录了一系列错误码code，我们可以在系统中对其进行处理，这里就不细说了。

#### 2、退款回调接口

以下为微信官方的`退款结果通知`文档：

```text
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_16&index=10
```

**2.1. 应用场景**

当商户申请的退款有结果后，微信会把相关结果发送给商户，商户需要接收处理，并返回应答。 对后台通知交互时，如果微信收到商户的应答不是成功或超时，微信认为通知失败，微信会通过一定的策略定期重新发起通知，尽可能提高通知的成功率，但微信不保证通知最终能成功。

（通知频率为15/15/30/180/1800/1800/1800/1800/3600，单位：秒）

注意：同样的通知可能会多次发送给商户系统。商户系统必须能够正确处理重复的通知。 推荐的做法是，当收到通知进行处理时，首先检查对应业务数据的状态，判断该通知是否已经处理过，如果没有处理过再进行处理，如果处理过直接返回结果成功。在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱。

特别说明：退款结果对重要的数据进行了加密，商户需要用商户秘钥进行解密后才能获得结果通知的内容

**2.2. 接口链接**

在申请退款接口中上传参数“notify\_url”以开通该功能 如果链接无法访问，商户将无法接收到微信通知。 通知url必须为直接可访问的url，不能携带参数。

```text
示例：notify_url：“https://pay.weixin.qq.com/wxpay/pay.action”
```

**2.3. 解密方式**

```text
解密步骤如下： 
（1）对加密串A做base64解码，得到加密串B
（2）对商户key做md5，得到32位小写key* ( key设置路径：微信商户平台-->账户设置-->API安全-->密钥设置 )
（3）用key*对加密串B做AES-256-ECB解密（PKCS7Padding）
```

PS：特别注意，如果要进行微信AES解密，因为GJ的进口管制限制，Java发布的运行环境包中的加解密有一定的限制。默认不允许256位密钥的AES加解密，解决方法就是修改策略文件，我们需要从官方网站下载无限制权限策略文件，注意自己JDK的版本别下错了。

jdk8的jce下载地址：`https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html`

将local\_policy.jar和US\_export\_policy.jar这两个文件替换%JRE\_HOME%\lib\security和%JDK\_HOME%\jre\lib\security下原来的文件，注意先备份原文件。

如果是jdk8，可能会遇到安全目录下有`policy`文件夹的情况，拿作者的电脑举例，jdk路径为`/opt/jdk1.8.0_152/jre/lib/security/policy`，此目录下有两个子文件夹`limited`、`limited`，需要替换`limited`文件夹下的文件 `local_policy.jar`、`US_export_policy.jar`这两个，最好先备份哦～！！！替换后重启项目即可。

**2.4. 调用接口**

因为退款回调接口是咋们系统被动接收微信的消息，所以此处和支付回调接口一致，也是使用了流的方式，格式为xml，下面我们来看代码：

```text
/**
 * 退款结果通知
 * <p>
 * 在申请退款接口中上传参数“notify_url”以开通该功能
 * 如果链接无法访问，商户将无法接收到微信通知。
 * 通知url必须为直接可访问的url，不能携带参数。示例：notify_url：“https://pay.weixin.qq.com/wxpay/pay.action”
 * <p>
 * 当商户申请的退款有结果后，微信会把相关结果发送给商户，商户需要接收处理，并返回应答。
 * 对后台通知交互时，如果微信收到商户的应答不是成功或超时，微信认为通知失败，微信会通过一定的策略定期重新发起通知，尽可能提高通知的成功率，但微信不保证通知最终能成功。
 * （通知频率为15/15/30/180/1800/1800/1800/1800/3600，单位：秒）
 * 注意：同样的通知可能会多次发送给商户系统。商户系统必须能够正确处理重复的通知。
 * 推荐的做法是，当收到通知进行处理时，首先检查对应业务数据的状态，判断该通知是否已经处理过，如果没有处理过再进行处理，如果处理过直接返回结果成功。在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱。
 * 特别说明：退款结果对重要的数据进行了加密，商户需要用商户秘钥进行解密后才能获得结果通知的内容
 * @param request req
 * @param response resp
 * @return res xml
 *
 * @author yclimb
 * @date 2018/6/21
 */
@ApiOperation(value = "微信支付|微信退款回调接口", httpMethod = "POST", notes = "该链接是通过【微信退款API】中提交的参数notify_url设置，如果参数中传了notify_url，则商户平台上配置的回调地址将不会生效。")
@RequestMapping("/refund")
public void refund(HttpServletRequest request, HttpServletResponse response) {

    String resXml = "";
    InputStream inStream;
    try {

        inStream = request.getInputStream();
        ByteArrayOutputStream outSteam = new ByteArrayOutputStream();
        byte[] buffer = new byte[1024];
        int len = 0;
        while ((len = inStream.read(buffer)) != -1) {
            outSteam.write(buffer, 0, len);
        }
        WXPayUtil.getLogger().info("refund:微信退款----start----");

        // 获取微信调用我们notify_url的返回信息
        String result = new String(outSteam.toByteArray(), "utf-8");
        WXPayUtil.getLogger().info("refund:微信退款----result----=" + result);

        // 关闭流
        outSteam.close();
        inStream.close();

        // xml转换为map
        Map<String, String> map = WXPayUtil.xmlToMap(result);
        if (WXPayConstants.SUCCESS.equalsIgnoreCase(map.get(WXPayConstants.RETURN_CODE))) {

            WXPayUtil.getLogger().info("refund:微信退款----返回成功");


            /** 以下字段在return_code为SUCCESS的时候有返回： **/
            // 加密信息：加密信息请用商户秘钥进行解密，详见解密方式
            String req_info = map.get("req_info");

            /**
             * 解密方式
             * 解密步骤如下：
             * （1）对加密串A做base64解码，得到加密串B
             * （2）对商户key做md5，得到32位小写key* ( key设置路径：微信商户平台(pay.weixin.qq.com)-->账户设置-->API安全-->密钥设置 )
             * （3）用key*对加密串B做AES-256-ECB解密（PKCS7Padding）
             */
            String resultStr = AESUtil.decryptData(req_info);

            // WXPayUtil.getLogger().info("refund:解密后的字符串:" + resultStr);
            Map<String, String> aesMap = WXPayUtil.xmlToMap(resultStr);


            /** 以下为返回的加密字段： **/
            //    商户退款单号    是    String(64)    1.21775E+27    商户退款单号
            String out_refund_no = aesMap.get("out_refund_no");
            //    退款状态    是    String(16)    SUCCESS    SUCCESS-退款成功、CHANGE-退款异常、REFUNDCLOSE—退款关闭
            String refund_status = aesMap.get("refund_status");
            //    商户订单号    是    String(32)    1.21775E+27    商户系统内部的订单号
            String out_trade_no = aesMap.get("out_trade_no");
            /*//    微信订单号    是    String(32)    1.21775E+27    微信订单号
            String transaction_id = null;
            //    微信退款单号    是    String(32)    1.21775E+27    微信退款单号
            String refund_id = null;
            //    订单金额    是    Int    100    订单总金额，单位为分，只能为整数，详见支付金额
            String total_fee = null;
            //    应结订单金额    否    Int    100    当该订单有使用非充值券时，返回此字段。应结订单金额=订单金额-非充值代金券金额，应结订单金额<=订单金额。
            String settlement_total_fee = null;
            //    申请退款金额    是    Int    100    退款总金额,单位为分
            String refund_fee = null;
            //    退款金额    是    Int    100    退款金额=申请退款金额-非充值代金券退款金额，退款金额<=申请退款金额
            String settlement_refund_fee = null;*/

            // 退款是否成功
            if (!WXPayConstants.SUCCESS.equals(refund_status)) {
                resXml = resFailXml;
            } else {
                // 通知微信.异步确认成功.必写.不然会一直通知后台.八次之后就认为交易失败了.
                resXml = resSuccessXml;
                isSuccess = true;
            }

            // 根据付款单号查询付款记录 out_refund_no

            // 付款记录修改 & 记录付款日志
            if (payment != null) {
                WXPayUtil.getLogger().error("refund:微信支付回调：修改支付单");
            } else {
                WXPayUtil.getLogger().error("refund:微信支付回调：找不到对应的支付单");
            }


        } else {
            WXPayUtil.getLogger().error("refund:支付失败,错误信息：" + map.get(WXPayConstants.RETURN_MSG));
            resXml = resFailXml;
        }

    } catch (Exception e) {
        WXPayUtil.getLogger().error("refund:微信退款回调发布异常：", e);
    } finally {
        try {
            // 处理业务完毕
            BufferedOutputStream out = new BufferedOutputStream(response.getOutputStream());
            out.write(resXml.getBytes());
            out.flush();
            out.close();
        } catch (IOException e) {
            WXPayUtil.getLogger().error("refund:微信退款回调发布异常:out：", e);
        }
    }
}
```

以上代码详细解释了如何接收微信回调数据和解码数据，具体的`AESUtil.decryptData(req_info)`请参考作者源码，文末有地址，这里就不细讲了。

具体的退款接收参数请参考微信官方文档，需要注意的是`商户退款单号`和`微信退款单号`，此两个参数是修改和记录退款的必要凭证。

#### 3、查询退款

以下为微信官方的`查询退款`文档：

```text
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_5
```

**3.1. 应用场景**

提交退款申请后，通过调用该接口查询退款状态。退款有一定延时，用零钱支付的退款20分钟内到账，银行卡支付的退款3个工作日后重新查询退款状态。

注意：如果单个支付订单部分退款次数超过20次请使用退款单号查询

**3.2. 接口链接**

```text
https://api.mch.weixin.qq.com/pay/refundquery
```

**3.3. 是否需要证书**

不需要

**3.4. 调用接口**

注意：当一个订单部分退款超过10笔后，商户用微信订单号或商户订单号调退款查询API查询退款时，默认返回前10笔和`total_refund_count`（订单总退款次数）。商户需要查询同一订单下超过10笔的退款单时，可传入订单号及offset来查询，微信支付会返回offset及后面的10笔，以此类推。当商户传入的offset超过`total_refund_count`，则系统会返回报错`PARAM_ERROR`。

举例：

```text
一笔订单下的退款单有36笔，当商户想查询第25笔时，可传入订单号及offset=24，微信支付平台会返回第25笔到第35笔的退款单信息，或商户可直接传入退款单号查询退款
```

以下为调用方式：

```text
private void doRefundQuery() {
    // 四选一，微信订单号查询的优先级是： refund_id > out_refund_no > transaction_id > out_trade_no
    HashMap<String, String> data = new HashMap<String, String>();
    // 商户订单号
    data.put("out_trade_no", out_trade_no);
    // 微信订单号
    data.put("transaction_id", out_trade_no);
    // 商户退款单号    
    data.put("out_refund_no", out_trade_no);
    // 微信退款单号    
    data.put("refund_id", out_trade_no);
    try {
        Map<String, String> r = wxpay.refundQuery(data);
        System.out.println(r);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

PS：微信订单号查询的优先级是： refund\_id &gt; out\_refund\_no &gt; transaction\_id &gt; out\_trade\_no

需要注意的是，查询退款时，需要注意退款返回的错误码，如果出现错误，需要及时同步商户系统中的退款数据。

#### 结语

以上为`申请退款、退款回调接口、查询退款`相关的解释和源码，特别需要注意的是接收退款时的解密方式和替换安全文件，小伙伴们一定要注意哦，具体的源码可以看作者的github，里面对每个方法有详细的注释。

预告：下一篇文章 `下载对账单和资金账单`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： ​ ​`​https://github.com/YClimb/wxpay-sdk/blob/master/README.md ​`

加作者私人微信，作者微信号如下 `yclimb`，标明 `微信支付` 可拉入微信支付讨论群与小伙伴一起探讨哦，一定要标明 `微信支付` 哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x5FAE;&#x4FE1;&#x516C;&#x4F17;&#x53F7;](../.gitbook/assets/er-wei-ma.jpg)

