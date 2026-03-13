## fourier_msgs/msg/BaseErrorInfo

### 文件

fourier_msgs/msg/BaseErrorInfo.msg

### 描述

基础错误信息消息，用于描述单个错误的详细信息。

### 定义

| 元素 | 类型 | 描述 |
| ---- | ---- | ---- |
| error_code | int32 | 完整的32位错误代码（如 0x02040100） |
| level | int32 | 错误级别 |
| component | int32 | 组件类别 |
| message | string | 错误描述信息 |
| timestamp | int64 | 时间戳 |

### 错误级别定义

| 级别 | 值 | 说明 |
| ---- | -- | ---- |
| Healthy | 0 | 健康状态 |
| Warn | 1 | 警告 |
| Error | 2 | 错误 |
| Fatal | 4 | 致命错误 |

### 组件类别定义

| 组件 | 值 | 说明 |
| ---- | -- | ---- |
| System | 1 << 16 | 系统组件 |
| Motion | 2 << 16 | 运动组件 |
| Sensor | 3 << 16 | 传感器组件 |
| Software | 4 << 16 | 软件组件 |

### 已定义的错误码

| 错误码 | 十六进制值 | 级别 | 组件 | 说明 |
| ------ | ---------- | ---- | ---- | ---- |
| SENSOR_ERROR_LIDAR_DISCONNECTED | 0x02030100 | Error | 传感器 | Lidar 已断开连接 |
| SOFTWARE_ERROR_RELOC_FAILED | 0x02040100 | Error | 软件 | 重新定位失败 |
| SOFTWARE_ERROR_LOW_LOCALIZATION_QUALITY | 0x02040300 | Error | 软件 | 定位质量低 |
| SOFTWARE_ERROR_LOCALIZATION_FAILED | 0x02040400 | Error | 软件 | 定位失败 |

### 相关消息

- [HealthInfo](health_info.md) - 聚合健康状态
