---
description: 本文是【浅析微信支付】系列文章的第十三篇，主要讲解在如何开通商户平台的红包功能和为用户发放红包，以及查询发送红包记录。
---

# 商户平台开通现金红包、指定用户发放、红包记录查询

浅析微信支付系列已经更新十三篇了哟～，没有看过的朋友们可以看一下哦。

[浅析微信支付：\(余额提现\)企业付款到微信用户零钱或银行卡账户](https://mp.weixin.qq.com/s/YZkrbYm5t8HJPT_S4W9zrQ)

[浅析微信支付：支付验收示例和验收指引](https://mp.weixin.qq.com/s/YESU2V5byxfM8z9YQXgnuA)

[浅析微信支付：如何使用沙箱环境测试](https://mp.weixin.qq.com/s/WmnsCnIrhN9STbvrNTQOiA)

[浅析微信支付：申请退款、退款回调接口、查询退款](https://mp.weixin.qq.com/s/IyWjWB__-VsqKO8SL0DL3Q)

上一篇文章我们说到，如果有`余额提现`、`返利福利`等需求时，就会用到商家向用户付款的操作，基于微信支付，上篇我们说了付款到用户余额和银行卡；本文来讲解如何使用现金红包的方式向用户发送`现金红包`，首先我们来了解什么是微信的现金红包。

#### 现金红包

现金红包，是微信支付商户平台提供的营销工具之一，上线以来深受广大商户与用户的喜爱。商户可以通过本平台向微信支付用户发放现金红包。用户领取红包后，资金到达用户微信支付零钱账户，和零钱包的其他资金有一样的使用出口；

注意：若用户未领取，资金将会在24小时后退回商户的微信支付账户中。

官方文档地址：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=13_1
```

**现金红包意义**

微信支付现金红包因资金的承载方式为现金，一直以来深受用户的青睐，近年来的春晚中，现金红包都扮演着重要的角色；在日常运营中也为商户的营销活动带来热烈的反响。总的来说，现金红包在包括但不仅限于以下场景中发挥着重要意义：

◆ 为企业拉取新用户、巩固老用户关系、提升用户活跃度 

◆ 结合巧妙的创意点子，辅以红包点缀，打造火爆的活动，提升企业与品牌知名度 

◆ 结合企业运营活动，以红包作为奖品，使你的抽奖、满送等营销活动更便利进行 

◆ 同时，除了营销之外，现金红包在企业日常的运营中也扮演着重要角色。如：为员工返福利、为供应商返利、会员积分/虚拟等级兑现等等。

什么意思？ 简单点讲，就是现金红包具有特殊的营销属性，拿公众号来讲，我们可以建立活动，通过活动的方式为用户发送现金红包，而这个红包触达的消息是在公众号聊天窗口页面，这样也可以引导用户关注公众号、提升活跃度等等。

**开通现金红包**

官方文档如下：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=13_3&index=2
```

在使用现金红包之前，请前往开通现金红包功能。 操作路径：【登录微信支付商户平台——&gt;产品中心——&gt;现金红包——&gt;开通】。

可以根据官方的声明来开通现金红包，这里说几个重要的点： 

1. 入住时间超过90天； 

2. 连续交易正常交易时间30天；

一定要注意：上面这两点是必要条件，很多新注册的公司很容易就着了道，入住时间不够、交易时间更不够，没搞明白，活活等了三个月时间；如果有小伙伴遇到这样的情况，可以换一个满足要求的主体公司来解决，我的github代码中也兼容不同主体的服务号使用微信支付相关功能，小伙伴可以看看源码`WXPayConstants`和`WXPay`这两个类，调用接口时扩展`WXPayConfigImpl`即可。

说明：在开通时请如实选择你的使用场景，且在红包的发放过程中如实上报你的场景，如有作假，微信支付将有权根据《微信支付商户平台使用协议》对你的商户号做出处理。

#### 开发前的准备

具体的操作步骤这里就不描述了，小伙伴们可以查看官方文档：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=13_3&index=2
```

上面文档中已经有详细的描述，我在这里简单描述一下重点注意项： 

1. 下载API证书 

2. 充值，保证商户余额有足够的钱（一定要注意基本账户和运营账户的区别，一般情况下，有运营账户的时候，都会从运营账户中扣款）操作路径：【登录商户平台——&gt;交易中心——&gt;资金管理——&gt;充值】 

3. 获取openid，指定用户发送红包必须先知道用户的标识openid，可以根据网页授权接口获得 

4. 设置红包参数，操作路径：【登录商户平台——&gt;产品中心——&gt;现金红包——&gt;产品设置】

对于第四点，可以设置和更改以下参数官方解释如下： 

1. 调用IP地址：设置之后，仅有已设置的IP地址可以调用，其余的IP调用会报错； 

2. 用户领取上限：限制同一openid同一日领取的个数； 

3. 防刷等级：防刷是指微信风控针对微信小号、僵尸号、机器号等的拦截，你可以通过更改防刷等级控制防刷的强度； 

4. 同时，你也可以申请更改红包额度。但是需要经过微信支付的审核，审核通过之后才会生效；

敲黑板！！！重点来了，以上第一点IP地址，就是我们调用现金红包发放的服务器IP地址了；第二点也要注意，每个用户可以领取的红包个数限制；

最需要注意的是，调用接口时，发放红包使用场景一定要慎重选择，查看一下每种场景对应的限制，比如在红包金额大于200或者小于1元时必传场景参数，这时就需要我们配置阀值。

#### 发放方式（接口发放）

方式一：接口发放 商户根据开发文档进行开发，一次调用可以给一个指定用户发送一个指定金额的红包，满足多元化的运营需求。

方式二：通过上传openid文件发放 收集要发送红包对象的openid，将openid编辑成txt文件，登录微信支付商户平台，使用上传文件功能发放。一份文件对应一个红包模板，便于管理。

方式三：配置营销规则“满额送”发放 商户可以在商户平台配置自助规则：用户使用微信支付发生交易满足一定条件，立送现金红包

本文主要讲通过接口发放的方式。

**接口链接**

```text
https://api.mch.weixin.qq.com/mmpaymkttransfers/sendredpack
```

**是否需要证书**

是

**调用接口**

官方文档地址：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=13_4&index=3
```

首先是发放规则： 

1. 发送频率限制------默认1800/min 

2. 发送个数上限------按照默认1800/min算 

3. 金额限制------默认红包金额为1-200元，如有需要，可前往商户平台进行设置和申请 

4. 其他其他限制吗？------单个用户可领取红包上线为10个/天，如有需要，可前往商户平台进行设置和申请 

5. 如果量上满足不了我们的需求，如何提高各个上限？------金额上限和用户当天领取次数上限可以在商户平台进行设置

注意1-红包金额大于200或者小于1元时，请求参数scene\_id必传。 

注意2-根据监管要求，新申请商户号使用现金红包需要满足两个条件：1、入驻时间超过90天 2、连续正常交易30天。 

注意3-移动应用的appid无法使用红包接口。

PS：上面是官方介绍，不难理解，划重点！！！（注意3的含义，就是只能使用公众号的openid，小程序的openid不可用。）

消息触达规则参考官方文档：

![&#x6D88;&#x606F;&#x89E6;&#x8FBE;&#x89C4;&#x5219;](https://img-blog.csdnimg.cn/20181116181518275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1lDbGltYg==,size_16,color_FFFFFF,t_70)

下面开始来干货，贴出源码吧，应用代码：

```text
/**
 * 发送现金红包
 *
 * @author yclimb
 * @date 2018/9/18
 */
private void sendRedPack() throws Exception {
    WXPay wxPay = new WXPay(AsydWXPayConfigImpl.getInstance());
    Map<String, String> resultMap = wxPay.sendRedPack(WXPayUtil.getPayNo(), "obX_c0YRpT47zKcvq-ZYpjU6GFuA", "1", "活动名称", "红包祝福语", "备注", "127.0.0.1");
    System.out.println("wxPay.sendRedPack:" + resultMap);
}
```

调用发送现金红包接口：

```text
/**
 * 作用：企业向指定微信用户的openid发放指定金额红包<br>
 * 场景：商户可以通过本平台向微信支付用户发放现金红包。用户领取红包后，资金到达用户微信支付零钱账户，和零钱包的其他资金有一样的使用出口；若用户未领取，资金将会在24小时后退回商户的微信支付账户中。
 * 接口文档地址：https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=13_4&index=3
 *
 * @param mch_billno       商户订单号
 * @param openid           用户openid
 * @param amount           企业付款金额
 * @param act_name         活动名称
 * @param wishing          红包祝福语
 * @param remark           备注
 * @param spbill_create_ip 该IP可传用户端或者服务端的IP
 * @return API返回数据
 * @throws Exception e
 */
public Map<String, String> sendRedPack(String mch_billno, String openid, String amount, String act_name, String wishing, String remark, String spbill_create_ip) throws Exception {

    /** 构造请求参数数据 **/
    Map<String, String> data = new HashMap<>();

    // 商户订单号     mch_billno    是    10000098201411111234567890    String(28) 商户订单号（每个订单号必须唯一。取值范围：0~9，a~z，A~Z）接口根据商户订单号支持重入，如出现超时可再调用。
    data.put("mch_billno", mch_billno);
    // 商户名称    send_name    是    天虹百货    String(32)    红包发送者名称
    data.put("send_name", "悦店");
    // 用户openid    re_openid    是    oxTWIuGaIt6gTKsQRLau2M0yL16E    String(32) 接受红包的用户openid openid为用户在wxappid下的唯一标识（获取openid参见微信公众平台开发者文档：网页授权获取用户基本信息）
    data.put("re_openid", openid);
    // 付款金额    total_amount    是    1000    int    付款金额，单位分
    data.put("total_amount", String.valueOf(new BigDecimal(amount).multiply(new BigDecimal(100)).setScale(2, BigDecimal.ROUND_HALF_UP).intValue()));
    // 红包发放总人数    total_num    是    1    int 红包发放总人数 total_num=1
    data.put("total_num", "1");
    // 红包祝福语    wishing    是    感谢您参加猜灯谜活动，祝您元宵节快乐！    String(128)    红包祝福语
    data.put("wishing", wishing);
    // Ip地址    client_ip    是    192.168.0.1    String(15)    调用接口的机器Ip地址
    data.put("client_ip", spbill_create_ip);
    // 活动名称    act_name    是    猜灯谜抢红包活动    String(32)    活动名称
    data.put("act_name", act_name);
    // 备注    remark    是    猜越多得越多，快来抢！    String(256)    备注信息
    data.put("remark", remark);

    /** 以下参数为非必填参数 **/
    /*
     * 场景id：scene_id    否    PRODUCT_8    String(32) 发放红包使用场景，红包金额大于200或者小于1元时必传
     * PRODUCT_1:商品促销
     * PRODUCT_2:抽奖
     * PRODUCT_3:虚拟物品兑奖
     * PRODUCT_4:企业内部福利
     * PRODUCT_5:渠道分润
     * PRODUCT_6:保险回馈
     * PRODUCT_7:彩票派奖
     * PRODUCT_8:税务刮奖
     */
    //data.put("scene_id", "PRODUCT_1");
    /*
     * 活动信息    risk_info    否    posttime%3d123123412%26clientversion%3d234134%26mobile%3d122344545%26deviceid%3dIOS    String(128)
     * posttime:用户操作的时间戳
     * mobile:业务系统账号的手机号，国家代码-手机号。不需要+号
     * deviceid :mac 地址或者设备唯一标识
     * clientversion :用户操作的客户端版本 把值为非空的信息用key=value进行拼接，再进行urlencode urlencode(posttime=xx& mobile =xx&deviceid=xx)
     */
    // 资金授权商户号    consume_mch_id    否    1222000096    String(32) 资金授权商户号 服务商替特约商户发放时使用

    /** 以下四个参数，在 this.redPackRequestData 方法中会自动赋值 **/
    // 商户号    mch_id    是    10000098    String(32)    微信支付分配的商户号
    // 随机字符串    nonce_str    是    5K8264ILTKCH16CQ2502SI8ZNMTM67VS    String(32)    随机字符串，不长于32位
    // 签名    sign    是    C380BEC2BFD727A4B6845133519F3AD6    String(32)    详见签名生成算法
    // 公众账号appid    wxappid    是    wx8888888888888888    String(32)    微信分配的公众账号ID（企业号corpid即为此appId）。在微信开放平台（open.weixin.qq.com）申请的移动应用appid无法使用该接口。

    // 微信调用接口
    Map<String, String> resultMap = this.sendRedPack(data);

    WXPayUtil.getLogger().info("wxPay.sendRedPack:" + resultMap);

    return resultMap;
}
```

以上为接口调用代码，对于接口调用的入参和出参，小伙伴看看官网文档哦，接口中的注释给大家一个参考。

官方还提供了一种发放`裂变红包`的接口，有需要的小伙伴可以了解一下，文档地址：

```text
https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=13_5&index=4
```

裂变红包：一次可以发放一组红包。首先领取的用户为种子用户，种子用户领取一组红包当中的一个，并可以通过社交分享将剩下的红包给其他用户。裂变红包充分利用了人际传播的优势。

#### 查询红包记录

用于商户对已发放的红包进行查询红包的具体信息，可支持普通红包和裂变包。

这个接口很简单，就是查询已经发送的红包记录，根据商户发放红包的商户订单号查询即可。

**接口链接**

```text
https://api.mch.weixin.qq.com/mmpaymkttransfers/gethbinfo
```

**是否需要证书**

是

**调用接口**

应用代码：

```text
/**
 * 查询现金红包
 *
 * @author yclimb
 * @date 2018/9/18
 */
private void getHbInfo() throws Exception {
    WXPay wxPay = new WXPay(AsydWXPayConfigImpl.getInstance());
    Map<String, String> resultMap = wxPay.getHbInfo("1502348237482342342");
    System.out.println("wxPay.getHbInfo:" + resultMap);
}
```

查询接口代码：

```text
/**
 * 作用：查询红包记录<br>
 * 场景：用于商户对已发放的红包进行查询红包的具体信息，可支持普通红包和裂变包。
 * 接口文档地址：https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=13_6&index=5
 *
 * @param mch_billno 商户订单号
 * @return API返回数据
 * @throws Exception e
 */
public Map<String, String> getHbInfo(String mch_billno) throws Exception {

    /** 构造请求参数数据 **/
    Map<String, String> data = new HashMap<>();

    // 商户订单号     mch_billno    是    10000098201411111234567890    String(28) 商户订单号（每个订单号必须唯一。取值范围：0~9，a~z，A~Z）接口根据商户订单号支持重入，如出现超时可再调用。
    data.put("mch_billno", mch_billno);
    // 订单类型    bill_type    是    MCHT    String(32)    MCHT:通过商户订单号获取红包信息。
    data.put("bill_type", "MCHT");

    /** 以下四个参数，在 this.fillRequestData 方法中会自动赋值 **/
    // 商户号    mch_id    是    10000098    String(32)    微信支付分配的商户号
    // 随机字符串    nonce_str    是    5K8264ILTKCH16CQ2502SI8ZNMTM67VS    String(32)    随机字符串，不长于32位
    // 签名    sign    是    C380BEC2BFD727A4B6845133519F3AD6    String(32)    详见签名生成算法
    // 公众账号appid    appid    是    wx8888888888888888    String(32)    微信分配的公众账号ID（企业号corpid即为此appId）。在微信开放平台（open.weixin.qq.com）申请的移动应用appid无法使用该接口。

    // 微信调用接口
    Map<String, String> resultMap = this.getHbInfo(data);

    WXPayUtil.getLogger().info("wxPay.getHbInfo:" + resultMap);

    return resultMap;
}
```

此接口通过商户订单号获取红包信息，很简单，对于返回参数小伙伴们可以查看官方文档，注意一下错误代码即可。

#### 结语

以上为`商户平台开通现金红包、指定用户发放、红包记录查询`相关的解释和源码，小伙伴们一定要注意看看官方文档哦，具体的源码可以看作者的github，里面对每个方法有详细的注释。

如果小伙伴有遇到解决不了的问题，可以关注作者微信公众号，加入讨论群中发出疑问，和小伙伴们一起解决哦～

预告：下一篇文章会讲发放奖励的另一种方式 `商户平台代金券或立减优惠开通、指定用户发放、查询等`，敬请期待！！！

​如果想要提前一览源码的小伙伴，可以先看看我的 github，地址如下： ​ ​`​https://github.com/YClimb/wxpay-sdk/blob/master/README.md ​`

关注作者微信公众号，点击下方`讨论群`，扫码即可加入`微信支付讨论群`与小伙伴一起探讨哦～

到此本文就结束了，关注公众号查看更多推送！！！

![&#x5173;&#x6CE8;&#x6211;&#x7684;&#x516C;&#x4F17;&#x53F7;](https://img-blog.csdn.net/20180130111432962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvWUNsaW1i/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

