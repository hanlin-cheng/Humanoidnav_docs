# LoadCompositeMap

**服务类型**: `map_manager/srv/LoadCompositeMap`

**服务名**: `/map_manager/load_composite_map`

## 描述

加载复合地图（多楼层地图）。支持目录格式和 `.mmap` 压缩包格式。
加载后会自动初始化所有楼层的定位层、导航层和语义层。

## 请求 (Request)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| map_path | string | 地图路径（目录或 .mmap 文件） |

## 响应 (Response)

| 字段 | 类型 | 描述 |
| ---- | ---- | ---- |
| success | bool | 操作是否成功 |
| message | string | 结果消息或错误信息 |
| map_info | CompositeMapInfo | 加载的地图信息 |

## 使用示例

### 命令行调用

```bash
# 加载目录格式地图
ros2 service call /map_manager/load_composite_map map_manager/srv/LoadCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/office'}"

# 加载压缩包格式地图
ros2 service call /map_manager/load_composite_map map_manager/srv/LoadCompositeMap \
  "{map_path: '/opt/fftai/Navigation/Map/office.mmap'}"
```

### Python 示例

```python
from map_manager.srv import LoadCompositeMap
import rclpy
from rclpy.node import Node

class MapLoader(Node):
    def __init__(self):
        super().__init__('map_loader')
        self.client = self.create_client(
            LoadCompositeMap, '/map_manager/load_composite_map')
        self.client.wait_for_service()

    def load(self, path: str):
        request = LoadCompositeMap.Request()
        request.map_path = path

        future = self.client.call_async(request)
        rclpy.spin_until_future_complete(self, future)
        result = future.result()

        if result.success:
            info = result.map_info
            print(f"地图加载成功: {info.name}")
            print(f"楼层数: {len(info.floors)}")
            for floor in info.floors:
                print(f"  - {floor.floor_id}: {floor.name}")
        else:
            print(f"加载失败: {result.message}")
        return result

rclpy.init()
loader = MapLoader()
loader.load('/opt/fftai/Navigation/Map/office')
rclpy.shutdown()
```

### C++ 示例

```cpp
#include <rclcpp/rclcpp.hpp>
#include <map_manager/srv/load_composite_map.hpp>

class MapLoader : public rclcpp::Node {
public:
    MapLoader() : Node("map_loader") {
        client_ = create_client<map_manager::srv::LoadCompositeMap>(
            "/map_manager/load_composite_map");
        client_->wait_for_service();
    }

    bool load(const std::string& path) {
        auto request = std::make_shared<map_manager::srv::LoadCompositeMap::Request>();
        request->map_path = path;

        auto future = client_->async_send_request(request);
        if (rclcpp::spin_until_future_complete(shared_from_this(), future) ==
            rclcpp::FutureReturnCode::SUCCESS) {
            auto result = future.get();
            if (result->success) {
                RCLCPP_INFO(get_logger(), "地图加载成功: %s, 楼层数: %zu",
                    result->map_info.name.c_str(), result->map_info.floors.size());
            }
            return result->success;
        }
        return false;
    }

private:
    rclcpp::Client<map_manager::srv::LoadCompositeMap>::SharedPtr client_;
};
```

## 支持的地图格式

### 目录格式
```
map_directory/
├── map_info.json           # 复合地图元信息
├── floors/
│   ├── F1/
│   │   ├── localization/   # 定位层 (3D点云)
│   │   ├── navigation/     # 导航层 (2D栅格)
│   │   ├── semantic/       # 语义层 (POI等)
│   │   └── virtual/        # 虚拟层 (虚拟墙等)
│   └── F2/
│       └── ...
└── transitions.json        # 楼层过渡点
```

### 压缩包格式 (.mmap)
`.mmap` 是目录格式的 tar.gz 压缩包，加载时自动解压。

## 相关接口

- [SaveCompositeMap](save_composite_map.md) - 保存复合地图
- [ClearCompositeMap](clear_composite_map.md) - 清除复合地图
- [CompositeMapInfo](../msg/composite_map_info.md) - 复合地图信息消息
