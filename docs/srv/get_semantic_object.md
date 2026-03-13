# GetSemanticObject

**服务类型**: `map_manager/srv/GetSemanticObject`

**服务名**: `/map_manager/get_semantic_object`

## 描述

获取指定语义对象信息

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| id | string | 唯一标识符 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| object | SemanticObject | 语义对象数据 |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/get_semantic_object map_manager/srv/GetSemanticObject \
  "{floor_id: '1F', id: 'example_id'}"
```
