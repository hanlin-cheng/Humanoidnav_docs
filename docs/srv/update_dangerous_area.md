# UpdateDangerousArea

**服务类型**: `map_manager/srv/UpdateDangerousArea`

**服务名**: `/map_manager/update_dangerous_area`

## 描述

更新限速区边界和速度限制

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| area | DangerousArea | 区域数据 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/update_dangerous_area map_manager/srv/UpdateDangerousArea \
  "{floor_id: '1F'}"
```
