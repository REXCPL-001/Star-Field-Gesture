# 手势控制星域粒子系统

基于 Web 的 3D 粒子星域，通过摄像头实时手势控制视野与粒子状态。

## 快速开始

1. 用浏览器打开 `star-field-gesture.html`
2. 允许摄像头权限
3. 等待手势识别模型加载完成（约 3-5 秒）
4. 伸出手掌开始交互

> 需要联网加载 Three.js、MediaPipe Hands 和 Camera Utils。

## 手势操作

| 手势 | 效果 |
|------|------|
| 手靠近屏幕 | 视野拉近（手掌在画面中变大） |
| 手远离屏幕 | 视野拉远（手掌在画面中变小） |
| 手掌左右 / 上下移动 | 视野跟随平移 |
| 握拳 | 粒子向中心聚拢，同时摄像机后退 |

松开拳头后粒子会缓慢回到原始散布位置。

## 技术架构

```
star-field-gesture.html
├── 渲染层: Three.js + 自定义 GLSL Shader
│   ├── 30,000 粒子分布在球形空间 (r = 2 ~ 14)
│   ├── Additive Blending 叠加发光
│   └── 顶点着色器: 透视缩放 + 距离衰减
│   └── 片段着色器: glow + halo + core 三层光效
├── 手势层: MediaPipe Hands
│   ├── 单手 21 关键点追踪
│   ├── 手掌位置 (landmark 9) → 摄像机 XY 平移
│   ├── 手掌尺寸 (landmark 0↔9 距离) → 摄像机 Z 远近
│   └── 手指弯曲度 → 握拳检测 → 聚拢力
└── 物理层: 粒子运动
    ├── 回归力: 弹性回到 homePos
    ├── 聚拢力: 向心衰减力 (convergeRadius = 2.0)
    ├── 漂移力: 微弱正弦扰动
    └── 阻尼: 速度 × 0.97 每帧
```

## 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `N` | 30000 | 粒子数量 |
| `CAMERA_Z_MIN` | 1.5 | 摄像机最近距离 |
| `CAMERA_Z_MAX` | 30.0 | 摄像机最远距离 |
| `PAN_RANGE` | 6.0 | XY 平移灵敏度 |
| `HAND_SIZE_MIN` | 0.08 | 手掌最小尺寸（远） |
| `HAND_SIZE_MAX` | 1.0 | 手掌最大尺寸（近） |
| `convergeRadius` | 2.0 | 聚拢目标球面半径 |
| `convergeStrength` | 0.015 | 聚拢力强度 |
| `homeStrength` | 0.004 | 回归力强度 |
| 速度阻尼 | 0.97 | 每帧速度衰减系数 |

## HUD 显示

左上角实时显示：
- **镜头 X / Y / Z**: 摄像机三维位置
- **聚拢力**: 当前握拳程度百分比
- **粒子数**: 总粒子数量
- **手势状态**: 当前识别到的手势图标与描述

## 浏览器兼容性

- Chrome 90+
- Edge 90+
- Firefox 不支持（MediaPipe Hands 依赖 Chromium 内核）

## 文件结构

```
├── star-field-gesture.html    # 主程序（单文件，无需构建）
├── hand-particles.html        # 早期 2D 版本参考
├── 3d-particle-system.html    # 早期双手版本参考
└── README.md
```
