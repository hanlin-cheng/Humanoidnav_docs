# RemoveVirtualWall

**服务类型**: `map_manager/srv/RemoveVirtualWall`

**服务名**: `/map_manager/remove_virtual_wall`

## 描述

删除指定虚拟墙

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| wall_id | int32 | 虚拟墙 ID |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/remove_virtual_wall map_manager/srv/RemoveVirtualWall \
  "{floor_id: '1F', wall_id: 0}"
```
