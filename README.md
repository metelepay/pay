# MetePay文档

## 下单

#### 基础信息
|  描述 |  信息  |
| ------------ | ------------ |
| API地址  | https://metelep.xyz/api/v1/order/create   |
| 请求方式  |  POST  |
| 接口数据  |  JSON  |

#### 请求参数

|  字段 |  类型  |  必须  |  描述  |
| ------------ | ------------ | ------------ | ------------ |
| appId | String | 是 | 申请的appId |
| method | OrderMethod | 是 | 请求下单的方式，详情见下方OrderMethod描述 |
| totalAmount | BigDecimal | 是 | 请求下单的金额，2位小数，至少1元 ，如1.00 |
| outTradeNo | String | 是 | 商户系统的订单号 |
| sign | String | 是 | 签名，详情见最下方签名过程 |

##### OrderMethod类型
|  键 |  说明  | 状态   |
| ------------ | ------------ | ------------ |
| ALIPAY_WEB  | 支付宝PC   | 支持   |
| ALIPAY_WAP  |  支付宝H5  | 支持   |
| ALIPAY_F2F  |  支付宝扫码  | 支持   |
| WXPAY       |   微信扫码 | 支持   |

#### 返回参数
如正确下单，则会返回以下参数

|  字段 |  类型  |  必须  |  描述  |
| ------|------ | ------|------ |
| url  | String | 是 | 支付的url地址   |
| tradeNo  | String | 是 | MetePay的订单号   |
| type  |  String | 是 | 该url需要做的操作，值为redirect或qrcode，分别对应跳转url支付和展现二维码支付  |


## 订单查询

#### 基础信息
|  描述 |  信息  |
| ------------ | ------------ |
| API地址  | https://metelep.xyz/api/v1/order/query   |
| 请求方式  |  POST  |
| 接口数据  |  JSON  |

|  字段 |  类型  |  必须  |  描述  |
| ------------ | ------------ | ------------ | ------------ |
| appId | String | 是 | 申请的appId |
| method | OrderMethod | 是 | 请求下单的方式 |
| outTradeNo | String | 是 | 商户系统的订单号 |
| sign | String | 是 | 签名，详情见最下方签名过程 |

#### 返回参数
如正确下单，则会返回以下参数

|  字段 |  类型  |  必须  |  描述  |
| ------|------ | --------|---- |
| appId | String | 是 | 申请的appId |
| tradeNo  | String | 是 | MetePay的订单号   |
| outTradeNo | String | 是 | 商户系统的订单号 |
| totalAmount | BigDecimal | 是 | 下单金额 |
| createTime | Date | 是 | 创建时间 |
| tradeStatus  |  OrderStatus | 是 | 订单状态  |

##### OrderStatus类型
|  键 |  说明  |
| ------------ | ------------ |
| TRADE_PENDING  | 待付款   |
| TRADE_CANCELED  |  已取消  |
| TRADE_SUCCESS  |  已支付/已完成  |
| TRADE_REFUND_PROCESSING  |  退款中  |
| TRADE_REFUND  |  有退款  |

---

## 通知回调

#### 验签
查看最下方的签名过程，把所有参数去掉sign后，进行ascii升序排序并且按照k1=v1&k2=v2的逻辑去生成sign，并且与MetePay通知回调中返回的sign的值进行比较来验签，商户自己处理完逻辑后返回字符串success给MetePay，则MetePay不会再次发送通知回调给商户
#### 参数
|  字段 |  类型  |  必须  |  描述  |
| ------------ | ------------ | ------------ | ------------ |
| appId | String | 是 | 申请的appId |
| totalAmount | BigDecimal | 是 | 请求下单的金额 2位小数，如1.00 |
| outTradeNo | String | 是 | 商户系统的订单号 |
| tradeNo | String | 是 | MetePay支付系统的订单号 |
| tradeStatus | OrderStatus | 是 | 订单状态 |
| sign | String | 是 | 签名，详情见下方签名过程 |

---

## 签名过程

1. 将需要签名的参数，以k1=v1&k2=v2 按照ASCII升序排序得到字符串signStr
2. 用md5加密算法对signStr进行加密得到md5Str
3. md5Str与该下单的appId设置的密钥进行拼接得到finalMd5Str
4. 对finalMd5再次进行md5加密，得到sign的值
   举例，假设密钥为123456
   如参数为以下json

```json
{"appId":"1635621825763438594","method":"ALIPAY_F2F","outTradeNo":"20230427200222","totalAmount":"1.00"}
```

则signStr为

```json
appId=1635621825763438594&method=ALIPAY_F2F&outTradeNo=20230427200222&totalAmount=1.00
```

md5Str为

```json
096c74e81a8e795a437d141e0027ce9d
```

finalMd5Str为

```json
096c74e81a8e795a437d141e0027ce9d123456
```

sign为

```json
867bd03fe384388285bb74e16337fad9
```

最终的json数据为

```json
{"appId":"1635621825763438594","method":"ALIPAY_F2F","outTradeNo":"20230427200222","sign":"867bd03fe384388285bb74e16337fad9","totalAmount":"1.00"}
```
