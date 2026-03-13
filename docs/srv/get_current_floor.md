# GetCurrentFloor

**服务类型**: `map_manager/srv/GetCurrentFloor`

**服务名**: `/map_manager/get_current_floor`

## 描述

获取当前活跃楼层信息

## 请求 (Request)

*无请求参数*

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor | Floor | 楼层信息 |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/get_current_floor map_manager/srv/GetCurrentFloor
```
