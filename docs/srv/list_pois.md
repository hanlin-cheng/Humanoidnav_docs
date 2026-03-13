# ListPOIs

**服务类型**: `map_manager/srv/ListPOIs`

**服务名**: `/map_manager/list_po_is`

## 描述

列出 POI（可按楼层过滤）

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| pois | POI[] | POI 列表 |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/list_po_is map_manager/srv/ListPOIs \
  "{floor_id: '1F'}"
```
