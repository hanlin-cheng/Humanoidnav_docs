# AddSemanticObject

**服务类型**: `map_manager/srv/AddSemanticObject`

**服务名**: `/map_manager/add_semantic_object`

## 描述

添加语义对象（如家具、设备等）

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| object | SemanticObject | 语义对象数据 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| object_id | string | 对象 ID |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/add_semantic_object map_manager/srv/AddSemanticObject \
  "{floor_id: '1F'}"
```
