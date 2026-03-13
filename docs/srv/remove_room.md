# RemoveRoom

**服务类型**: `map_manager/srv/RemoveRoom`

**服务名**: `/map_manager/remove_room`

## 描述

删除指定房间

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| room_id | string | 房间 ID |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| objects_removed | int32 | 删除的对象数量 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/remove_room map_manager/srv/RemoveRoom \
  "{floor_id: '1F', room_id: ''}"
```
