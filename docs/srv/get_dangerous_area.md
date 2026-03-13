# GetDangerousArea

**服务类型**: `map_manager/srv/GetDangerousArea`

**服务名**: `/map_manager/get_dangerous_area`

## 描述

获取指定限速区信息

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| id | int32 | 唯一标识符 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| area | DangerousArea | 区域数据 |
| area_floor_id | string | 区域所在楼层 ID |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/get_dangerous_area map_manager/srv/GetDangerousArea \
  "{floor_id: '1F', id: 'example_id'}"
```
