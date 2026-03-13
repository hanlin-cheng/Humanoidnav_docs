# UpdateVirtualWall

**服务类型**: `map_manager/srv/UpdateVirtualWall`

**服务名**: `/map_manager/update_virtual_wall`

## 描述

更新虚拟墙位置

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| wall | VirtualWall | 虚拟墙数据 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/update_virtual_wall map_manager/srv/UpdateVirtualWall \
  "{}"
```
