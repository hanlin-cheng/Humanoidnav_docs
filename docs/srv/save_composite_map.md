# SaveCompositeMap

**服务类型**: `map_manager/srv/SaveCompositeMap`

**服务名**: `/map_manager/save_composite_map`

## 描述

保存复合地图到磁盘。支持保存为目录格式或 `.mmap` 压缩包格式。
保存时会同时保存所有楼层的定位层、导航层、语义层和虚拟层数据。

## 请求 (Request)

| 字段 | 类型 | 默认值 | 描述 |
| ---- | ---- | ------ | ---- |
| map_path | string | - | 保存路径 |
| compress | bool | false | 是否压缩为 .mmap 格式 |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| saved_path | string | 实际保存路径（可能与输入不同） |

## 使用示例

### 命令行调用

```bash
# 保存为目录格式
ros2 service call /map_manager/save_composite_map map_manager/srv/SaveCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/my_map', compress: false}"

# 保存为压缩包格式
ros2 service call /map_manager/save_composite_map map_manager/srv/SaveCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/my_map', compress: true}"

# 路径以 .mmap 结尾时自动压缩
ros2 service call /map_manager/save_composite_map map_manager/srv/SaveCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/my_map.mmap'}"
```

### Python 示例

```python
from map_manager.srv import SaveCompositeMap
import rclpy
from rclpy.node import Node

class MapSaver(Node):
    def __init__(self):
        super().__init__('map_saver')
        self.client = self.create_client(
            SaveCompositeMap, '/map_manager/save_composite_map')
        self.client.wait_for_service()

    def save(self, path: str, compress: bool = False):
        request = SaveCompositeMap.Request()
        request.map_path = path
        request.compress = compress

        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()

        if result.success:
            print(f"地图保存成功: {result.saved_path}")
        else:
            print(f"保存失败: {result.message}")
        return result

rclpy.init()
saver = MapSaver()

# 保存为目录
saver.save('/opt/fftai/Navigation/Map/my_map', compress=False)

# 保存为压缩包
saver.save('/opt/fftai/Navigation/Map/my_map', compress=True)

rclpy.shutdown()
```

### C++ 示例

```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/save_composite_map.hpp>

class MapSaver : public rclcpp::Node {
public:
    MapSaver() : Node("map_saver") {
        client_ = create_client<map_manager::srv::SaveCompositeMap>(
            "/map_manager/save_composite_map");
        client_->wait_for_service();
    }

    bool save(const std::string& path, bool compress = false) {
        auto request = std::make_shared<map_manager::srv::SaveCompositeMap::Request>();
        request->map_path = path;
        request->compress = compress;

        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "地图保存成功: %s", result->saved_path.c_str());
            }
            return result->success;
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::SaveCompositeMap>::SharedPtr client_;
};
```

## 压缩格式说明

- **compress=false**: 保存为目录结构，便于查看和编辑
- **compress=true**: 保存为 `.mmap` 文件（tar.gz 压缩），便于传输和部署
- 如果 `map_path` 以 `.mmap` 结尾，自动设置 `compress=true`

## 保存内容

| 图层 | 内容 | 格式 |
| ---- | ---- | ---- |
| localization | 3D 点云地图 | .pcd 文件 |
| navigation | 2D 栅格地图 | .pgm + .yaml |
| semantic | POI、房间、语义对象 | .json |
| virtual | 虚拟墙、禁区、限速区 | .json |

## 相关接口

- [LoadCompositeMap](load_composite_map.md) - 加载复合地图
- [ClearCompositeMap](clear_composite_map.md) - 清除复合地图
- [SaveFloor](save_floor.md) - 保存单个楼层
