---
description: 本文是【浅析微信支付】系列文章的第十一篇，主要讲解支付验收示例和验收指引。
---

# 支付验收示例和验收指引

浅析微信支付系列已经更新十一篇了哟～，没有看过的朋友们可以看一下。

[浅析微信支付：如何使用沙箱环境测试](https://mp.weixin.qq.com/s/WmnsCnIrhN9STbvrNTQOiA)

[浅析微信支付：下载对账单和资金账单](https://mp.weixin.qq.com/s/XCR1Ts-uabuC573_vLb3Qg)

[浅析微信支付：申请退款、退款回调接口、查询退款](https://mp.weixin.qq.com/s/IyWjWB__-VsqKO8SL0DL3Q)

[浅析微信支付：查询订单和关闭订单](https://mp.weixin.qq.com/s/SG4sTHsUKKJF-_Qgpjh0jA)

上一篇文章我们讲了 `如何使用沙箱环境测试`，文中有讲到沙箱环境不仅可以用来当开发环境使用，及时返回接口数据，还能当作微信支付的 `验收示例`，官方指出，为了安全考虑希望所有商户都接入验收，以下我们会结合官方文档为大家讲解如何接入及相关的验收用例。

#### 验收指引

官方文档地址：

```text
https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=23_1
```

本文阅读对象为：商户自有系统（包括但不限于：在线购物平台、人工收银系统、自动化智能收银系统、APP应用等）负责微信支付功能验收的测试及开发人员。

为保证商户接入质量，提升交易安全及用户体验，微信支付的合作服务商在正式上线交易前，必须先根据本文指引完成验收。验收完成后，服务商在验收公众平台（`微信号：WXPayAssist`）提交验收通过申请，审核通过后，才能开通相应的支付权限（如：刷卡支付）。否则，请根据审核驳回提示，重新完成验收。

注：仿真测试环境中的商户号（父子商户号）需使用真实商户号。

**验收流程**

![&#x56FE;2 &#x5546;&#x6237;&#x63A5;&#x5165;&#x9A8C;&#x6536;&#x6D41;&#x7A0B;](https://img-blog.csdnimg.cn/20181113200711866.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lDbGltYg==,size_16,color_FFFFFF,t_70)

如图2，商户在收到微信支付审核通过的邮件后，即可用邮件中提供的开发者信息，启动测试验收工作。验收开始后，验收负责人可按照下表步骤操作：

![&#x9A8C;&#x6536;&#x6B65;&#x9AA4;](https://img-blog.csdnimg.cn/20181113200728186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lDbGltYg==,size_16,color_FFFFFF,t_70)

以上为验收的基本步骤，首先，我们需要接入 `沙箱环境`，不知道的小伙伴可以查看我的上一篇文章，有详细描述，这里就不细说了。

**验收测试用例**

如果已经接入沙箱环境，我们就可以开始选择微信官方对应的验收用例进行测试了，官方提供了四种验收用例，如下：

请根据您需要开通的功能来选择相应的验收用例进行测试：

◆ [刷卡支付验收用例](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=23_11) 

◆ [扫码支付验收用例](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=23_12) 

◆ [公众号支付验收用例](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=23_13) 

◆ [免充值券验收用例](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=23_15)

这里我们以 `公众号支付验收用例` 来做例子，下面为官方的验收流程： ![&#x516C;&#x4F17;&#x53F7;&#x652F;&#x4ED8;&#x9A8C;&#x6536;&#x7528;&#x4F8B;](https://img-blog.csdnimg.cn/20181113200745632.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lDbGltYg==,size_16,color_FFFFFF,t_70)

流程我们已经知道了，重点来了，我们需要下载验收用例，下面是地址：

```text
https://pay.weixin.qq.com/wiki/doc/api/download/jsapi_yanshou.zip
```

首先，请关注上面图片中的二维码，如果遇到问题，可以查看官方的异常解答；下载验收用例后，我们会得到 4 个用例文档，需要根据文档中的描述来进行验收，`支付成功`、`支付失败`接口是必须验收的。

**如何验收？**

简单讲，验收分为以下几个步骤： 1. 获取`sandbox_signkey` 2. 修改正常接口地址为沙箱环境地址，增加 `sandboxnew` 路径 3. 根据用例集标题中的金额传入参数，调用相应的接口 4. 查看返回值与用例集中是否一致，如果一致则成功，否则失败

需要注意的是，一定要根据用例集中的标题传入金额，比如`支付成功用例集`需要传入金额`1.01`元，那我们就必须传入这个金额，传入其他金额会导致失败。

以下为示例代码：

```text
public static void main(String[] args) throws Exception {
    System.out.println("--------------->");

    // 沙箱环境测试
    WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance(), true, true);

    Map<String, String> resultMap = wxPay.unifiedOrder(notify_url, openid, body, out_trade_no, 
    "1.01", spbill_create_ip, goods_tag, detail,
            timeStart, timeExpire);


    System.out.println(resultMap);

    /*Map<String, String> resultMap = wxPay.refund(null, "10000", "10001", "1.01", "0.01", "测试微信退款");
    System.out.println(WXPayUtil.isSignatureValid(resultMap, WXPayConstants.API_KEY));*/


    System.out.println("<---------------");
}
```

上面代码中是作者封装好的sdk方法，开启沙箱环境只需要实例化对象时传入参数即可：

```text
// 沙箱环境测试
WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance(), true, true);

// 正式环境
WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance());
```

具体源码见下面文末github地址。

#### 结语

给小伙伴们分享点验收的经验，首先，一定要先看一遍官方文档，然后跟着官方文档一步步的操作，对于官方所讲的关键信息，必须仔细检查，比如上面所说的金额，还有官方标红的一些注释，本文主要目的是给大家一个分享和参考，比较方便的是作者已经封装好的sdk中有相关的 `沙箱环境` 切换示例，不需要大家再分析具体实现，关注如何应用即可。

如果小伙伴有遇到解决不了的问题，可以关注作者微信公众号，加入讨论群中发出疑问，和小伙伴们一起解决哦～

预告：下一篇文章 `(余额提现)企业付款到微信用户零钱账户`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： ​ ​`​https://github.com/YClimb/wxpay-sdk/blob/master/README.md ​`

关注作者微信公众号，点击下方`讨论群`，扫码即可加入`微信支付讨论群`与小伙伴一起探讨哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x516C;&#x4F17;&#x53F7;](https://img-blog.csdn.net/20180130111432962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvWUNsaW1i/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

