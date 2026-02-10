# Hotspot 功能文档

## 简介

Hotspot（热点）功能允许您将 DOM 元素绑定到 3D 场景中的特定位置，实现 2D UI 与 3D 世界的交互。热点会自动跟随相机旋转、处理遮挡检测，并支持丰富的交互功能。

## 核心特性

- **3D 到 2D 坐标转换**：自动将 3D 世界坐标转换为屏幕坐标
- **相机跟随**：热点随相机旋转自动更新位置
- **对象绑定**：热点可以绑定到模型的特定网格，跟随对象移动
- **遮挡检测**：自动检测并隐藏被其他对象遮挡的热点
- **锚点系统**：支持 9 个锚点位置，灵活定位 DOM 元素
- **图层管理**：使用图层批量控制热点的显示/隐藏
- **视角跳转**：点击热点时可以自动切换相机视角
- **数据导出/导入**：支持 JSON 格式的数据序列化

## 快速开始

### 基础使用

```typescript
import { Viewer, HotspotAnchor } from '@anker-studio-3d/anker-viewer';

// 创建 viewer
const viewer = new Viewer(container);

// 添加一个简单的热点
const hotspot = viewer.addHotspot({
    position: [0, 1, 0],  // 3D 世界坐标
    anchor: HotspotAnchor.CENTER
});
```

### 自定义样式

```typescript
// 使用自定义 HTML 元素
viewer.addHotspot({
    position: [1, 0.5, 0],
    element: '<div class="my-hotspot">Product Info</div>',
    anchor: HotspotAnchor.TOP,
    offset: [0, -10]  // 像素偏移
});
```

### 绑定到对象

```typescript
// 热点位置相对于对象，跟随对象移动
viewer.addHotspot({
    position: [0, 0.2, 0],  // 相对于对象的位置
    object: mesh,           // Three.js Mesh 对象
    element: '<div class="tag">Part A</div>',
    anchor: HotspotAnchor.TOP
});
```

### 添加交互

```typescript
viewer.addHotspot({
    position: [0, 1, 0],
    element: '<div class="button">Click Me</div>',
    onClick: (hotspot) => {
        console.log('Hotspot clicked!', hotspot.getUserData());
    },
    view: {
        position: [2, 1, 2],
        target: [0, 1, 0]
    },  // 点击后跳转到的视角
    userData: { id: 'my-hotspot', name: 'Example' }
});
```

## API 参考

### Viewer 方法

#### addHotspot(options: HotspotOptions): Hotspot
添加一个热点到场景中。

**参数：**
- `position`: `[number, number, number]` - 3D 世界坐标（必需）
- `object`: `Object3D` - 绑定的 Three.js 对象（可选）
- `objectId`: `string` - 对象 ID，用于序列化（可选）
- `element`: `HTMLElement | string` - DOM 元素或 HTML 字符串（可选，默认使用内置样式）
- `anchor`: `HotspotAnchor` - 锚点位置（可选，默认为 CENTER）
- `offset`: `[number, number]` - 像素偏移 [x, y]（可选）
- `onClick`: `(hotspot: Hotspot) => void` - 点击回调（可选）
- `view`: `View` - 关联的相机视角（可选）
- `visible`: `boolean` - 是否可见（可选，默认为 true）
- `userData`: `Record<string, any>` - 自定义数据（可选）
- `layer`: `string` - 所属图层 ID（可选，默认为默认图层）

**返回：** `Hotspot` 对象

#### removeHotspot(hotspotId: string): void
移除指定 ID 的热点。

#### getHotspot(hotspotId: string): Hotspot | null
获取指定 ID 的热点对象。

#### getAllHotspots(): Hotspot[]
获取所有热点。

#### createHotspotLayer(name: string): HotspotLayer
创建一个新的热点图层。

#### getHotspotLayer(layerId: string): HotspotLayer | null
获取指定 ID 的图层。

#### exportHotspots(): HotspotData
导出所有热点数据为 JSON 格式。

**返回：**
```typescript
{
    version: string,
    layers: LayerSerializedData[]
}
```

#### importHotspots(data: HotspotData): void
从 JSON 数据导入热点。

#### clearHotspots(): void
清空所有热点。

### HotspotAnchor 枚举

定义热点 DOM 元素相对于 3D 坐标点的位置：

```typescript
enum HotspotAnchor {
    TOP = 'top',              // 顶部中心
    TOP_LEFT = 'top-left',    // 左上角
    TOP_RIGHT = 'top-right',  // 右上角
    CENTER = 'center',        // 中心
    LEFT = 'left',            // 左侧中心
    RIGHT = 'right',          // 右侧中心
    BOTTOM = 'bottom',        // 底部中心
    BOTTOM_LEFT = 'bottom-left',    // 左下角
    BOTTOM_RIGHT = 'bottom-right'   // 右下角
}
```

### Hotspot 类方法

#### setVisible(visible: boolean): void
设置热点可见性。

#### getVisible(): boolean
获取热点可见性。

#### setPosition(position: [number, number, number]): void
设置热点位置。

#### getPosition(): [number, number, number]
获取热点位置。

