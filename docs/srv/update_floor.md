# UpdateFloor

**服务类型**: `map_manager/srv/UpdateFloor`

**服务名**: `/map_manager/update_floor`

## 描述

更新楼层信息

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| floor | Floor | 楼层信息 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/update_floor map_manager/srv/UpdateFloor \
  "{floor_id: '1F'}"
```
