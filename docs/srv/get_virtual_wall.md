# GetVirtualWall

**服务类型**: `map_manager/srv/GetVirtualWall`

**服务名**: `/map_manager/get_virtual_wall`

## 描述

获取指定虚拟墙信息

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| id | int32 | 唯一标识符 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| wall | VirtualWall | 虚拟墙数据 |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/get_virtual_wall map_manager/srv/GetVirtualWall \
  "{floor_id: '1F', id: 'example_id'}"
```
