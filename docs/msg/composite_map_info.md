# CompositeMapInfo

**消息类型**: `map_manager/msg/CompositeMapInfo`

## 描述

复合地图元信息，包含地图 ID、楼层列表和过渡点

## 字段定义

### 字段

| 字段名 | 类型 | 描述 |
| ------ | ---- | ---- |
| map_id | string | 地图唯一标识符 |
| name | string | 楼层可读名称 |
| version | string | 地图版本号 |
| created_at | builtin_interfaces/Time | 地图创建时间戳 |
| modified_at | builtin_interfaces/Time | 地图最后修改时间戳 |
| origin | geometry_msgs/Pose | 地图全局原点位姿 |
| floors | Floor[] | 楼层列表 |
| transitions | FloorTransition[] | 楼层过渡点列表 |
| root_path | string | 地图根目录路径 |

## 使用示例

*请参考 map_manager API 文档获取使用示例*
## 相关接口

- [LoadCompositeMap](../srv/load_composite_map.md) - 加载复合地图
- [SaveCompositeMap](../srv/save_composite_map.md) - 保存复合地图
- [ClearCompositeMap](../srv/clear_composite_map.md) - 清除复合地图
