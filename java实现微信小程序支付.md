### 微信小程序支付

#### 小程序请求路径

```
下单接口路径
https://api.mch.weixin.qq.com/v3/pay/transactions/jsapi
```

##### 请求头：

| key              | value                   |
| ---------------- | ----------------------- |
| Accept           | application/json        |
| Content-Type     | application/json        |
| Wechatpay-Serial | 商户API证书的证书序列号 |



##### 请求参数：

```json
{
    "appid" : "小程序appId",
    "mchid" : "直连商户号",
    "description" : " 商品描述",
    "out_trade_no" : "商户系统内部订单号，只能是数字、大小写字母_-*且在同一个商户号下唯一。",
    "notify_url" : "服务端接收小程序请求的地址",
    "amount" : {
      "total" : 100,// 金额 单位为：分
      "currency" : "CNY"// 币种
    },
    "payer" : {
      "openid" : "用户oppenId"
    }
    
  }
```

##### 响应

```json
{
    // 预支付交易会话标识。用于后续接口调用中使用，该值有效期为2小时
  "prepay_id" : "wx201410272009395522657a690389285100"
}
```



#### 开始前配置

##### xml文件配置

```xml
  wechat:
    # 小程序的appId
    appid: wx29dd5446d8d78409
    # 小程序的密钥
    secret: d9cea18424e7438e95006aea53e5bbd1
    # 商户号
    mchid: 1561414331
    # 商户API证书的证书序列号
    mch-serial-no: 4B3B3DC35414AD50B1B755BAF8DE9CC7CF407606
    # 商户私钥文件
    private-key-file-path: C:\software\apiclient_key.pem
    # 证书解密的密钥
    api-v3-key: CZBK51236435wxpay435434323FFDuv3
    # 平台证书
    we-chat-pay-cert-file-path: C:\software\wechatpay_166D96F876F45C7D07CE98952A96EC980368ACFC.pem
    # 支付成功的回调地址
    notify-url:  https://5d52663b.r11.cpolar.top/notify/paySuccess
    # 退款成功的回调地址
    refund-notify-url: https://5d52663b.r11.cpolar.top/notify/refundSuccess
```

##### 创建WechatProperties.class 配置类文件

```java
package com.sky.properties;

import lombok.Data;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "项目名.wechat")
@Data
public class WeChatProperties {

    private String appid; //小程序的appid
    private String secret; //小程序的秘钥
    private String mchid; //商户号
    private String mchSerialNo; //商户API证书的证书序列号
    private String privateKeyFilePath; //商户私钥文件
    private String apiV3Key; //证书解密的密钥
    private String weChatPayCertFilePath; //平台证书
    private String notifyUrl; //支付成功的回调地址
    private String refundNotifyUrl; //退款成功的回调地址

}

```



##### 代码实现

向微信支付的接口发送POST请求获取prepay_id（预支付交易会话标识），prepay_id后面用来处理小程序端用户向微信发送支付请求的参数

```java
/**
     * jsapi下单
     *
     * @param orderNum    商户订单号
     * @param total       总金额
     * @param description 商品描述
     * @param openid      微信用户的openid
     * @return
     */
    private String jsapi(String orderNum, BigDecimal total, String description, String openid) throws Exception {
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("appid", weChatProperties.getAppid());
        jsonObject.put("mchid", weChatProperties.getMchid());
        jsonObject.put("description", description);
        jsonObject.put("out_trade_no", orderNum);
        jsonObject.put("notify_url", weChatProperties.getNotifyUrl());

        JSONObject amount = new JSONObject();
        amount.put("total", total.multiply(new BigDecimal(100)).setScale(2, BigDecimal.ROUND_HALF_UP).intValue());
        amount.put("currency", "CNY");

        jsonObject.put("amount", amount);

        JSONObject payer = new JSONObject();
        payer.put("openid", openid);

        jsonObject.put("payer", payer);

        String body = jsonObject.toJSONString();
        return post(JSAPI, body);
    }

 /**
     * 发送post方式请求
     *
     * @param url
     * @param body
     * @return
     */
    private String post(String url, String body) throws Exception {
        CloseableHttpClient httpClient = getClient();

        HttpPost httpPost = new HttpPost(url);
        httpPost.addHeader(HttpHeaders.ACCEPT, ContentType.APPLICATION_JSON.toString());
        httpPost.addHeader(HttpHeaders.CONTENT_TYPE, ContentType.APPLICATION_JSON.toString());
        httpPost.addHeader("Wechatpay-Serial", weChatProperties.getMchSerialNo());
        httpPost.setEntity(new StringEntity(body, "UTF-8"));

        CloseableHttpResponse response = httpClient.execute(httpPost);
        try {
            String bodyAsString = EntityUtils.toString(response.getEntity());
            return bodyAsString;
        } finally {
            httpClient.close();
            response.close();
        }
    }
```

