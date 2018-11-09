---
description: 本文是【浅析微信支付】系列文章的第六篇，主要讲解支付成功后，微信回调商户支付结果通知的处理。
---

# 支付结果通知

浅析微信支付系列已经更新五篇了哟～，没有看过的朋友们可以看一下哦。

[浅析微信支付：统一下单接口](https://mp.weixin.qq.com/s/lYPiad1pyZ6ZdqH-sg66eg)

[浅析微信支付：微信公众号网页授权](https://mp.weixin.qq.com/s/BQPn_r5pcwZO3tzqnoTiKg)

[浅析微信支付：开发前的准备](https://mp.weixin.qq.com/s/mkxukWU4aMw92NxnL91dng)

前面一章已经讲了如何调用统一下单接口和调起微信支付窗口，在调用下单接口时，我们会传入 `异步接收微信支付结果通知的回调地址`，顾名思义这个地址作用就是用来接收支付结果通知，当用户在前端支付成功后，微信服务器会自动调用此地址，然后商户再进行处理。

#### 1、支付结果通知

以下为接口官方解释：

```text
支付完成后，微信会把相关支付结果和用户信息发送给商户，商户需要接收处理，并返回应答。

对后台通知交互时，如果微信收到商户的应答不是成功或超时，微信认为通知失败，微信会通过一定的策略定期重新发起通知，尽可能提高通知的成功率，但微信不保证通知最终能成功。 （通知频率为15/15/30/180/1800/1800/1800/1800/3600，单位：秒）

注意：同样的通知可能会多次发送给商户系统。商户系统必须能够正确处理重复的通知。
推荐的做法是，当收到通知进行处理时，首先检查对应业务数据的状态，判断该通知是否已经处理过，如果没有处理过再进行处理，如果处理过直接返回结果成功。在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱。

特别提醒：商户系统对于支付结果通知的内容一定要做签名验证,并校验返回的订单金额是否与商户侧的订单金额一致，防止数据泄漏导致出现“假通知”，造成资金损失。
技术人员可登进微信商户后台扫描加入接口报警群。
```

支付结果通知接口文档地址：

```text
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_7&index=8
```

需要注意的事项有以下几点：

```text
1. 该链接是通过【统一下单API】中提交的参数notify_url设置，如果链接无法访问，商户将无法接收到微信通知。
2. 通知url必须为直接可访问的url，不能携带参数，也就是必须使用外网接口地址，不能使用本地调试地址
3. 商户需要接收处理，并返回应答。如果微信收到商户的应答不是成功或超时，微信认为通知失败，微信会通过一定的策略定期重新发起通知，尽可能提高通知的成功率，但微信不保证通知最终能成功。
4. 通知频率为15/15/30/180/1800/1800/1800/1800/3600，单位：秒
5. 同样的通知可能会多次发送给商户系统。商户系统必须能够正确处理重复的通知。
6. 特别提醒：商户系统对于支付结果通知的内容一定要做签名验证,并校验返回的订单金额是否与商户侧的订单金额一致，防止数据泄漏导致出现“假通知”，造成资金损失。
```

PS：推荐的做法是，当收到通知进行处理时，首先检查对应业务数据的状态，判断该通知是否已经处理过，如果没有处理过再进行处理，如果处理过直接返回结果成功。在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱。

关于具体的签名和接收通知代码如下：

```text
package imall.controller.wx;
package ...

/**
 * 微信支付Controller
 *
 * @author yclimb
 * @date 2018/6/15
 */
@Api
@RestController
@RequestMapping("/weixin/pay")
public class WXPayController extends BaseController {

    // 需要注入的一些service

    /**
     * 返回成功xml
     */
    private String resSuccessXml = "<xml><return_code><![CDATA[SUCCESS]]></return_code><return_msg><![CDATA[OK]]></return_msg></xml>";

    /**
     * 返回失败xml
     */
    private String resFailXml = "<xml><return_code><![CDATA[FAIL]]></return_code><return_msg><![CDATA[报文为空]]></return_msg></xml>";

    /**
     * 该链接是通过【统一下单API】中提交的参数notify_url设置，如果链接无法访问，商户将无法接收到微信通知。
     * 通知url必须为直接可访问的url，不能携带参数。示例：notify_url：“https://pay.weixin.qq.com/wxpay/pay.action”
     * <p>
     * 支付完成后，微信会把相关支付结果和用户信息发送给商户，商户需要接收处理，并返回应答。
     * 对后台通知交互时，如果微信收到商户的应答不是成功或超时，微信认为通知失败，微信会通过一定的策略定期重新发起通知，尽可能提高通知的成功率，但微信不保证通知最终能成功。
     * （通知频率为15/15/30/180/1800/1800/1800/1800/3600，单位：秒）
     * 注意：同样的通知可能会多次发送给商户系统。商户系统必须能够正确处理重复的通知。
     * 推荐的做法是，当收到通知进行处理时，首先检查对应业务数据的状态，判断该通知是否已经处理过，如果没有处理过再进行处理，如果处理过直接返回结果成功。在对业务数据进行状态检查和处理之前，要采用数据锁进行并发控制，以避免函数重入造成的数据混乱。
     * 特别提醒：商户系统对于支付结果通知的内容一定要做签名验证，防止数据泄漏导致出现“假通知”，造成资金损失。
     *
     * @author yclimb
     * @date 2018/6/15
     */
    @ApiOperation(value = "微信支付|支付回调接口", httpMethod = "POST", notes = "该链接是通过【统一下单API】中提交的参数notify_url设置，如果链接无法访问，商户将无法接收到微信通知。")
    @RequestMapping("/wxnotify")
    public void wxnotify(HttpServletRequest request, HttpServletResponse response) {

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

            WXPayUtil.getLogger().info("wxnotify:微信支付----start----");

            // 获取微信调用我们notify_url的返回信息
            String result = new String(outSteam.toByteArray(), "utf-8");
            WXPayUtil.getLogger().info("wxnotify:微信支付----result----=" + result);

            // 关闭流
            outSteam.close();
            inStream.close();

            // xml转换为map
            Map<String, String> resultMap = WXPayUtil.xmlToMap(result);
            boolean isSuccess = false;
            if (WXPayConstants.SUCCESS.equalsIgnoreCase(resultMap.get(WXPayConstants.RESULT_CODE))) {

                WXPayUtil.getLogger().info("wxnotify:微信支付----返回成功");

                if (WXPayUtil.isSignatureValid(resultMap, WXPayConstants.API_KEY)) {

                    // 订单处理 操作 orderconroller 的回写操作?
                    WXPayUtil.getLogger().info("wxnotify:微信支付----验证签名成功");

                    // 通知微信.异步确认成功.必写.不然会一直通知后台.八次之后就认为交易失败了.
                    resXml = resSuccessXml;
                    isSuccess = true;

                } else {
                    WXPayUtil.getLogger().error("wxnotify:微信支付----判断签名错误");
                }

            } else {
                WXPayUtil.getLogger().error("wxnotify:支付失败,错误信息：" + resultMap.get(WXPayConstants.ERR_CODE_DES));
                resXml = resFailXml;
            }

            // 付款记录修改 & 记录付款日志

            // 回调方法，处理业务 - 修改订单状态
            WXPayUtil.getLogger().info("wxnotify:微信支付回调：修改的订单===>" + resultMap.get("out_trade_no"));
            int updateResult = ...;
            if (updateResult > 0) {
                WXPayUtil.getLogger().info("wxnotify:微信支付回调：修改订单支付状态成功");
            } else {
                WXPayUtil.getLogger().error("wxnotify:微信支付回调：修改订单支付状态失败");
            }

        } catch (Exception e) {
            WXPayUtil.getLogger().error("wxnotify:支付回调发布异常：", e);
        } finally {
            try {
                // 处理业务完毕
                BufferedOutputStream out = new BufferedOutputStream(response.getOutputStream());
                out.write(resXml.getBytes());
                out.flush();
                out.close();
            } catch (IOException e) {
                WXPayUtil.getLogger().error("wxnotify:支付回调发布异常:out：", e);
            }
        }

    }

}
```

验证是否本商户返回的正确通知：

```text
// xml转换为map
Map<String, String> resultMap = WXPayUtil.xmlToMap(result);

// 判断签名是否正确，必须包含sign字段，否则返回false。使用MD5签名。
WXPayUtil.isSignatureValid(resultMap, WXPayConstants.API_KEY);
```

从以上代码可以得知，微信是以流的方式来调用支付结果通知，流中数据格式为xml，所以需要先对流进行解析，然后再将xml转换为map，上面方法已经实现这个过程，有现成的代码可供参考，无需再重新编写代码。

需要注意的是，不论成功或者失败，必须向微信返回对应的返回值，如上方代码中的`resSuccessXml`、`resFailXml`，否则会引起重复调用的问题，在代码中我们应该规避风险。

#### 2、对于支付单的处理

在收到支付结果后，我们会对系统中的支付单进行状态修改等操作，此时需要注意，如果微信返回失败，接口直接返回失败即可；

如果微信通知付款成功，返回时有一个 `out_trade_no` 参数，此参数为调用 `统一下单接口` 时传入微信的支付单号，可根据此参数取得对应的支付单，然后进行修改操作，若操作时出现异常，一定要向微信返回错误的xml代码，用微信的重试机制来二次回调，否则就需要记录通知信息，在本系统自动重试来解决。

在处理微信验证数据时，还需要注意微信最终返回的结果中是否使用了代金券，如果使用了代金券，还需要另行处理，此处先不详细描述，后面章节会详细解释代金券的操作。

#### 结语

以上为 `微信支付结果通知接口` 的接收方式，它是一个微信服务自身控制的异步调用方法，在自身商户系统中需要处理很多异常，如网络抖动和服务异常等问题，所以尽量在商户系统中保存微信结果通知数据和增加失败重试机制，保证数据的正确性。

预告：下一篇文章 `查询订单和关闭订单`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： `https://github.com/YClimb/wxpay-sdk/blob/master/README.md`

加作者私人微信，作者微信号如下 `yclimb`，标明 `微信支付` 可拉入微信支付讨论群与小伙伴一起探讨哦，一定要标明 `微信支付` 哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x5FAE;&#x4FE1;&#x516C;&#x4F17;&#x53F7;](../.gitbook/assets/er-wei-ma.jpg)

