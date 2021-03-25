---
type: docs
title: "Nacos"
linkTitle: "Nacos"
weight: 4000
date: 2021-03-22
description: >
  Nacos 调研
---





- [Open API 指南](https://nacos.io/zh-cn/docs/open-api.html)
- [java sdk](https://nacos.io/zh-cn/docs/sdk.html)



#### 获取配置

```java
public String getConfig(String dataId, String group, long timeoutMs) throws NacosException
```

| 参数名  | 参数类型 | 描述                                                         |
| :------ | :------- | :----------------------------------------------------------- |
| dataId  | string   | 配置 ID，采用类似 package.class（如com.taobao.tc.refund.log.level）的命名规则保证全局唯一性，class 部分建议是配置的业务含义。全部字符小写。只允许英文字符和 4 种特殊字符（"."、":"、"-"、"_"），不超过 256 字节。 |
| group   | string   | 配置分组，建议填写产品名:模块名（Nacos:Test）保证唯一性，只允许英文字符和4种特殊字符（"."、":"、"-"、"_"），不超过128字节。 |
| tenant  | string   | 租户信息，对应 Nacos 的命名空间字段(非必填)                  |
| timeout | long     | 读取配置超时时间，单位 ms，推荐值 3000。                     |



```bash
curl -X GET 'http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.example&group=com.alibaba.nacos' 

```