##### 获取到prepay_id，使用它处理我们小程序需要发送请求的参数

###### 用户向微信发送请求需要的参数

```javascript
wx.requestPayment
(
  {
    "timeStamp": "时间戳，标准北京时间，时区为东八区，自1970年1月1日 0点0分0秒以来的秒数。注意：部分系统取到的值为毫秒级，需要转换成秒(10位数字)。",
    "nonceStr": "随机字符串，不长于32位",
    "package": "小程序下单接口返回的prepay_id参数值，提交格式如：prepay_id=***",
      // 签名类型
    "signType": "RSA",
      // 签名，使用字段appid、timeStamp、nonceStr、package计算得出的签名值签名所使用的appid，为【小程序下单】时传入的appid，微信支付会校验下单与调起支付所使用的appid的一致性。
    "paySign": "oR9d8PuhnIc+YZ8cBHFCwfgpaK9gd7vaRvkYD7rthRAZ\/X+QBhcCYL21N7cHCTUxbQ+EAt6Uy+lwSN22f5YZvI45MLko8Pfso0jm46v5hqcVwrk6uddkGuT+Cdvu4WBqDzaDjnNa5UK3GfE1Wfl2gHxIIY5lLdUgWFts17D4WuolLLkiFZV+JSHMvH7eaLdT9N5GBovBwu5yYKUR7skR8Fu+LozcSqQixnlEZUfyE55feLOQTUYzLmR9pNtPbPsu6WVhbNHMS3Ss2+AehHvz+n64GDmXxbX++IOBvm2olHu3PsOUGRwhudhVf7UcGcunXt8cqNjKNqZLhLw4jq\/xDg==",
      // 成功回调
    "success":function(res){},
     // 失败回调
    "fail":function(res){},
    // 完成回调
    "complete":function(res){}
  }
)
```

###### java代码

```java
/**
     * 小程序支付
     *
     * @param orderNum    商户订单号
     * @param total       金额，单位 元
     * @param description 商品描述
     * @param openid      微信用户的openid
     * @return
     */
    public JSONObject pay(String orderNum, BigDecimal total, String description, String openid) throws Exception {
        //统一下单，生成预支付交易单
        String bodyAsString = jsapi(orderNum, total, description, openid);
        //解析返回结果
        JSONObject jsonObject = JSON.parseObject(bodyAsString);
        System.out.println(jsonObject);
        // 获取prepayId
        String prepayId = jsonObject.getString("prepay_id");
        // 获取到prepayId
        if (prepayId != null) {
            /*
            * 获取当前时间戳并除以1000时间戳，标准北京时间，时区为东八区，
            * 自1970年1月1日 0点0分0秒以来的秒数。注意：部分系统取到的值为毫秒级，
            * 需要转换成秒(10位数字)。
            * */
            String timeStamp = String.valueOf(System.currentTimeMillis() / 1000);
            // 随机生成长度为32的字符串
            String nonceStr = RandomStringUtils.randomNumeric(32);
            ArrayList<Object> list = new ArrayList<>();
            /*
            * 将小程序appId，时间戳，随机字符串，订单详情扩展字符串，生成签名值
            * */
            list.add(weChatProperties.getAppid());
            list.add(timeStamp);
            list.add(nonceStr);
            list.add("prepay_id=" + prepayId);
            //二次签名，调起支付需要重新签名
            StringBuilder stringBuilder = new StringBuilder();
            for (Object o : list) {
                stringBuilder.append(o).append("\n");
            }
            String signMessage = stringBuilder.toString();
            byte[] message = signMessage.getBytes();
            /*
            * Signature 签名类
            * getInstance(String algorithm)  返回实现指定签名算法的 Signature 对象。
            * 使用商户私钥对待签名串进行SHA256 with RSA签名
            * */
            Signature signature = Signature.getInstance("SHA256withRSA");
            // 读取商户私钥文件
            // initSign(PrivateKey privateKey, SecureRandom random) 初始化这个用于签名的对象
            // PemUtil.loadPrivateKey 加载证书的工具类
            signature.initSign(PemUtil.loadPrivateKey(new FileInputStream(new File(weChatProperties.getPrivateKeyFilePath()))));
            // update(byte[] data) 使用指定的 byte 数组更新要签名或验证的数据。
            signature.update(message);
            // 并对签名结果进行Base64编码得到签名值
            String packageSign = Base64.getEncoder().encodeToString(signature.sign());

            //构造数据给微信小程序，用于调起微信支付
            JSONObject jo = new JSONObject();
            jo.put("timeStamp", timeStamp);
            jo.put("nonceStr", nonceStr);
            jo.put("package", "prepay_id=" + prepayId);
            jo.put("signType", "RSA");
            jo.put("paySign", packageSign);

            return jo;
        }
        return jsonObject;
    }
```

