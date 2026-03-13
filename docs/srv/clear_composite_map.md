# ClearCompositeMap

**服务类型**: `map_manager/srv/ClearCompositeMap`

**服务名**: `/map_manager/clear_composite_map`

## 描述

清除当前加载的复合地图

## 请求 (Request)

*无请求参数*

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/clear_composite_map map_manager/srv/ClearCompositeMap
```
