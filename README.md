# MatePay文档

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
| totalAmount | BigDecimal 或 String | 是 | 请求下单的金额，2位小数，至少1元 ，如1.00 |
| outTradeNo | String | 是 | 商户系统的订单号 |
| sign | String | 是 | 签名，详情见下方签名过程 |

##### OrderMethod类型
|  键 |  说明  |
| ------------ | ------------ |
| ALIPAY_WEB  | 支付宝PC   |
| ALIPAY_WAP  |  支付宝H5  |
| ALIPAY_F2F  |  支付宝扫码  |

##### 签名过程

1. 将需要签名的参数，以k1=v1&k2=v2 按照ASCII升序排序得到字符串signStr
2. 用md5加密算法对signStr进行加密得到md5Str
3. md5Str与该下单的appId设置的密钥进行拼接得到finalMd5Str
4. 对finalMd5再次进行md5加密，得到sign的值

#### 返回参数
如正确下单，则会返回以下参数

|  键 |  说明  |
| ------------ | ------------ |
| url  | 支付的url地址   |
| type  |  该url需要做的操作，值为redirect或qrcode，分别对应跳转url支付和展现二维码支付  |


---

### 通知回调

#### 验签
也是按上方签名过程，把所有参数去掉sign后，进行ascii升序排序并且按照k1=v1&k2=v2的逻辑去生成sign，并且与MatePay通知回调中返回的sign的值进行比较来验签
#### 参数
|  字段 |  类型  |  必须  |  描述  |
| ------------ | ------------ | ------------ | ------------ |
| appId | String | 是 | 申请的appId |
| totalAmount | BigDecimal 或 String | 是 | 请求下单的金额 |
| outTradeNo | String | 是 | 商户系统的订单号 |
| tradeNo | String | 是 | MatePay支付系统的订单号 |
| tradeStatus | OrderState | 是 | 订单状态，详情见下方OrderState描述 |
| sign | String | 是 | 签名，详情见下方签名过程 |

##### OrderState类型
|  键 |  说明  |
| ------------ | ------------ |
| PENDING  | 待付款   |
| SUCCESS  |  已支付/已完成  |
| CANCELED  |  已取消  |
| REFUND  |  有退款  |
| REFUND_PROCESSING  |  退款中  |
| PROCESSING  |  处理中  |
| FROZEN  |  冻结中  |
