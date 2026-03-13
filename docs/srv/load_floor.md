# LoadFloor

**服务类型**: `map_manager/srv/LoadFloor`

**服务名**: `/map_manager/load_floor`

## 描述

从磁盘加载楼层数据

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| floor_id | string | 楼层唯一标识 |
| input_path | string | input_path |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| floor_id | string | 楼层唯一标识 |

## 使用示例

### 命令行调用

```bash
ros2 service call /map_manager/load_floor map_manager/srv/LoadFloor \
  "{floor_id: '1F', input_path: ''}"
```
