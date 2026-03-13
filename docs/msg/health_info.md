## fourier_msgs/msg/HealthInfo

### 文件

fourier_msgs/msg/HealthInfo.msg

### 描述

聚合后的系统健康状态消息，由 `health_node` 发布，包含所有组件的健康信息汇总。

### 定义

| 元素 | 类型 | 描述 |
| ---- | ---- | ---- |
| errors | BaseErrorInfo[] | 所有错误列表 |
| has_warning | bool | 是否存在警告级别错误 |
| has_error | bool | 是否存在错误级别错误 |
| has_fatal | bool | 是否存在致命级别错误 |

### 话题信息

- **话题名**: `/Humanoid_nav/health`
- **发布节点**: `health_node`
- **发布频率**: 10 Hz（可配置）

### 相关消息

- [BaseErrorInfo](base_error_info.md) - 基础错误信息
