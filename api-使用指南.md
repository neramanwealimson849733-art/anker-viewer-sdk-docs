# api-使用指南

---

## 目录
- [快速开始](#快速开始)
- [Viewer 总览](#viewer-总览)
- [构造参数](#构造参数)
- [核心模块](#核心模块)
- [API 详解](#api-详解)
  - [基础 API](#基础-api)
  - [模型管理](#模型管理)
  - [相机控制](#相机控制)
  - [场景管理](#场景管理)
  - [后处理效果](#后处理效果)
  - [辅助工具](#辅助工具)
  - [尺寸信息与标注](#尺寸信息与标注)
- [组件系统](#组件系统)
- [事件系统](#事件系统)
- [资源管理](#资源管理)
- [使用示例](#使用示例)
- [常见问题 FAQ](#常见问题-faq)
- [高级功能详解](#高级功能详解)
- [性能优化建议](#性能优化建议)
- [最佳实践](#最佳实践)
- [事件系统详解](#事件系统详解)
- [实用工具函数](#实用工具函数)
- [注意事项](#注意事项)
- [TypeScript 类型定义](#typescript-类型定义)
- [在线示例](#在线示例)
- [更多示例](#更多示例)
- [总结](#总结)

---





# -------------分割线------------

## 快速开始

### 什么是 Anker3D SDK？

Anker3D SDK 是一个用于在网页中显示和操作 3D 模型的工具库。就像视频播放器用来播放视频一样，Anker3D 用来显示和操作 3D 模型。

**主要能力：**
- 📦 加载 3D 模型（支持 GLB/GLTF 格式）
- 🎮 旋转、缩放、平移 3D 模型
- 🌞 设置不同的光线和背景
- ✨ 添加特效（发光、阴影等）
- 📸 截图功能

### 基础使用示例

```html
<!DOCTYPE html>
<html>
<head>
  <title>我的 3D 查看器</title>
  <style>
    #viewer-container {
      width: 100%;
      height: 100vh;  /* 占满整个屏幕高度 */
      margin: 0;
    }
  </style>
</head>
<body>
  <!-- 这个 div 就是用来显示 3D 模型的容器 -->
  <div id="viewer-container"></div>

  <!-- 引入 Anker3D SDK -->
  <script type="module" src="./main.js"></script>
</body>
</html>
```

### 第二步：引入 SDK 并创建查看器

创建 `main.js` 文件：

```javascript
// 方式 1：如果是 ES 模块（推荐）
import * as Anker3D from './dist/anker3d.module.js';

// 或者方式 2：如果是在 HTML 中直接引入
// <script type="module">
//   import * as Anker3D from "./dist/anker3d.module.js";
//   // 下面的代码写在这里
// </script>

// 获取 HTML 中的容器元素
const container = document.getElementById('viewer-container');

// 创建 3D 查看器（这就像创建一个播放器实例）
const viewer = new Anker3D.Viewer(container, {
  // dracoPath 是解码器路径，用于解压缩模型（必需！）
  dracoPath: '/public/draco/',
  // 可选：相机初始位置
  cameraParams: {
    position: [0, 1, 5],  // [x, y, z] 坐标
    fov: 50               // 视角大小
  },
  // 可选：是否添加默认灯光
  defaultLight: false     // false = 不使用默认灯光，使用环境光
});

// 将 viewer 挂载到全局，方便在浏览器控制台调试
window.viewer = viewer;

console.log('查看器创建成功！');
```

### 第三步：加载 3D 模型

现在查看器已创建，接下来加载一个 3D 模型：

```javascript
// 加载模型
viewer.loadModel({
  // 模型的路径（本地文件路径或网络 URL）
  urlOrId: '/public/models/S1pro.glb',
  
  // 环境贴图路径（提供光照效果，模型不会看起来黑黑的）
  envMapUrl: '/public/environments/S1pro.hdr',
  
  // 模型加载成功的回调函数
  onLoad: (model) => {
    console.log('模型加载成功！', model);
    // 现在你可以在页面上看到 3D 模型了！
  },
  
  // 加载进度回调（显示进度条用）
  onProgress: (progress) => {
    console.log('加载进度:', progress + '%');
  },
  
  // 加载失败的回调
  onError: (error) => {
    console.error('加载失败:', error);
    alert('模型加载失败，请检查路径是否正确');
  },
  
  // 模型是否自动居中显示
  isAutoCenter: true,
  
  // 是否显示阴影
  castShadow: true,
  
  // 是否生成底部阴影（让模型看起来稳稳地放在地面上）
  bottomShadow: true
});

console.log('开始加载模型...');
```

### 尺寸信息与标注

在不修改模型几何的前提下，SDK 支持展示产品的“长、宽、高”尺寸标注，并提供单位切换与手动覆盖显示值。

#### 快速上手

```javascript
// 开启尺寸标注（基于模型包围盒自动计算）
model.dimensionVisible = true;

// 切换单位（mm/cm/dm/m/in/ft）
model.changeDimensionUnit(Anker3D.LengthUnit.IN);

// 获取当前显示尺寸（按当前单位返回，保留 0.01 精度）
const dim = model.getDimension();
console.log(dim.length, dim.width, dim.height);
```

#### 手动覆盖显示尺寸

当需要展示与几何无关的“标称尺寸”时，可直接设置显示值：

```javascript
// 以毫米传入
model.setDimensionInfo(120.5, 60.2, 35.0, Anker3D.LengthUnit.MM);

// 或以英寸传入（内部会转换为毫米保存，展示按当前单位）
model.setDimensionInfo(4.75, 2.37, 1.38, Anker3D.LengthUnit.IN);
```

#### 仅读取尺寸（不显示标注）

可使用工具函数直接读取尺寸与中心点：

```javascript
model.computeBoundingBox();          // 确保包围盒最新
const size = Anker3D.Utils.getObjectSize(model);    // Vector3: x/y/z（米）
const center = Anker3D.Utils.getObjectCenter(model);
```

#### 单位与换算

- **内部单位**：Three.js 默认以米为单位；尺寸标注展示默认以毫米显示。
- **支持单位**：mm、cm、dm、m、in（英寸）、ft（英尺）。
- **换算规则**：
  - 写入显示值时统一换算为毫米保存：
    - 1 cm = 10 mm，1 dm = 100 mm，1 m = 1000 mm
    - 1 in = 25.4 mm，1 ft = 304.8 mm
  - 展示时按当前单位从毫米换算：
    - 1 mm → 0.1 cm、0.01 dm、0.001 m、1/25.4 in、1/304.8 ft
- **精度**：展示时四舍五入到 0.01（例如 120.456 → 120.46）。

#### 注意事项

- 布局刷新：尺寸标注的线条与文本位置在首次创建时依据包围盒确定；若后续几何/缩放发生显著变化，建议先关闭再开启以重建标注：

```javascript
model.dimensionVisible = false;
model.computeBoundingBox();
model.dimensionVisible = true;
```

- 不改变几何：`setDimensionInfo` 仅影响显示文本，不会改变模型包围盒或网格数据。
- 性能表现：单位切换或数值变化仅重建文字贴图与文本平面，线条与布局不会重复计算。

#### 示例

- 在线示例：[尺寸标注示例](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=dimensions)

#### 样式与随视角切换

```javascript
// BBox 风格（默认），随相机自动选择“面向用户”的边；第二个参数控制是否固定
model.setDimensionStyle(Anker3D.DimensionStyle.BBox);          // 动态（默认）
model.setDimensionStyle(Anker3D.DimensionStyle.BBox, true);    // 固定不随相机变化

// Corner 风格（三轴从同一拐角出发），同样支持是否固定
model.setDimensionStyle(Anker3D.DimensionStyle.Corner);        // 动态
model.setDimensionStyle(Anker3D.DimensionStyle.Corner, true);  // 固定
```

说明：
- 动态模式下通过六面比较选择“最正对用户”的面，只有当新面显著更正对时才切换（内置抖动阈值），避免小角度抖动。
- BBox 与 Corner 都会将尺寸线外扩到模型外侧，保持清晰的视觉间距。

---

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>我的第一个 3D 查看器</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      font-family: Arial, sans-serif;
    }
    #viewer-container {
      width: 100vw;
      height: 100vh;
    }
    #loading {
      position: absolute;
      top: 20px;
      left: 20px;
      background: rgba(0,0,0,0.7);
      color: white;
      padding: 10px 20px;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <div id="viewer-container"></div>
  <div id="loading">加载中... 0%</div>

  <script type="module">
    import * as Anker3D from './dist/anker3d.module.js';

    // 1. 获取页面元素
    const container = document.getElementById('viewer-container');
    const loadingDiv = document.getElementById('loading');

    // 2. 创建查看器
    const viewer = new Anker3D.Viewer(container, {
      dracoPath: '/public/draco/',
      cameraParams: {
        position: [0, 1, 5],
        fov: 50
      },
      defaultLight: false
    });

    // 3. 加载模型
    viewer.loadModel({
      urlOrId: '/public/models/S1pro.glb',
      envMapUrl: '/public/environments/S1pro.hdr',
      onLoad: (model) => {
        loadingDiv.textContent = '加载完成！';
        loadingDiv.style.background = 'rgba(0,255,0,0.7)';
        console.log('模型加载成功:', model);
        
        // 3 秒后隐藏加载提示
        setTimeout(() => {
          loadingDiv.style.display = 'none';
        }, 3000);
      },
      onProgress: (progress) => {
        loadingDiv.textContent = `加载中... ${progress}%`;
      },
      onError: (error) => {
        loadingDiv.textContent = '加载失败！';
        loadingDiv.style.background = 'rgba(255,0,0,0.7)';
        console.error('加载失败:', error);
      },
      isAutoCenter: true,
      castShadow: true,
      bottomShadow: true
    });

    // 4. 挂载到全局用于调试
    window.viewer = viewer;
    window.Anker3D = Anker3D;

    console.log('3D 查看器已启动！');
    console.log('提示：在控制台输入 viewer 可以查看查看器对象');
  </script>
</body>
</html>
```

### 快速测试

如果一切正常，你现在应该能看到：
1. ✅ 3D 模型显示在页面上
2. ✅ 可以用鼠标拖动旋转模型
3. ✅ 可以用滚轮缩放模型
4. ✅ 可以用右键拖动平移模型

### 在线示例

查看更多示例代码和效果：
- [简单示例](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=simple) - 最简单的查看器示例
- [完整示例](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=loadModel) - 带所有参数的完整示例
- [更多示例](#在线示例) - 查看所有在线示例

---



# -------------分割线------------

## Viewer 总览

`Viewer` 是 Anker3D SDK 的核心类，基于 Three.js 构建，提供完整的 3D 渲染解决方案。它集成了渲染管理、相机控制、场景管理、资源加载等功能模块。

### 继承关系
- `Viewer` 继承自 `BaseViewer`
- `BaseViewer` 继承自 `EventDispatcher`

### 核心特性
- 🎯 **自动模型居中与缩放**
- 🎥 **相机控制与动画**
- 🌍 **HDR 环境贴图支持**
- ✨ **后处理效果（Bloom、SSAO、SSR）**
- 📊 **性能监控与调试**
- 📸 **截图功能**
- 🎮 **交互拾取**

---

## 构造参数

```typescript
new Viewer(container?: HTMLElement, viewerParams?: ViewerParams)
```

### ViewerParams 接口

```typescript
interface ViewerParams {
  cameraParams?: CameraParams;     // 相机参数
  defaultLight?: boolean;          // 是否添加默认灯光，默认 false
  dracoPath?: string;             // Draco 解码器路径，默认 "/public/draco/"
  modelParams?: ModelParams;      // 初始模型参数
}
```

### CameraParams 接口

```typescript
interface CameraParams {
  type?: CameraType;              // 相机类型：Perspective | Orthographic
  position?: number[];            // 相机位置 [x, y, z]
  target?: Vector3;              // 相机目标点
  fov?: number;                  // 视场角，默认 50
  near?: number;                 // 近裁切面，默认 0.1
  far?: number;                  // 远裁切面，默认 1000
  dampingFactor?: number;        // 惯性系数，默认 0.05
  minDistance?: number;          // 最小距离
  maxDistance?: number;          // 最大距离
  enableDamping?: boolean;       // 是否启用阻尼，默认 true
}
```

### CameraType 枚举

```typescript
enum CameraType {
  Perspective = 0,      // 透视相机（默认）
  Orthographic = 1     // 正交相机
}
```

---

## 核心模块

### RenderManager - 渲染管理
- 渲染器管理与配置
- 动画循环控制
- 窗口尺寸变更处理
- 渲染更新事件分发

### CameraManager - 相机管理
- 透视/正交相机切换
- 相机控制器（CameraControls）
- 相机动画：`flyTo`、`moveAlongThePath`
- 视角管理：`getView`/`setView`
- 模型自动聚焦：`lookAtModel`

### SceneManager - 场景管理
- 场景创建、切换、销毁
- 默认场景管理
- 环境贴图加载
- 背景设置

### ResourceManager - 资源管理
- 模型/纹理/HDR 缓存
- 资源自动释放
- Draco 解码器支持
- 异步加载管理

### ComponentManager - 组件管理
- 组件生命周期管理
- 帧更新分发
- 组件添加/移除

---







# -------------分割线------------

## API 详解

### 基础 API

#### Viewer 属性
```typescript
// 核心对象访问
viewer.renderer: WebGLRenderer          // Three.js 渲染器
viewer.camera: PerspectiveCamera | OrthographicCamera  // 当前相机
viewer.scene: SceneObject               // 当前场景
viewer.controls: CameraControls         // 相机控制器
viewer.domElement: HTMLCanvasElement    // 画布元素

// 管理器访问
viewer.renderManager: RenderManager     // 渲染管理器
viewer.cameraManager: CameraManager     // 相机管理器
viewer.sceneManager: SceneManager       // 场景管理器
viewer.componentManager: ComponentManager // 组件管理器
viewer.sceneHelperManager: SceneHelperManager // 辅助器管理器
viewer.picking: Picking                 // 拾取器
```

#### 基础方法
```typescript
// 可见性控制
viewer.visible: boolean                 // 设置查看器可见性

// 尺寸更新
viewer.onResize(): void                 // 手动触发尺寸更新

// 生命周期
viewer.destroy(reserveResource?: boolean): void  // 销毁查看器
```

### 模型管理

模型管理是 SDK 的核心功能之一。你可以加载 3D 模型、设置它们的位置、旋转、缩放等属性。

#### 1. 基础加载模型

最简单的模型加载方式：

```javascript
// 加载本地模型文件
viewer.loadModel({
  urlOrId: '/public/models/product.glb'
});
```

#### 2. 完整加载模型（带所有选项）

```javascript
viewer.loadModel({
  // 【必填】模型路径或ID
  urlOrId: '/public/models/product.glb',  // 本地文件路径
  
  // 【可选】环境贴图（提供光照，让模型看起来有立体感）
  envMapUrl: '/public/environments/studio.hdr',
  
  // 【可选】回调函数
  onLoad: (model) => {
    // 模型加载完成时调用
    console.log('模型加载成功:', model);
    console.log('模型名称:', model.name);
  },
  
  onProgress: (progress) => {
    // progress 是 0-100 的数字
    console.log(`加载进度: ${progress}%`);
  },
  
  onError: (error) => {
    // 加载失败时调用
    console.error('加载失败:', error);
  },
  
  // 【可选】模型位置（[x, y, z]）
  position: [0, 0, 0],
  
  // 【可选】模型旋转（[x, y, z] 单位：度）
  rotation: [0, 0, 0],
  
  // 【可选】模型缩放（[x, y, z]）
  scale: [1, 1, 1],
  
  // 【可选】是否产生阴影
  castShadow: true,
  
  // 【可选】是否自动居中（会自动调整相机位置让模型显示在中心）
  isAutoCenter: true,
  
  // 【可选】自动居中的缩放系数（越大模型显示越小）
  scaleFactor: 1.5,
  
  // 【可选】是否生成底部阴影（让模型看起来放在地面上）
  bottomShadow: true,
  
  // 【可选】是否替换现有模型
  replace: false,
  
  // 【可选】替换时是否保持视角
  keepView: true
});
```

#### 3. 使用资源 ID 加载（从 DAM 平台）

```javascript
// 如果你使用的是 DAM 平台上的模型，可以直接使用 ID
viewer.loadModel({
  urlOrId: 'qafKij69qraH',  // DAM 平台的资源 ID
  envMapUrl: '/public/environments/studio.hdr',
  onLoad: (model) => {
    console.log('远程模型加载成功');
  }
});
```

#### 4. 模型参数说明

下面是所有参数的详细说明：

#### ModelParams 接口
```typescript
interface ModelParams {
  urlOrId: string;                      // 模型路径或 ID
  file?: File;                         // 文件对象（拖拽上传）
  onLoad?: (model: ModelObject) => void; // 加载完成回调
  onProgress?: (progress: number) => void; // 加载进度回调
  onError?: (error: unknown) => void;   // 加载错误回调
  
  // 变换参数
  position?: number[];                  // 模型位置 [x, y, z]
  rotation?: number[];                  // 模型旋转 [x, y, z] (度)
  scale?: number[];                     // 模型缩放 [x, y, z]
  
  // 渲染参数
  castShadow?: boolean;                // 是否产生阴影
  envMapUrl?: string;                  // 环境贴图路径
  
  // 场景参数
  scene?: SceneObject;                 // 目标场景
  isAutoCenter?: boolean;              // 是否自动居中
  scaleFactor?: number;                // 缩放系数
  bottomShadow?: boolean;              // 是否显示底部阴影
  
  // 替换参数
  replace?: boolean;                   // 是否替换现有模型
  keepView?: boolean;                  // 替换时是否保持视角
  
  modelObject?: ModelObject;           // 指定模型对象
}
```

#### 模型操作
```typescript
// 添加模型到场景
viewer.addModel(model: ModelObject, useModelView?: boolean): void

// 异步加载模型
ResourceManager.loadModel(params: ModelParams): Promise<ModelObject>
```

### 相机控制

#### 什么是相机？
相机就像你的"眼睛"，它决定了你从哪个角度看 3D 模型。相机可以：
- 📍 **移动位置**：前后左右上下移动
- 🎯 **对准目标**：看向某个点
- 🔍 **缩放视野**：拉近拉远

#### 1. 快速视角切换

最简单的方式是使用预设视角：

```javascript
// 切换到前视图（从正面看）
viewer.cameraManager.switchView(Anker3D.CameraView.Front);

// 切换到后视图（从后面看）
viewer.cameraManager.switchView(Anker3D.CameraView.Back);

// 切换到左视图（从左边看）
viewer.cameraManager.switchView(Anker3D.CameraView.Left);

// 切换到右视图（从右边看）
viewer.cameraManager.switchView(Anker3D.CameraView.Right);

// 切换到顶视图（从上方看）
viewer.cameraManager.switchView(Anker3D.CameraView.Top);

// 切换到底视图（从下方看）
viewer.cameraManager.switchView(Anker3D.CameraView.Bottom);
```

**完整示例：创建一个视角切换按钮**

```html
<!DOCTYPE html>
<html>
<head>
  <title>视角切换示例</title>
  <style>
    #viewer-container { width: 100vw; height: 100vh; }
    #controls {
      position: absolute;
      top: 20px;
      left: 20px;
      display: flex;
      gap: 10px;
    }
    button {
      padding: 10px 20px;
      font-size: 14px;
      cursor: pointer;
      border: none;
      background: #007bff;
      color: white;
      border-radius: 5px;
    }
    button:hover { background: #0056b3; }
  </style>
</head>
<body>
  <div id="viewer-container"></div>
  <div id="controls">
    <button onclick="switchToFront()">前视图</button>
    <button onclick="switchToBack()">后视图</button>
    <button onclick="switchToLeft()">左视图</button>
    <button onclick="switchToRight()">右视图</button>
    <button onclick="switchToTop()">顶视图</button>
  </div>

  <script type="module">
    import * as Anker3D from './dist/anker3d.module.js';
    
    const viewer = new Anker3D.Viewer(
      document.getElementById('viewer-container'),
      { dracoPath: '/public/draco/' }
    );
    
    viewer.loadModel({
      urlOrId: '/public/models/product.glb',
      isAutoCenter: true
    });
    
    // 定义视角切换函数
    window.switchToFront = () => viewer.cameraManager.switchView(Anker3D.CameraView.Front);
    window.switchToBack = () => viewer.cameraManager.switchView(Anker3D.CameraView.Back);
    window.switchToLeft = () => viewer.cameraManager.switchView(Anker3D.CameraView.Left);
    window.switchToRight = () => viewer.cameraManager.switchView(Anker3D.CameraView.Right);
    window.switchToTop = () => viewer.cameraManager.switchView(Anker3D.CameraView.Top);
  </script>
</body>
</html>
```

#### 2. 平滑相机动画（flyTo）

flyTo 让相机平滑地飞行到指定位置，而不是瞬间切换：

```javascript
// 平滑飞行到前视图
viewer.cameraManager.flyTo({
  view: {
    position: [0, 0, 5],      // 相机位置：[x, y, z]
    target: [0, 0, 0]         // 相机看的目标点
  },
  duration: 1000,             // 动画持续时间（毫秒）
  easing: Anker3D.Easing.Cubic_Out,  // 缓动函数
  onFinish: () => {
    console.log('相机动画完成！');
  }
});
```

**实际应用示例：切换到近距离观察**

```javascript
// 用户点击"细节查看"按钮
function showDetailView() {
  viewer.cameraManager.flyTo({
    view: {
      position: [2, 1, 2],     // 比较近的位置
      target: [0, 0, 0]        // 仍然看向模型中心
    },
    duration: 1500,
    easing: Anker3D.Easing.Cubic_Out
  });
}

// 返回正常视角
function showNormalView() {
  viewer.cameraManager.flyTo({
    view: {
      position: [0, 1, 5],     // 正常距离
      target: [0, 0, 0]
    },
    duration: 1000,
    easing: Anker3D.Easing.Cubic_Out
  });
}
```

#### 3. 保存和恢复视角

你可以保存当前视角，稍后恢复：

```javascript
// 保存当前视角
const myView = viewer.cameraManager.getView();
console.log('已保存视角:', myView);

// 稍后恢复这个视角
viewer.cameraManager.setView(myView);

// 或者使用 flyTo 平滑地恢复到之前的视角
viewer.cameraManager.flyTo({
  view: myView,
  duration: 1000
});
```

**高级 API：视角管理**

#### 视角管理
```typescript
// 获取当前视角
viewer.cameraManager.getView(): View

// 设置视角
viewer.cameraManager.setView(view: View): void

// 保存视角到模型
viewer.cameraManager.saveViewToModel(model: ModelObject): void
```

#### View 接口
```typescript
interface View {
  position: Vector3 | {x,y,z} | number[];  // 相机位置
  target?: Vector3 | {x,y,z} | number[];   // 相机目标
  polarAngle?: number;                      // 俯仰角
  azimuthalAngle?: number;                  // 水平角
  normal?: Vector3;                         // 相机朝向
  near?: number;                           // 近裁切面
  far?: number;                            // 远裁切面
}
```

#### 相机动画
```typescript
// 飞行到指定视角
viewer.cameraManager.flyTo(params: FlyViewParams): TWEEN.Tween | null

// 沿路径移动
viewer.cameraManager.moveAlongThePath(params: PathMoveParams): void

// 切换到固定视角
viewer.cameraManager.switchView(view: CameraView): void
```

#### FlyViewParams 接口
```typescript
interface FlyViewParams {
  view: View;                          // 目标视角
  motion?: MotionType;                 // 运动方式：Linear | Arc
  duration?: number;                   // 持续时间（毫秒），默认 1000
  easing?: EasingFunction;             // 缓动函数
  delay?: number;                      // 延迟时间，默认 0
  onFinish?: () => void;               // 完成回调
}
```

#### 模型聚焦
```typescript
// 相机聚焦模型
viewer.cameraManager.lookAtModel(object: ModelObject, scaleFactor?: number): void

// 聚焦模型并保持视角
viewer.cameraManager.lookAtModelAndKeepView(object: ModelObject, scaleFactor?: number, view: View): void
```

#### 水平摆动效果
```typescript
// 开始水平摆动（类似 ModelViewer 的操作提示）
viewer.cameraManager.horizontalHunting(angle?: number, iconElement: HTMLElement, iconOffset?: number): TWEEN.Tween

// 停止水平摆动
viewer.cameraManager.stopHorizontalHunting(): void
```

### 场景管理

#### 模型操作
```typescript
// 添加模型
viewer.scene.addModel(model:ModelObject,onAdd:()=>void=()=>{}): void

// 删除模型
viewer.scene.removeModel(model:ModelObject): void

//替换模型
viewer.scene.replaceModel(newModel:ModelObject,targetModel:ModelObject|null = this.activedModel,onReplaced?:()=>void): void

//使用模型id替换模型
viewer.scene.replaceModelById(newModelId:string,targetModelId:string): void
```

#### 环境设置
```typescript
// 加载环境贴图
viewer.scene.loadEnvironmentMap(url: string, asBackground?: boolean): void

// 设置背景
viewer.scene.setBackground(color: string | Color | Texture): void
```

#### 色调映射
```typescript
// 设置色调映射
viewer.renderer.toneMapping = Anker3D.NeutralToneMapping; // 或其他映射类型
viewer.renderer.toneMappingExposure = 1.0; // 曝光值
```

### 后处理效果

后处理效果是在渲染完成后对画面进行二次处理，可以添加各种视觉特效。

#### 1. Bloom 辉光效果

**什么是 Bloom？**
Bloom 让亮的部分变得更亮，产生发光效果。常用于金属材质、玻璃、灯光等。

**基础使用：**

```javascript
// 1. 创建 Bloom 组件
const bloomPass = new Anker3D.BloomPassComponent();

// 2. 添加到查看器
viewer.addComponent(bloomPass);

// 现在整个场景都会有微弱的发光效果
```

**应用到特定对象：**

```javascript
// 加载模型后
viewer.loadModel({
  urlOrId: '/models/product.glb',
  onLoad: (model) => {
    // 找到模型中的某个部件
    const part = model.getObjectByName('screen');  // 假设有个屏幕部件
    
    if (part) {
      // 只对这个部件应用发光效果
      bloomPass.applyToObject(part);
    }
  }
});
```

**调整参数：**

```javascript
const bloomPass = new Anker3D.BloomPassComponent();

// 阈值：亮度超过这个值的部分才会发光（0-1）
bloomPass.threshold = 0.5;  // 默认 0.5，值越大发光范围越小

// 强度：发光有多亮（0-3）
bloomPass.strength = 1.0;   // 默认 1.0，值越大越亮

// 半径：发光的范围（0-1）
bloomPass.radius = 0.5;     // 默认 0.5，值越大发光范围越广

// 添加后才会生效
viewer.addComponent(bloomPass);
```

**完整示例：屏幕发光效果**

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    #viewer-container { width: 100vw; height: 100vh; }
    button {
      position: absolute;
      top: 20px;
      left: 20px;
      padding: 10px;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <div id="viewer-container"></div>
  <button onclick="toggleBloom()">切换发光效果</button>

  <script type="module">
    import * as Anker3D from './dist/anker3d.module.js';
    
    const viewer = new Anker3D.Viewer(
      document.getElementById('viewer-container'),
      { dracoPath: '/public/draco/' }
    );
    
    const bloom = new Anker3D.BloomPassComponent();
    bloom.threshold = 0.6;
    bloom.strength = 1.5;
    
    viewer.loadModel({
      urlOrId: '/models/product.glb',
      onLoad: (model) => {
        // 应用发光到模型的特定部分
        bloom.applyToObject(model);
      }
    });
    
    let bloomEnabled = false;
    window.toggleBloom = () => {
      if (bloomEnabled) {
        viewer.removeComponent(bloom);
        bloomEnabled = false;
      } else {
        viewer.addComponent(bloom);
        bloomEnabled = true;
      }
    };
  </script>
</body>
</html>
```

#### 2. SSAO 环境光遮蔽

**什么是 SSAO？**
SSAO 让物体之间的接触区域变暗，增加立体感和真实感。

```javascript
// 创建 SSAO 组件
const ssaoPass = new Anker3D.SSAOPassComponent();

// 调整参数
ssaoPass.kernelRadius = 8;        // 采样范围，值越大效果越明显（默认 8）
ssaoPass.minDistance = 0.005;     // 最小检测距离（默认 0.005）
ssaoPass.maxDistance = 0.1;       // 最大检测距离（默认 0.1）

// 添加到查看器
viewer.addComponent(ssaoPass);
```

#### 3. SSR 屏幕空间反射

**什么是 SSR？**
SSR 让光滑的表面能反射周围的物体，常用于地面、水面、玻璃等。

```javascript
// 创建 SSR 组件
const ssrPass = new Anker3D.SSRPassComponent();

// 调整参数
ssrPass.resolutionScale = 1;      // 分辨率缩放（0-1），越大效果越好但性能越低
ssrPass.thickness = 0.018;        // 反射厚度
ssrPass.infiniteThick = false;    // 是否无限厚度

// 添加到查看器
viewer.addComponent(ssrPass);
```

#### 4. 移除后处理效果

```javascript
// 移除 Bloom
viewer.removeComponent(bloomPass);

// 完全销毁（释放内存）
viewer.destroyComponent(ssaoPass);
```

**注意事项：**
- 后处理效果会占用额外的性能，移动设备建议谨慎使用
- 多个后处理效果可以同时使用
- 效果参数需要在添加到查看器前设置

#### SSR 屏幕空间反射
```typescript
// 创建 SSR 组件
const ssrPass = new Anker3D.SSRPassComponent();
viewer.addComponent(ssrPass);
```

### 辅助工具

#### 内置工具栏 ToolbarComponent

SDK 内置了一个无需外部 HTML 的工具栏 UI，默认自动添加到 Viewer（`viewer.toolbar`）。它会随屏幕方向自动布局：横屏左侧竖排，竖屏顶部横排；尺寸基于 `vmin` 与百分比自适应。

- 按钮：Angle（六视图）、Measure（尺寸显示）、Capture（截图）、Fullscreen（全屏）、Share（分享）
- 语言：支持 EN/CN/DE，按钮 `aria-label` 跟随语言变化
- 分享：带二维码与复制链接，默认离线本地生成二维码，失败自动回退在线生成

常用 API：

```ts
// 显示/隐藏子按钮（全部隐藏时主面板也隐藏）
viewer.toolbar.showMeasure();
viewer.toolbar.hideCapture();
viewer.toolbar.showFullscreen();
viewer.toolbar.showShare();

// 多语言切换（同步更新 aria-label 与提示文案）
viewer.toolbar.setLanguage('CN'); // 'EN' | 'CN' | 'DE'
```

分享二维码（离线/在线）：

- 默认“离线优先”：在本地动态导入 `qrcode` 生成 dataURL；若失败，自动使用在线服务生成。
- 若你的环境仅允许在线，可在你自行实例化工具栏时传入 `{ qrOffline: false }`。
- 注意：离线模式需要在项目里安装依赖：`npm i qrcode`。

#### 场景辅助器
```typescript
// 显示/隐藏辅助器（坐标轴、网格等）
viewer.sceneHelperManager.show(show: boolean): void
```

#### 性能监控
```typescript
// 显示性能监控面板
viewer.showStats(): void
```

#### 截图功能
```typescript
// 单张截图
viewer.screenShot(params: ScreenShotParams): string

// 环绕截图
viewer.encircleScreenShot(params: ScreenShotParams): string[]
```

#### ScreenShotParams 接口
```typescript
interface ScreenShotParams {
  width?: number;          // 截图宽度
  height?: number;         // 截图高度
  format?: string;         // 图片格式：'png' | 'jpeg'
  quality?: number;        // 图片质量 (0-1)
}
```

#### 交互拾取
```typescript
// 拾取 3D 对象
const result = viewer.picking.pickObject(event: MouseEvent);
if (result) {
  console.log('拾取到对象:', result.object);
  console.log('交点:', result.point);
}
```

---







# -------------分割线------------

## 组件系统

### 组件管理
```typescript
// 添加组件
viewer.addComponent(component: Component): void

// 移除组件
viewer.removeComponent(component: Component): void

// 销毁组件
viewer.destroyComponent(component: Component): void
```

### 自定义组件
```typescript
import { Component } from 'anker3d';

class CustomComponent extends Component {
  onUpdate(delta: number): void {
    // 每帧更新逻辑
  }
  
  onResize(width: number, height: number): void {
    // 尺寸变更处理
  }
  
  destroy(): void {
    // 清理资源
    super.destroy();
  }
}
```

---

## 事件系统

### 事件监听
```typescript
// 监听渲染更新事件
viewer.renderManager.addEventListener('update', (event) => {
  console.log('渲染更新:', event.data);
});

// 监听相机控制器事件
viewer.controls.addEventListener('start', () => {
  console.log('开始交互');
});

viewer.controls.addEventListener('change', () => {
  console.log('相机变化');
});

viewer.controls.addEventListener('end', () => {
  console.log('交互结束');
});
```

---

## 资源管理

### 资源缓存
```typescript
// 设置 Draco 解码器路径
ResourceManager.dracoPath = '/public/draco/';

// 清理所有资源
ResourceManager.destroy(): void
```

### 生命周期管理
```typescript
// 销毁查看器（可选择保留资源）
viewer.destroy(reserveResource?: boolean): void
```

---

## 使用示例

### 基础模型查看器
```typescript
import * as Anker3D from 'anker3d';

const container = document.getElementById('viewer');
const viewer = new Anker3D.Viewer(container, {
  cameraParams: {
    position: [0, 1, 5],
    fov: 50
  }
});

viewer.loadModel({
  urlOrId: '/public/models/S1pro.glb',
  envMapUrl: '/public/environments/S1pro.hdr',
  isAutoCenter: true,
  onLoad: (model) => {
    console.log('模型加载完成:', model);
  }
});
```

### 带后处理效果的查看器
```typescript
const viewer = new Anker3D.Viewer(container);

// 添加辉光效果
const bloom = new Anker3D.BloomPassComponent();
viewer.addComponent(bloom);

viewer.loadModel({
  urlOrId:'/public/models/A2697.glb',
  envMapUrl:"/public/environments/A2697.hdr",
  onLoad: (model) => {
    // 对特定部件应用辉光
    const part = model.getObjectByName('20240830-171542');
    if (part) {
      bloom.applyToObject(part);
    }
  }
});

// 点击切换辉光效果
document.addEventListener('click', (event) => {
  const result = viewer.picking.pickObject(event);
  if (result) {
    bloom.applyToObject(result.object);
  }
});
```

### 相机动画示例
```typescript
// 预设视角
const views = {
  front: { position: [0, 0, 5], target: [0, 0, 0] },
  top: { position: [0, 5, 0], target: [0, 0, 0] },
  side: { position: [5, 0, 0], target: [0, 0, 0] }
};

// 切换到前视图
viewer.cameraManager.flyTo({
  view: views.front,
  duration: 1000,
  easing: Anker3D.Easing.Cubic_Out,
  onFinish: () => console.log('动画完成')
});
```

### 多模型管理
```typescript
const models = [];

// 加载多个模型
const modelPaths = ['/model1.glb', '/model2.glb', '/model3.glb'];

modelPaths.forEach((path, index) => {
  viewer.loadModel({
    urlOrId: path,
    position: [index * 2, 0, 0],
    onLoad: (model) => {
      models.push(model);
      if (models.length === modelPaths.length) {
        console.log('所有模型加载完成');
      }
    }
  });
});
```

```ts
import * as Anker3D from 'your-sdk-path';

const container = document.getElementById('container');
const viewer = new Anker3D.Viewer(container, {
  cameraParams: {
    position: [0, 1, 5],
    fov: 50,
    near: 0.01,
    far: 1000
  }
});

viewer.loadModel({
  urlOrId: '/public/models/S1pro.glb',
  envMapUrl: '/public/environments/S1pro.hdr',
  scene: viewer.sceneManager.defaultScene,
  isAutoCenter: true,
  castShadow: true,
  scaleFactor: 4

// HDR 作为背景或纯色背景
viewer.scene.loadEnvironmentMap('public/environments/S1pro.hdr', true);
viewer.scene.setBackground('#242424');

// 色调映射与曝光
viewer.RenderManager.toneMapping = Anker3D.NeutralToneMapping;
viewer.RenderManager.toneMappingExposure = 1.0;

// 相机视角管理
const v = viewer.cameraManager.getView();
viewer.cameraManager.flyTo({ view: v, easing: Anker3D.Easing.Cubic_Out });

// 销毁
// viewer.destroy();
```

---







# -------------分割线------------

## 常见问题 FAQ

**Q: 为什么模型加载后没有显示？**
A: 请检查模型路径是否正确，Three.js支持的格式（如glb/gltf），以及模型是否包含有效的几何体。

**Q: 为什么模型加载后是全黑色的**
A: 请检查环境贴图（HDR）是否正常加载，因为场景默认是没有开启光照的，模型的光照全来自于环境反射，在缺失环境贴图的情况下模型会呈现黑色状态。

**Q: 环境贴图切换无效？**
A: 请确保HDR路径正确，且贴图为HDR格式。可通过`setEnvironmentMap`的`applyToBackground`参数控制是否作为背景。

**Q: 如何在控制台调试API？**
A: 可将viewer实例挂载到window（如`window.viewer = viewer`），在浏览器控制台直接调用API。

**Q: 销毁viewer后还能继续用吗？**
A: 不可以。`destroy()`会彻底释放所有资源，销毁后请勿再调用任何API。

---

## 高级功能详解

### 渲染任务队列

#### 添加渲染前后任务
```typescript
// 添加渲染前任务
viewer.renderManager.addBeforeRenderingTask((renderer, scene, camera) => {
  console.log('渲染前的操作');
});

// 添加渲染后任务
viewer.renderManager.addAfterRenderingTask((renderer, scene, camera) => {
  console.log('渲染后的操作');
});

// 移除任务
viewer.renderManager.removeBeforeRenderingTask(taskFunction);
viewer.renderManager.removeAfterRenderingTask(taskFunction);
```

### 运动方式说明

#### MotionType 枚举
```typescript
enum MotionType {
  Linear = 0,  // 直线运动（适用于位置变化）
  Arc = 1      // 弧线运动（适用于角度插值）
}
```

#### 缓动函数
```typescript
// 可用的缓动函数
Anker3D.Easing.Linear_None        // 线性
Anker3D.Easing.Quadratic_In       // 二次缓入
Anker3D.Easing.Quadratic_Out      // 二次缓出
Anker3D.Easing.Quadratic_InOut    // 二次缓入缓出
Anker3D.Easing.Cubic_Out         // 三次缓出
Anker3D.Easing.Quartic_InOut     // 四次
Anker3D.Easing.Quintic_Out       // 五次
Anker3D.Easing.Sinusoidal_In     // 正弦
Anker3D.Easing.Exponential_Out   // 指数
Anker3D.Easing.Circular_InOut    // 圆形
Anker3D.Easing.Elastic_Out       // 弹性
Anker3D.Easing.Back_In           // 回退
Anker3D.Easing.Bounce_Out        // 弹跳
```

### 相机路径动画

```typescript
// 定义相机路径
const path = [
  new Anker3D.Vector3(0, 0, 10),
  new Anker3D.Vector3(5, 2, 8),
  new Anker3D.Vector3(0, 0, 5),
];

// 沿路径移动
viewer.cameraManager.moveAlongThePath({
  path: path,
  duration: 3000,
  easing: Anker3D.Easing.Cubic_Out,
  onFinish: () => {
    console.log('路径动画完成');
  }
});
```

### 模型替换功能

```typescript
// 替换模型（自动清理旧模型）
viewer.loadModel({
  urlOrId: '/public/models/newModel.glb',
  replace: true,         // 替换模式
  keepView: true,        // 保持当前视角
  onLoad: (model) => {
    console.log('模型已替换');
  }
});
```

### 多场景管理

```typescript
// 创建新场景
const newScene = viewer.sceneManager.createScene('ecologicalScene');

// 切换到新场景
viewer.sceneManager.switchScene('ecologicalScene');

// 在新场景中加载模型
viewer.loadModel({
  urlOrId: '/models/product.glb',
  scene: newScene
});

// 返回默认场景
viewer.sceneManager.switchScene('defaultScene');
```

### 资源缓存管理

```typescript
// 修正资源管理器路径
ResourceManager.dracoPath = '/public/draco/';

// 启用/禁用缓存
ResourceManager.cacheEnable = true;  // 默认 true

// 从缓存获取模型（不克隆）
const cachedModel = ResourceManager.getModel('/path/to/model.glb', false);

// 从缓存获取模型（克隆）
const clonedModel = ResourceManager.getModel('/path/to/model.glb', true);

// 获取任意资源
const resource = ResourceManager.getResource(url, isClone);

// 清理所有资源
ResourceManager.destroy();
```

## 性能优化建议

### 1. 模型优化

```typescript
// 启用 Draco 压缩
viewer.loadModel({
  urlOrId: 'model.glb',
  // Draco 解码器路径已在初始化时设置
});

// 合理设置自动居中的缩放系数
viewer.loadModel({
  urlOrId: 'model.glb',
  scaleFactor: 1.5,  // 值越大模型显示越小，性能更好
});
```

### 2. 渲染优化

```typescript
// 禁用不必要的后处理
const bloomEnabled = false;  // 关闭 Bloom 提升性能

// 根据设备性能调整后处理参数
const ssaoPass = new Anker3D.SSAOPassComponent();
ssaoPass.kernelRadius = 8;  // 降低内核半径提升性能
```

### 3. 内存管理

```typescript
// 模型加载后及时清理不需要的资源
viewer.loadModel({
  urlOrId: 'model.glb',
  onLoad: (model) => {
    // 保存引用
    viewer.addModel(model);
    
    // 及时清理临时变量
    // ...
  }
});

// 销毁时释放资源
viewer.destroy(false);  // false = 不保留资源
```

### 4. 事件监听优化

```typescript
// 避免频繁触发的事件
let lastTime = 0;
viewer.controls.addEventListener('change', () => {
  const now = Date.now();
  if (now - lastTime > 100) {  // 节流
    console.log('相机变化');
    lastTime = now;
  }
});
```

## 最佳实践

### 1. 初始化最佳实践

```typescript
// ✅ 推荐：提供完整的初始化参数
const viewer = new Anker3D.Viewer(container, {
  cameraParams: {
    position: [0, 1, 5],
    fov: 50,
    near: 0.01,
    far: 1000
  },
  dracoPath: '/public/draco/',
  defaultLight: false
});

// ❌ 不推荐：使用默认参数后手动调整
const viewer = new Anker3D.Viewer(container);
// 后续调整导致的不一致
```

### 2. 模型加载最佳实践

```typescript
// ✅ 推荐：提供错误处理
viewer.loadModel({
  urlOrId: 'model.glb',
  onLoad: (model) => {
    console.log('加载成功');
  },
  onProgress: (progress) => {
    console.log(`加载进度: ${progress}%`);
  },
  onError: (error) => {
    console.error('加载失败:', error);
    // 显示错误提示给用户
  }
});

// ❌ 不推荐：缺少错误处理
viewer.loadModel({ urlOrId: 'model.glb' });
```

### 3. 相机控制最佳实践

```typescript
// ✅ 推荐：使用 flyTo 而非直接设置
viewer.cameraManager.flyTo({
  view: { position: [0, 5, 5], target: [0, 0, 0] },
  duration: 1000,
  easing: Anker3D.Easing.Cubic_Out
});

// ❌ 不推荐：直接修改位置（没有动画）
viewer.camera.position.set(0, 5, 5);
```

### 4. 组件使用最佳实践

```typescript
// ✅ 推荐：组件统一管理
const bloom = new Anker3D.BloomPassComponent();
viewer.addComponent(bloom);

// 使用完毕后移除
viewer.removeComponent(bloom);
viewer.destroyComponent(bloom);

// ❌ 不推荐：组件未清理
const bloom = new Anker3D.BloomPassComponent();
viewer.addComponent(bloom);
// 忘记移除组件
```

## 事件系统详解

### 监听相机事件

```typescript
// 开始交互
viewer.controls.addEventListener('start', () => {
  console.log('开始拖动相机');
  // 可以暂停自动动画
});

// 相机变化
viewer.controls.addEventListener('change', () => {
  // 相机每帧都会触发
  // 注意性能影响
});

// 交互结束
viewer.controls.addEventListener('end', () => {
  console.log('拖动结束');
  // 恢复自动动画
});
```

### 监听渲染事件

```typescript
// 每帧更新
viewer.renderManager.addEventListener('update', (event) => {
  const delta = event.data;  // 时间间隔
  console.log('帧间隔:', delta);
});
```

## 实用工具函数

### 颜色转换

```typescript
import { Color } from 'anker3d';

const color = new Color('#ff0000');
console.log(color.r, color.g, color.b);
```

### 数学工具

```typescript
import { Math } from 'anker3d';

// 角度转弧度
const radians = Math.degToRad(180);
// 弧度转角度
const degrees = Math.radToDeg(Math.PI);
```

## 注意事项

### 1. 资源路径

```typescript
// ✅ 正确
viewer.loadModel({
  urlOrId: '/public/models/S1pro.glb',  // 绝对路径
  envMapUrl: '/public/environments/S1pro.hdr'
});

// ✅ 正确（使用资源ID）
viewer.loadModel({
  urlOrId: 'qafKij69qraH',  // DAM平台资源ID
});

// ❌ 错误
viewer.loadModel({
  urlOrId: '../models/S1pro.glb',  // 相对路径可能失效
});
```

### 2. 环境贴图

```typescript
// 检查环境贴图是否加载成功
viewer.scene.loadEnvironmentMap('path/to/env.hdr', false);
if (!viewer.scene.environment) {
  console.warn('环境贴图加载失败');
}

// 设置环境强度
viewer.scene.environmentIntensity = 1.0;  // 建议范围 0.5 - 2.0
```

### 3. 相机范围

```typescript
// 设置合理的相机范围
viewer.camera.near = 0.01;  // 不能为0或负数
viewer.camera.far = 1000;   // 根据场景大小调整

// 透视相机的视场角
viewer.camera.fov = 50;  // 建议范围 30 - 75
```

## TypeScript 类型定义

### 完整的类型导入

```typescript
import {
  Viewer,
  type ViewerParams,
  type CameraParams,
  type ModelParams,
  ModelObject,
  ResourceManager,
  Easing
} from 'anker3d';
```

## 在线示例

### 基础示例
- [简单示例](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=simple) - 最简单的模型加载示例
- [加载三维平台模型](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=loadModel) - 完整的模型加载示例
- [加载本地模型](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=loadLocalModel) - 加载本地文件示例
- [异步加载](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=asyncLoad) - 使用 async/await 加载模型
- [切换模型](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=changeModel) - 动态切换不同模型

### 视角控制
- [视角切换](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=view) - 预设视角切换示例
- [多视图](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=multipleViewers) - 多个查看器同时显示

### 材质与光照
- [材质切换](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=switchMaterial) - 动态切换模型材质
- [环境贴图](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=environment) - HDR 环境贴图示例

### 后处理效果
- [泛光效果](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=bloom) - Bloom 后处理效果
- [环境光遮蔽](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=ssao) - SSAO 后处理效果

### 动画
- [动画管理](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=animation) - 动画播放、暂停、循环控制示例

### 交互功能
- [拖拽加载](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=drop) - 拖拽文件加载模型
- [尺寸标注](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=dimensions) - 显示模型尺寸信息

### 高级功能
- [SKU切换](https://studio3d-sit.anker-in.com/anker-viewer/examples/index.html?example=sku) - SKU产品切换示例

## 更多示例

### 示例 1: 带加载进度条的查看器

```typescript
const container = document.getElementById('viewer');
const viewer = new Anker3D.Viewer(container);

const progressBar = document.getElementById('progress-bar');
const progressPercent = document.getElementById('progress-percent');

viewer.loadModel({
  urlOrId: '/models/large-model.glb',
  onProgress: (progress) => {
    progressBar.style.width = progress + '%';
    progressPercent.textContent = progress + '%';
  },
  onLoad: () => {
    progressBar.style.display = 'none';
  },
  onError: (error) => {
    console.error(error);
    progressBar.style.display = 'none';
    alert('模型加载失败');
  }
});
```

### 示例 2: 响应式查看器

```typescript
const viewer = new Anker3D.Viewer(container);

// 监听窗口大小变化
window.addEventListener('resize', () => {
  viewer.onResize();
});

// 手动触发尺寸更新
viewer.onResize();
```

### 示例 3: 自定义拾取交互

```typescript
viewer.domElement.addEventListener('click', (event) => {
  const result = viewer.picking.pickObject(event);
  
  if (result) {
    console.log('Selected model:', result.object.name);
    console.log('Pick point:', result.point);
    
    // 高亮选中的对象
    viewer.scene.traverse((obj) => {
      if (obj !== result.object) {
        obj.userData.originalMaterial = obj.material;
        obj.material = outlineMaterial;
      }
    });
  }
});
```

### 示例 4: 动态切换背景

```typescript
const backgrounds = ['#000000', '#ffffff', '#242424'];
let currentBg = 0;

function toggleBackground() {
  currentBg = (currentBg + 1) % backgrounds.length;
  viewer.scene.setBackground(backgrounds[currentBg]);
}

document.getElementById('bg-toggle').addEventListener('click', toggleBackground);
```

---

## 总结

Anker3D SDK 提供了完整的 3D 可视化解决方案，通过阅读本文档，您可以：

1. 了解 SDK 的核心架构和设计理念
2. 掌握基础的模型加载和相机控制
3. 使用后处理效果增强视觉效果
4. 通过最佳实践提升性能和用户体验
5. 根据实际需求扩展自定义功能

如需更详细的参数说明、扩展用法或自定义模块文档，请参考 [API 参考文档](api/index.html) 或联系技术团队！