将处理后的jo对象返回给前端

```java

JSONObject jsonObject = weChatPayUtil.pay(
                ordersPaymentDTO.getOrderNumber(), //商户订单号
                new BigDecimal(0.01), //支付金额，单位 元
                "商品描述", //商品描述
                user.getOpenid() //微信用户的openid
        );

        if (jsonObject.getString("code") != null && jsonObject.getString("code").equals("ORDERPAID")) {
            throw new OrderBusinessException("该订单已支付");
        }

        OrderPaymentVO vo = jsonObject.toJavaObject(OrderPaymentVO.class);
        vo.setPackageStr(jsonObject.getString("package"));

        return vo;
```



##### 微信向我们服务器请求

<span style="color:red;">注意：我们的服务器必须部署在公网</span>

微信会向我们之前填写的  notify_url 地址发送请求

处理方式：

```java
/**
     * 支付成功回调
     *
     * @param request
     */
    @RequestMapping("/paySuccess")
    public void paySuccessNotify(HttpServletRequest request, HttpServletResponse response) throws Exception {
        //读取数据
        String body = readData(request);
        log.info("支付成功回调：{}", body);

        //数据解密,解密resource对象
        String plainText = decryptData(body);
        log.info("解密后的文本：{}", plainText);

        JSONObject jsonObject = JSON.parseObject(plainText);
        String outTradeNo = jsonObject.getString("out_trade_no");//商户平台订单号
        String transactionId = jsonObject.getString("transaction_id");//微信支付交易号

        log.info("商户平台订单号：{}", outTradeNo);
        log.info("微信支付交易号：{}", transactionId);

        //业务处理，修改订单状态、来单提醒
        orderService.paySuccess(outTradeNo);

        //给微信响应
        responseToWeixin(response);
    }

    /**
     * 读取数据
     *
     * @param request
     * @return
     * @throws Exception
     */
    private String readData(HttpServletRequest request) throws Exception {
        BufferedReader reader = request.getReader();
        StringBuilder result = new StringBuilder();
        String line = null;
        while ((line = reader.readLine()) != null) {
            if (result.length() > 0) {
                result.append("\n");
            }
            result.append(line);
        }
        return result.toString();
    }

    /**
     * 数据解密
     *
     * @param body
     * @return
     * @throws Exception
     */
    private String decryptData(String body) throws Exception {
        JSONObject resultObject = JSON.parseObject(body);
        JSONObject resource = resultObject.getJSONObject("resource");
        String ciphertext = resource.getString("ciphertext");
        String nonce = resource.getString("nonce");
        String associatedData = resource.getString("associated_data");
		
        AesUtil aesUtil = new AesUtil(weChatProperties.getApiV3Key().getBytes(StandardCharsets.UTF_8));
        //密文解密
        String plainText = aesUtil.decryptToString(associatedData.getBytes(StandardCharsets.UTF_8),
                nonce.getBytes(StandardCharsets.UTF_8),
                ciphertext);

        return plainText;
    }

    /**
     * 给微信响应
     * @param response
     */
    private void responseToWeixin(HttpServletResponse response) throws Exception{
        response.setStatus(200);
        HashMap<Object, Object> map = new HashMap<>();
        map.put("code", "SUCCESS");
        map.put("message", "SUCCESS");
        response.setHeader("Content-type", ContentType.APPLICATION_JSON.toString());
        response.getOutputStream().write(JSONUtils.toJSONString(map).getBytes(StandardCharsets.UTF_8));
        response.flushBuffer();
    }
```

### 退款处理

#### 	请求地址

```
https://api.mch.weixin.qq.com/v3/refund/domestic/refunds
```

#### 请求头

| key              | value                   |
| ---------------- | ----------------------- |
| Accept           | application/json        |
| Content-Type     | application/json        |
| Wechatpay-Serial | 商户API证书的证书序列号 |

#### 请求参数

```json
{
    "out_trade_no" : "商户订单号",
    "out_refund_no" : "商户系统内部的退款单号，商户系统内部唯一",
    "reason" : "退款原因",
    "notify_url" : "退款结果回调url",
    "funds_account" : "AVAILABLE",
    "amount" : {
        // 退款金额
      "refund" : 888,
        // 【原订单金额】 原支付交易的订单总金额，单位为分，只能为整数。
      "total" : 888,
        // 币种
      "currency" : "CNY"
    }
  }
```



