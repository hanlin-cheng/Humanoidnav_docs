# RemoveForbiddenArea

**服务类型**: `map_manager/srv/RemoveForbiddenArea`

**服务名**: `/map_manager/remove_forbidden_area`

## 描述

删除指定禁区

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| area_id | int32 | 区域 ID |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| walls_removed | int32 | 删除的虚拟墙数量 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/remove_forbidden_area map_manager/srv/RemoveForbiddenArea \
  "{floor_id: '1F', area_id: 0}"
```
