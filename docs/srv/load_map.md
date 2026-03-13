## fourier_msgs/srv/LoadMap

### 文件

fourier_msgs/srv/LoadMap.srv

### 描述

加载指定地图并自动切换到定位模式。系统将：
1. 停止当前建图或定位模式
2. 加载指定地图
3. 设置初始位姿
4. 启动定位模式

### 请求

| 元素 | 类型 | 描述 |
| ---- | ---- | ---- |
| map_path | string | 地图路径，支持：<br/>**绝对路径**: 以 `/` 开头<br/>**相对路径**: 相对于默认地图目录<br/>*地图目录应包含 `global.pcd` 和 `map.yaml`/`map.pgm`* |
| x | float64 | 初始 x 位置（米） |
| y | float64 | 初始 y 位置（米） |
| z | float64 | 初始 z 位置（米） |
| yaw | float64 | 初始 yaw 角度（弧度） |

### 响应

| 元素 | 类型 | 描述 |
| ---- | ---- | ---- |
| result | uint32 | 结果码：<br/>`0`: 成功<br/>`1`: 地图未找到<br/>`2`: 地图加载失败<br/>`3`: 模式切换失败 |
| message | string | 结果描述信息 |

### 地图文件结构

加载的地图目录应包含以下文件：

```
map_directory/
├── global.pcd        # 3D点云地图（必需）
├── map.yaml          # 2D地图配置文件
└── map.pgm           # 2D占用栅格图像
```
