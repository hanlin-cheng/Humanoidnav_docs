# ClearFloor

**服务类型**: `map_manager/srv/ClearFloor`

**服务名**: `/map_manager/clear_floor`

## 描述

清除楼层数据（保留楼层结构）

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/clear_floor map_manager/srv/ClearFloor \
  "{floor_id: '1F'}"
```