#### setElement(element: HTMLElement | string): void
设置热点的 DOM 元素。

#### getElement(): HTMLElement
获取热点的 DOM 元素。

#### setUserData(data: Record<string, any>): void
设置自定义数据。

#### getUserData(): Record<string, any>
获取自定义数据。

#### setView(view: View | null): void
设置关联的相机视角。

#### getView(): View | null
获取关联的相机视角。

### HotspotLayer 类方法

#### show(): void
显示图层中的所有热点。

#### hide(): void
隐藏图层中的所有热点。

#### setVisible(visible: boolean): void
设置图层可见性。

#### isVisible(): boolean
获取图层可见性。

#### getHotspots(): Hotspot[]
获取图层中的所有热点。

#### clear(): void
清空图层中的所有热点。

## 高级用法

### 图层管理

```typescript
// 创建不同的图层用于分类管理
const infoLayer = viewer.createHotspotLayer('Information');
const interactiveLayer = viewer.createHotspotLayer('Interactive');

// 添加热点到特定图层
viewer.addHotspot({
    position: [0, 1, 0],
    element: '<div>Info</div>',
    layer: infoLayer.id
});

// 批量控制图层
infoLayer.hide();        // 隐藏所有信息热点
interactiveLayer.show(); // 显示所有交互热点
```

### 数据持久化

```typescript
// 导出热点配置
const data = viewer.exportHotspots();
localStorage.setItem('hotspots', JSON.stringify(data));

// 稍后恢复
const savedData = JSON.parse(localStorage.getItem('hotspots'));
viewer.importHotspots(savedData);
```

### 对象生命周期

```typescript
// 热点会自动跟踪绑定对象的生命周期
const mesh = scene.getObjectByName('part-a');
const hotspot = viewer.addHotspot({
    position: [0, 0, 0],
    object: mesh
});

// 当 mesh 从场景中移除时，热点会自动销毁
scene.remove(mesh);  // hotspot 将自动清理
```

### 视角跳转

```typescript
viewer.addHotspot({
    position: [1, 0.5, 0],
    element: '<div>View Point</div>',
    view: {
        position: [2, 1, 2],
        target: [1, 0.5, 0]
    }  // 点击时使用动画跳转到此视角
});
```

## 样式定制

### 默认样式

SDK 提供了默认的热点样式（白色圆点，24x24px，渐变边缘）：

```css
.anker-hotspot-default {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    background: radial-gradient(circle,
        rgba(255, 255, 255, 1) 0%,
        rgba(255, 255, 255, 0.8) 50%,
        rgba(255, 255, 255, 0) 100%);
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}
```

### 自定义样式

您可以传入任何 HTML 字符串或 DOM 元素：

```typescript
// 方式 1：HTML 字符串
viewer.addHotspot({
    position: [0, 1, 0],
    element: `
        <div style="
            background: #4CAF50;
            color: white;
            padding: 10px;
            border-radius: 8px;
            font-weight: bold;
        ">
            Custom Content
        </div>
    `
});

// 方式 2：创建 DOM 元素
const element = document.createElement('div');
element.className = 'my-hotspot';
element.innerHTML = 'Complex Content';
element.addEventListener('mouseover', () => {
    element.style.transform = 'scale(1.2)';
});

viewer.addHotspot({
    position: [0, 1, 0],
    element: element
});
```

**重要提示：**
- 热点容器的 `pointer-events` 默认为 `none`，要实现交互需要在元素上设置 `pointer-events: auto`
- 热点会自动计算 z-index（基于到相机的距离），无需手动设置

## 性能考虑

1. **可见性优化**：隐藏的热点不会更新位置计算，节省性能
2. **遮挡检测**：使用 Raycaster 进行遮挡检测，会消耗一定性能。对于大量热点，建议按需启用
3. **图层管理**：使用图层批量控制热点可以提高性能
4. **对象生命周期**：系统每秒检查一次对象是否被删除，对性能影响较小

## 注意事项

1. 热点位置在画布外时会自动隐藏
2. 绑定到对象的热点，当对象不可见时热点也会隐藏
3. 热点的 `onClick` 回调在视角跳转之前执行
4. 导出的数据包含 `innerHTML`，导入时会重建 DOM 结构
5. 对象绑定使用 `uuid` 进行引用，确保对象在导入时存在于场景中

## 示例

完整的示例代码请参考：`examples/hotspot.html`

运行示例：
```bash
npm run dev
# 访问 http://localhost:5173/examples/hotspot.html
```

## 类型定义

```typescript
interface HotspotOptions {
    position: [number, number, number];
    object?: Object3D;
    objectId?: string;
    element?: HTMLElement | string;
    anchor?: HotspotAnchor;
    offset?: [number, number];
    onClick?: (hotspot: Hotspot) => void;
    view?: View;
    visible?: boolean;
    userData?: Record<string, any>;
    layer?: string;
}

interface View {
    position: { x: number; y: number; z: number } | number[];
    target?: { x: number; y: number; z: number } | number[];
    polarAngle?: number;
    azimuthalAngle?: number;
    normal?: number[];
    near?: number;
    far?: number;
}
```
