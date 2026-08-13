# LCC Plugin for UE — 虚幻引擎 3D 高斯泼溅插件

> 3D Gaussian Splatting / 3DGS / UE5插件 / 高斯泼溅 / 点云渲染 / 辐射场 / 虚拟制片 / 数字孪生 / NeRF替代 / 实时渲染 / VR / XR

[English](./README.md) | 中文

[![Discord](https://img.shields.io/badge/Discord-加入社区-5865F2?logo=discord&logoColor=white)](https://discord.gg/R8ysxTfDj)
[![Forum](https://img.shields.io/badge/论坛-开发者社区-orange)](https://developer.xgrids.cn/#/forum)
[![Website](https://img.shields.io/badge/官网-xgrids.com-blue)](https://xgrids.com/intl/lccUE)
[![UE5](https://img.shields.io/badge/Unreal%20Engine-5.1--5.8-black?logo=unrealengine)](https://xgrids.com/intl/lccUE)
[![Platform](https://img.shields.io/badge/平台-Windows%20|%20Linux-lightgrey)]()
[![License](https://img.shields.io/badge/许可-免费%20%2B%20Pro-green)]()

## 这是什么？

**LCC Plugin for UE** 是一款面向生产环境的虚幻引擎插件，用于加载、渲染、编辑和重打光 **十亿点级3D高斯泼溅（3D Gaussian Splatting, 3DGS）** 场景——这一技术正是近年神经辐射场和照片级场景重建领域突破性进展的核心。

如果你在使用 **3DGS、高斯泼溅、点云、NeRF风格采集或摄影测量**，并需要将它们带入 **Unreal Engine 5** 进行实时可视化、虚拟制片、仿真或VR体验，这款插件正为此而生。

## 为什么选择 LCC Plugin for UE？

| | LCC Plugin for UE |
|---|---|
| 规模 | 十亿点级3DGS场景一次性完整加载 |
| 格式 | LCC2、LCC、PLY、SOG、SPZ（兼容v2/v3/v4） |
| 性能 | 流式LoD，稳定高帧率 |
| 重打光 | 基于Proxy Mesh代理网格的重打光 |
| 深度 | 原生深度缓冲，支持电影级景深效果 |
| VR | 高保真串流至PICO、Meta Quest等 |
| 虚拟制片 | nDisplay、disguise、Pixotope、Aximmetry |
| 集成 | C++ API + 蓝图，基础使用无需编码 |
| 引擎 | UE 5.1 / 5.2 / 5.3 / 5.4 / 5.5 / 5.6 / 5.7 / 5.8 |
| 平台 | Windows、Linux（交叉编译） |
| 图形API | DirectX 11、DirectX 12、Vulkan |


## 介绍

3D高斯泼溅（3D Gaussian Splatting, 3DGS）是新一代实时渲染技术，通过数百万个3D高斯基元表示场景，能够对从照片或LiDAR扫描采集的真实世界环境进行照片级真实感重建与实时浏览。相比NeRF，3DGS训练和渲染速度更快，视觉质量相当甚至更优。

LCC Plugin for UE 让十亿级高斯泼溅世界进入虚幻引擎。在引擎内导入、编辑、重打光并渲染大规模3DGS场景——适用于影视、数字孪生、仿真、游戏和VR。

基于LCC2的极致压缩技术，插件可将十亿点级3D高斯场景一次性完整加载到虚幻引擎中，保持高保真度的同时覆盖数十万平方米的真实世界空间。

## 应用场景

- **影视与虚拟制片** — 将真实世界3D高斯泼溅场景转化为LED墙可用的虚拟置景。重打光、编辑并添加电影级效果。
- **数字孪生** — 从已采集的环境创建交互式空间数字孪生，用于可视化、设计评审和运维。
- **仿真** — 基于真实世界3DGS采集构建高保真仿真环境，适用于机器人、自动驾驶、CARLA和培训。
- **游戏与VR** — 将摄影测量和高斯泼溅采集转化为沉浸式游戏关卡和VR体验，部署到PICO、Meta Quest等XR设备。
- **建筑与BIM** — 结合Cesium地理空间信息的3DGS采集，或与网格资产配合用于设计评审。

## 核心功能

### 数据与格式
- 多格式3D高斯泼溅支持：LCC2、LCC、PLY、SOG、SPZ（兼容v2/v3/v4，SPZ v2 支持 Marble 导出的高斯）
- 标准3DGS PLY文件导入（支持gsplat、3DGS原版、Nerfstudio、Postshot、Luma、Polycam等输出）
- 无点数限制——可加载十亿级高斯
- LCC2极致压缩，支持大规模真实世界场景
- 支持更大尺寸的 SOG / SPZ / PLY 模型

### 渲染与性能
- 流式Level-of-Detail (LoD)，稳定高帧率
- GPU 驱动间接绘制，提升渲染效率
- NVIDIA DLSS支持
- SingleLayerWater 水面渲染支持
- DirectX 11 / DirectX 12 / Vulkan
- 原生深度渲染
- 抗锯齿开关

### 灯光与后处理
- 基于Proxy Mesh代理网格的重打光（Pro）
- 高斯泼溅自阴影（Pro）
- 基于原生深度缓冲的景深效果
- 曝光控制、色彩校正区域
- 后处理体积(Post Process Volume)支持
- 专业色彩管理：OCIO / ACES工作流（Pro）

### 场景编辑
- 无缝3DGS + 网格资产融合协同编辑
- 多3DGS场景、网格和特效混合渲染
- 裁剪与剖切（Pro版无限制）
- 位移、旋转、缩放
- 全局Alpha控制
- Splat Scale缩放
- 高程着色
- Node Bound可视化

### XR与虚拟制片
- 高保真VR串流（PICO、Meta Quest等主流头显）
- 原生nDisplay支持
- disguise、Pixotope、Aximmetry集成
- LED墙虚拟制片工作流

### 仿真与地理空间
- Cesium for Unreal Engine 支持
- CARLA 仿真器支持
- 碰撞检测（单射线、多射线）
- 导航网格（NavMesh）支持，用于 AI 寻路
- 内置 PROJ 坐标转换库，无需依赖引擎 GeoReferencing 插件
- SOG / SPZ / PLY 自动沿 X 轴旋转，匹配外部高斯泼溅坐标轴定义

### 开发者体验
- 无代码蓝图可视化脚本
- C++原生API，运行时控制
- 控制台命令用于性能分析和调试
- 可配置线程池（遍历、加载、排序）
- LCC文件选择对话框
- 加载动画支持正向 / 反向播放
- 数据动态加载/卸载
- Actor 面板 ShowCollision / ShowFPS 快捷操作
- 视口相机聚焦根组件（按 F 键框选）
- 编辑器多语言本地化（英文、中文、德语、法语、日语、韩语）
- UE 5.1 - 5.8 支持


## 下载

请访问 [developer.xgrids.cn](https://developer.xgrids.cn/#/download?page=LCC_UNREAL_SDK_UE54) 获取最新版本插件。

## 版本定价

| | 免费版 | Pro版（限时免费） |
|---|---|---|
| 多格式3DGS加载 | ✅ | ✅ |
| 渲染与LoD性能 | ✅ | ✅ |
| 深度渲染与景深 | ✅ | ✅ |
| NVIDIA DLSS | ✅ | ✅ |
| 碰撞 | ✅ | ✅ |
| VR串流 | ✅ | ✅ |
| nDisplay | ✅ | ✅ |
| Cesium & CARLA | ✅ | ✅ |
| 基础重打光 | ✅ | ✅ |
| 有限裁剪 | ✅ | ✅ |
| Proxy Mesh代理网格重打光 | — | ✅ |
| 自阴影 | — | ✅ |
| ACES / OCIO色彩管线 | — | ✅ |
| 无限裁剪与剖切 | — | ✅ |

## 文档

- [开发者文档](https://developer.xgrids.cn/#/document?titleId=cn-1720170162723)
- [教程](https://developer.xgrids.cn/#/tutorial?page=UE_SDK)
- [更新日志](./CHANGELOG_zh.md) —— 各版本详情见 [`zh-cn/changelog/`](./zh-cn/changelog)
- [在线更新日志](https://developer.xgrids.cn/#/document?titleId=cn-1720170723795)

## 兼容性

| 引擎 | UE 5.1、5.2、5.3、5.4、5.5、5.6、5.7、5.8 |
|---|---|
| 平台 | Windows、Linux（交叉编译） |
| 图形API | DirectX 11、DirectX 12、Vulkan |
| VR头显 | PICO、Meta Quest等 |
| 虚拟制片 | nDisplay、disguise、Pixotope、Aximmetry |

## 关于XGRIDS其域创新

XGRIDS 致力于构建全球领先的空间智能平台，连接物理世界与数字世界，为AI在真实环境中运行提供基础设施。

100余项发明专利驱动着AI空间智能引擎、多传感器SLAM及3D高斯泼溅工作流。2000+客户覆盖测绘、建筑、文化遗产保护和数字娱乐领域。

## 联系方式

- 邮箱：enterprise@xgrids.com
- 官网：[xgrids.com](https://xgrids.com/intl/lccUE)
- Discord：[discord.gg/R8ysxTfDj](https://discord.gg/R8ysxTfDj)

---

**关键词：** 3D Gaussian Splatting, 3DGS, 高斯泼溅, Unreal Engine插件, UE5插件, UE5 3DGS, 点云渲染, 辐射场, NeRF虚幻引擎, 摄影测量, 虚拟制片, 数字孪生, 实时渲染, VR可视化, nDisplay, 高斯泼溅查看器, 大规模点云, 十亿点渲染, XGRIDS, 其域创新, LCC, 空间计算
