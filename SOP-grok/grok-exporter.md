# grok-exporter

## 指标部分

### 指标类型概述

指标部分包含指标定义列表，指定日志行如何映射到 Prometheus 指标。支持四种指标类型：

- [柜台](https://github.com/fstab/grok_exporter/blob/master/CONFIG_v2.md#counter-metric-type)
- [测量](https://github.com/fstab/grok_exporter/blob/master/CONFIG_v2.md#gauge-metric-type)
- [直方图](https://github.com/fstab/grok_exporter/blob/master/CONFIG_v2.md#histogram-metric-type)
- [概括](https://github.com/fstab/grok_exporter/blob/master/CONFIG_v2.md#summary-metric-type)

match是匹配字符串的正则表达式，但实际只是预定义片段（即你想要匹配的内容），实际需要通过patterns来解析完整

