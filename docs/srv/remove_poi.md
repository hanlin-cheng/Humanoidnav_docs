# RemovePOI

**服务类型**: `map_manager/srv/RemovePOI`

**服务名**: `/map_manager/remove_poi`

## 描述

删除指定 POI

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| id | string | 唯一标识符 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/remove_poi map_manager/srv/RemovePOI \
  "{floor_id: '1F', id: 'example_id'}"
```
