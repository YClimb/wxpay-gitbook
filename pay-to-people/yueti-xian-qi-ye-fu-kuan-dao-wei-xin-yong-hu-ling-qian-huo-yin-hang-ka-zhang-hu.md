---
description: 本文是【浅析微信支付】系列文章的第十二篇，主要讲解在商户存在的提现、商户付款到微信用户零钱或者银行卡需求。
---

# \(余额提现\)企业付款到微信用户零钱或银行卡账户

浅析微信支付系列已经更新十二篇了哟～，没有看过的朋友们可以看一下哦。

[浅析微信支付：支付验收示例和验收指引](https://mp.weixin.qq.com/s/YESU2V5byxfM8z9YQXgnuA)

[浅析微信支付：如何使用沙箱环境测试](https://mp.weixin.qq.com/s/WmnsCnIrhN9STbvrNTQOiA)

[浅析微信支付：下载对账单和资金账单](https://mp.weixin.qq.com/s/XCR1Ts-uabuC573_vLb3Qg)

[浅析微信支付：申请退款、退款回调接口、查询退款](https://mp.weixin.qq.com/s/IyWjWB__-VsqKO8SL0DL3Q)

如果你是做电商或者某些有福利返利的系统，基本上会遇到诸如 `余额提现` 这类需求，主要就是平台向用户返利现金，积累到某一个门槛，可以领取到自己的余额账号、银行卡；或者是使用为用户发送现金红包的方式。

接下来的两篇文章，会为大家描述在微信支付中，像用户付款的以上三种方式。

以下为三种付款方式的必要条件： 

1. 商户号（或同主体其他非服务商商户号）已入驻90日 

2. 商户号（或同主体其他非服务商商户号）有30天连续正常交易 

3. 登录微信支付商户平台-产品中心，开通企业付款。

#### 企业付款到微信用户零钱

企业付款提供由商户直接付钱至用户微信零钱的能力，支持平台操作及接口调用两种方式，资金到账速度快，使用及查询方便。主要用来解决合理的商户对用户付款需求，比如：保险理赔、彩票兑换等等。

如何开通？ 

1. 入驻成为商户：在线提交营业执照、身份证、银行账户等基本信息，快速提交申请；

2. 超级管理员开通：前往商户平台-产品中心-企业付款到零钱-申请开通； 

3. **特殊要求：交易资金是即时入账到商户号基本户的商户，需要满足以下要求：需入驻满90天，连续交易30天。**

所需资料：开通企业付款到零钱功能无需提供额外的材料。 费用：试用期间免费使用。

**应用场景**

企业付款为企业提供付款至用户零钱的能力，支持通过API接口付款，或通过微信支付商户平台（pay.weixin.qq.com）网页操作付款。

以下为官方的解释：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=14_1
```

抓重点，首先需要知道的是，开通了`运营账户`的商户，付款时会从运营账户余额中扣除，这个一定要注意，以免金额不足时付款失败（可以使用主账户为运营账户充值，参考\[交易中心\]-\[充值/转入\]）。

以下为特别需要注意的地方，为大家标记出来，设计系统时一定要参考一下，以免入坑。

![&#x4F01;&#x4E1A;&#x4ED8;&#x6B3E;&#x5230;&#x4F59;&#x989D;-1](https://img-blog.csdnimg.cn/20181115194658528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lDbGltYg==,size_16,color_FFFFFF,t_70)

**接口链接**

```text
https://api.mch.weixin.qq.com/mmpaymkttransfers/promotion/transfers
```

**是否需要证书**

请求需要双向证书。

**调用接口**

注意事项： 

◆ 当返回错误码为“SYSTEMERROR”时，请不要更换商户订单号，一定要使用原商户订单号重试，否则可能造成重复支付等资金风险。 

◆ XML具有可扩展性，因此返回参数可能会有新增，而且顺序可能不完全遵循此文档规范，如果在解析回包的时候发生错误，请商户务必不要换单重试，请商户联系客服确认付款情况。如果有新回包字段，会更新到此API文档中。 

◆ 因为错误代码字段err\_code的值后续可能会增加，所以商户如果遇到回包返回新的错误码，请商户务必不要换单重试，请商户联系客服确认付款情况。如果有新的错误码，会更新到此API文档中。 

◆ 错误代码描述字段err\_code\_des只供人工定位问题时做参考，系统实现时请不要依赖这个字段来做自动化处理。

PS：目前支持向指定微信用户的openid付款。

官方文档如下：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=14_2
```

具体的传入参数，这里就不一一列举了，请大家参考一下官方文档，下面贴上具体的实现源码：

```text
/**
 * [微信支付提现接口] - 保存调用的相关记录
 * @param payment 支付对象
 * @param wxPayConfig 微信支付单例对象
 * @return map
 *
 * @author yclimb
 * @date 2018/7/30
 */
public Map<String, String> saveWxPayTransfers(Payment payment, WXPayConfig wxPayConfig) throws Exception {
    // 支付前验证

    // 微信支付对象
    // WXPay wxPay = new WXPay(WXPayConfigImpl.getInstance());
    WXPay wxPay = new WXPay(wxPayConfig);

    // 微信退款接口
    Map<String, String> resultMap = wxPay.transfers(...);
    logger.info("saveWxPayTransfers:resultMap:" + resultMap.toString());

    // 下单失败，进行处理
    if (WXPayConstants.FAIL.equals(resultMap.get(WXPayConstants.RETURN_CODE)) || WXPayConstants.FAIL.equals(resultMap.get(WXPayConstants.RESULT_CODE))) {

        // 处理结果返回，无需继续执行

        // 余额不足提醒
        if (WXPayCodeEnum.ERR_CODE_NOTENOUGH.getCode().equals(resultMap.get(WXPayConstants.ERR_CODE))) {
            // 发送余额不足的消息提醒

        }
    }

    // 付款记录修改 & 记录付款日志

    return resultMap;
}
```

以上为调用的应用方法，下面为大家贴出微信接口调用代码 `imall.weixin.sdk.WXPay`：

```text
/**
 * 作用：企业向微信用户个人付款<br>
 * 场景：企业付款为企业提供付款至用户零钱的能力，支持通过API接口付款，或通过微信支付商户平台（pay.weixin.qq.com）网页操作付款。
 * 接口文档地址：https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=14_2
 *
 * @param partner_trade_no 商户订单号
 * @param openid           用户openid
 * @param amount           企业付款金额
 * @param desc             企业付款描述信息
 * @param spbill_create_ip 该IP可传用户端或者服务端的IP
 * @return API返回数据
 * @throws Exception e
 */
public Map<String, String> transfers(String partner_trade_no, String openid, String amount, String desc, String spbill_create_ip) throws Exception {

    /** 构造请求参数数据 **/
    Map<String, String> data = new HashMap<>();

    // 商户订单号    partner_trade_no    是    10000098201411111234567890    String    商户订单号，需保持唯一性(只能是字母或者数字，不能包含有符号)
    data.put("partner_trade_no", partner_trade_no);
    // 用户openid    openid    是    oxTWIuGaIt6gTKsQRLau2M0yL16E    String    商户appid下，某用户的openid
    data.put("openid", openid);
    // 校验用户姓名选项    check_name    是    FORCE_CHECK    String    NO_CHECK：不校验真实姓名,FORCE_CHECK：强校验真实姓名
    data.put("check_name", "NO_CHECK");
    // 金额    amount    是    10099    int    企业付款金额，单位为分
    data.put("amount", String.valueOf(new BigDecimal(amount).multiply(new BigDecimal(100)).setScale(2, BigDecimal.ROUND_HALF_UP).intValue()));
    // 企业付款描述信息    desc    是    理赔    String    企业付款操作说明信息。必填。
    data.put("desc", desc);
    // Ip地址    spbill_create_ip    是    192.168.0.1    String(32)    该IP同在商户平台设置的IP白名单中的IP没有关联，该IP可传用户端或者服务端的IP。
    data.put("spbill_create_ip", spbill_create_ip);

    /** 以下参数为非必填参数 **/

    /*// 设备号    device_info    否    013467007045764    String(32)    微信支付分配的终端设备号
    data.put("device_info", "xxx");
    // 收款用户姓名    re_user_name    可选    王小王    String    收款用户真实姓名。(如果check_name设置为FORCE_CHECK，则必填用户真实姓名)
    data.put("re_user_name", "xxx");*/

    // 微信调用接口
    Map<String, String> resultMap = this.transfers(data);

    WXPayUtil.getLogger().info("wxPay.transfers:" + resultMap);

    return resultMap;
}
```

PS：推荐数据库中对于金额存储为数值单位，以分为单位来存储，1.1元可以储存为101，这样和微信对应，会方便很多。

对于企业付款查询的接口，这里就不详细描述了，以下为具体的官方文档链接：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=14_3
```

需要的朋友，根据文档进行接口查询即可，非高频接口。

#### 企业付款到银行卡

企业付款到银行卡提供由商户直接付钱至指定银行卡账户的能力，支持平台操作及接口调用两种方式，资金到账速度快，使用及查询方便。主要用来解决合理的商户对用户付款需求，比如：保险理赔、彩票兑换等等。

开通流程： 

1. 入驻成为商户：在线提交营业执照、身份证、银行账户等基本信息，快速提交申请； 

2. 超级管理员开通：前往商户平台-产品中心-企业付款到银行卡-申请开通； 

3. 特殊要求：交易资金是即时入账到商户号基本户的商户，需要满足以下要求：需入驻满90天，连续交易30天。

**所需资料：开通企业付款到银行卡功能无需提供额外的材料。 费用：此功能需收取手续费，按照单笔金额收取，每笔收取0.1%,最低1元，最高25元。**

**应用场景**

微信支付已上线企业付款至银行卡功能。商户可以将商户号余额付款至指定的收款银行账户。通过指定收款银行账户户名、卡号，以及收款银行信息即可实现付款。

官方文档地址：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=24_1&index=1
```

功能说明： 

1. 企业付款至银行卡只支持新资金流类型账户 

2. 目前企业付款到银行卡支持17家银行，更多银行逐步开放中 

3. 付款到账实效为1-3日，最快次日到账 

4. 每笔按付款金额收取手续费，按金额0.1%收取，最低1元，最高25元,如果商户开通了运营账户，手续费和付款的金额都从运营账户出。如果没有开通，则都从基本户出。 

5. 每个商户号每天可以出款100万，单商户给同一银行卡付款每天限额5万 

6. 发票：在账户中心-发票信息页面申请开票的商户会按月收到发票（已申请的无需重复申请）。 企业付款到银行卡发票与交易手续费发票为拆分单独开具。

需要注意的是，微信支持的银行有限，具体的支持银行见如下链接：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=24_4&index=5
```

所以肯定会出现不支持的银行，小伙伴们在开发的时候，可以在前后端控制用户选择提现银行来解决。

平台上手动付款流程： 

1. 在产品中心，开通企业付款到个人银行卡功能 

2. 进入交易中心-企业付款到银行卡页面进行付款 

3. 指定收款银行账号、户名、收款方开户行，及付款金额信息，即可实现付款

**接口链接**

```text
https://api.mch.weixin.qq.com/mmpaysptrans/pay_bank
```

**是否需要证书**

请求需要双向证书。

**调用接口**

接口介绍： 用于企业向微信用户银行卡付款 目前支持接口API的方式向指定微信用户的银行卡付款。

接口调用规则： 

◆ 单商户日限额——单日100w 

◆ 单次限额——单次5w 

◆ 单商户给同一银行卡单日限额——单日5w

注意：重点来了，首先，收款方银行卡号`enc_bank_no`、收款方用户名`enc_true_name` 这两个入参是需要 `采用标准RSA算法，公钥由微信侧提供` 得到的，所以还需要先拿到这个密钥，下面是官方文档地址：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=24_7&index=4
```

以上文档详细介绍了如何得到具体的密钥方式，如果有看不明白的小伙伴，可以直接百度 `获取RSA加密公钥API`，可以得到很多示例，这里我就不讲了。

除入参和`企业付款到微信用户零钱`有所不一致之外，其他方面都差不多，小伙伴们可以参考上面付款到零钱的接口来实现付款到银行卡接口。

#### 结语

以上为`微信余额提现`相关的解释和源码，小伙伴们一定要注意看看官方文档哦，具体的源码可以看作者的github，里面对每个方法有详细的注释。

如果小伙伴有遇到解决不了的问题，可以关注作者微信公众号，加入讨论群中发出疑问，和小伙伴们一起解决哦～

预告：下一篇文章会讲发放奖励的另一种方式 `商户平台-现金红包`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： ​ ​`​https://github.com/YClimb/wxpay-sdk/blob/master/README.md ​`

关注作者微信公众号，点击下方`讨论群`，扫码即可加入`微信支付讨论群`与小伙伴一起探讨哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x516C;&#x4F17;&#x53F7;](https://img-blog.csdn.net/20180130111432962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvWUNsaW1i/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

