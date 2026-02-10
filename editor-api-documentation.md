# Anker3D 编辑器完整API文档

## 文档信息
- 创建日期：2026-01-22
- 版本：v1.0
- 状态：设计完成

---

## 目录

1. [EditorViewer 统一API](#1-editorviewer-统一api)
2. [ModelEditorComponent API](#2-modeleditorcomponent-api)
3. [SceneEditorComponent API](#3-sceneeditorcomponent-api)
4. [事件系统](#4-事件系统)
5. [类型定义](#5-类型定义)
6. [使用示例](#6-使用示例)

---

## 1. EditorViewer 统一API

### 1.1 构造函数

```typescript
class EditorViewer extends Viewer {
  constructor(container: HTMLElement, params?: ViewerParams);
}
```

**参数**：
- `container`: HTML容器元素
- `params`: 可选的Viewer参数（继承自基类Viewer）

**示例**：
```typescript
const editor = new EditorViewer(document.getElementById('canvas-container'), {
  enableStats: true,
  antialias: true
});
```

---

### 1.2 模式管理

#### 1.2.1 进入模型编辑模式

```typescript
enterModelEditMode(model: ModelObject): void;
```

**说明**：
- 进入模型编辑模式，编辑单个模型的显示效果
- 如果当前处于其他编辑模式，会自动先退出

**参数**：
- `model`: 要编辑的模型对象

**触发事件**：`modeChanged`

**示例**：
```typescript
const model = await viewer.loadModel({ urlOrId: 'M3EF684585' });
editor.enterModelEditMode(model);
```

---

#### 1.2.2 进入场景编辑模式

```typescript
enterSceneEditMode(): void;
```

**说明**：
- 进入场景编辑模式，组合多个模型创建场景
- 如果当前处于其他编辑模式，会自动先退出

**触发事件**：`modeChanged`

**示例**：
```typescript
editor.enterSceneEditMode();
```

---

#### 1.2.3 退出编辑模式

```typescript
exitEditMode(): void;
```

**说明**：
- 退出当前编辑模式，返回IDLE状态
- 自动卸载当前编辑器组件
- 清理所有编辑状态

**触发事件**：`modeChanged`

**示例**：
```typescript
editor.exitEditMode();
```

---

### 1.3 状态查询

#### 1.3.1 获取当前模式

```typescript
get currentMode(): EditorMode;
```

**返回值**：
- `EditorMode.IDLE`: 未进入编辑模式
- `EditorMode.MODEL_EDIT`: 模型编辑模式
- `EditorMode.SCENE_EDIT`: 场景编辑模式

**示例**：
```typescript
console.log(editor.currentMode); // 'model' | 'scene' | 'idle'
```

---

#### 1.3.2 状态检查

```typescript
get isModelEditMode(): boolean;
get isSceneEditMode(): boolean;
get isEditMode(): boolean;
```

**示例**：
```typescript
if (editor.isModelEditMode) {
  // 执行模型编辑特定逻辑
}

if (editor.isSceneEditMode) {
  // 执行场景编辑特定逻辑
}

if (!editor.isEditMode) {
  console.warn('Not in edit mode');
}
```

---

### 1.4 统一加载API

#### 1.4.1 加载模型（统一API）

```typescript
async loadModel(
  modelIdOrUrl: string,
  options?: EditorLoadModelOptions
): Promise<ModelObject>;
```

**说明**：
- **模型编辑模式**：替换当前编辑的模型
- **场景编辑模式**：添加模型实例到场景，支持屏幕坐标定位
- **IDLE模式**：抛出错误

**参数**：
```typescript
interface EditorLoadModelOptions extends ModelLoaderOptions {
  // 场景编辑模式专用参数
  skuId?: string;                      // 指定使用的SKU ID
  screenPosition?: [number, number];   // 屏幕坐标定位
  worldPosition?: [number, number, number]; // 世界坐标定位
  autoSelect?: boolean;                // 是否自动选中（默认true）
}
```

**返回值**：加载的模型对象

**抛出异常**：
- `Error('Cannot load model: not in edit mode')` - 当不在编辑模式时

**示例**：
```typescript
// 模型编辑模式
editor.enterModelEditMode(model);
const newModel = await editor.loadModel('M3EF684586');
// → 替换当前模型

// 场景编辑模式
editor.enterSceneEditMode();
const productA = await editor.loadModel('M3EF684585', {
  skuId: 'sku_black',
  worldPosition: [-2, 0, 0],
  autoSelect: true
});
// → 添加到场景并自动选中

const productB = await editor.loadModel('M3EF684585', {
  skuId: 'sku_white',
  screenPosition: [100, 200]  // 在屏幕坐标(100, 200)处添加
});
// → 添加到场景，使用屏幕坐标计算世界位置
```

---

#### 1.4.2 加载环境贴图（统一API）

```typescript
async loadEnvironment(
  url: string,
  options?: EnvironmentLoadOptions
): Promise<void>;
```

**说明**：
- **模型编辑模式**：设置当前模型的环境贴图
- **场景编辑模式**：设置场景全局环境贴图
- **IDLE模式**：抛出错误

**参数**：
```typescript
interface EnvironmentLoadOptions {
  intensity?: number;                   // 环境贴图强度
  rotation?: [number, number, number];  // 环境贴图旋转（欧拉角）
}
```

**抛出异常**：
- `Error('Cannot load environment: not in edit mode')`

**示例**：
```typescript
// 模型编辑模式
await editor.loadEnvironment('studio.hdr', {
  intensity: 1.5,
  rotation: [0, Math.PI / 2, 0]
});
// → 设置模型的环境，保存到模型配置

// 场景编辑模式
await editor.loadEnvironment('outdoor.hdr', {
  intensity: 2.0
});
// → 设置场景全局环境
```

---

#### 1.4.3 设置环境贴图强度（统一API）

```typescript
setEnvironmentIntensity(intensity: number): void;
```

**参数**：
- `intensity`: 环境贴图强度，范围 0-10，默认 1.0

**抛出异常**：
- `Error('Not in edit mode')`

**示例**：
```typescript
editor.setEnvironmentIntensity(2.5);
```

---

#### 1.4.4 设置环境贴图旋转（统一API）

```typescript
setEnvironmentRotation(x: number, y: number, z: number): void;
```

**参数**：
- `x, y, z`: 欧拉角旋转（弧度）

**抛出异常**：
- `Error('Not in edit mode')`

**示例**：
```typescript
editor.setEnvironmentRotation(0, Math.PI / 4, 0);
```

---

### 1.5 背景设置API

#### 1.5.1 设置背景（统一API）

```typescript
setBackground(background: string | Color | Texture): void;
```

**说明**：
- **模型编辑模式**：设置模型的背景
- **场景编辑模式**：设置场景的背景

**参数**：
- `background`:
  - `string`: 颜色字符串，如 '#242424', 'rgb(255, 0, 0)'
  - `Color`: Three.js Color对象
  - `Texture`: Three.js Texture对象

**抛出异常**：
- `Error('Not in edit mode')`

**示例**：
```typescript
// 使用颜色字符串
editor.setBackground('#242424');

// 使用Three.js Color
editor.setBackground(new THREE.Color(0xff0000));

// 使用纹理
const texture = textureLoader.load('background.jpg');
editor.setBackground(texture);
```

---

#### 1.5.2 使用环境贴图作为背景（统一API）

```typescript
useEnvironmentAsBackground(enable: boolean): void;
```

**参数**：
- `enable`: true表示使用环境贴图作为背景，false表示使用自定义背景

**抛出异常**：
- `Error('Not in edit mode')`

**示例**：
```typescript
await editor.loadEnvironment('studio.hdr');
editor.useEnvironmentAsBackground(true);
// → 背景将显示为环境贴图
```

---

### 1.6 渲染参数API

#### 1.6.1 设置色调映射（统一API）

```typescript
setToneMapping(type: ToneMappingType): void;
```

**参数**：
```typescript
enum ToneMappingType {
  NoToneMapping = 0,
  LinearToneMapping = 1,
  ReinhardToneMapping = 2,
  CineonToneMapping = 3,
  ACESFilmicToneMapping = 4
}
```

**抛出异常**：
- `Error('Not in edit mode')`

**示例**：
```typescript
import { ACESFilmicToneMapping } from 'three';
editor.setToneMapping(ACESFilmicToneMapping);
```

---

#### 1.6.2 设置曝光度（统一API）

```typescript
setToneMappingExposure(exposure: number): void;
```

**参数**：
- `exposure`: 曝光度，范围 0-10，默认 1.0

**抛出异常**：
- `Error('Not in edit mode')`

**示例**：
```typescript
editor.setToneMappingExposure(1.5);
```

---

### 1.7 数据序列化API

#### 1.7.1 序列化当前编辑数据（统一API）

```typescript
serialize(): ModelEditorData | SceneData;
```

**说明**：
- **模型编辑模式**：返回模型配置数据（包含SKU）
- **场景编辑模式**：返回场景JSON数据

**返回值**：
- `ModelEditorData`: 模型编辑配置（见 model-editor-data-structure.md）
- `SceneData`: 场景数据（见 scene-data-structure.md）

**抛出异常**：
- `Error('Not in edit mode')`

**示例**：
```typescript
// 模型编辑模式
const modelData = editor.serialize() as ModelEditorData;
console.log(modelData.skuIds);

// 场景编辑模式
const sceneData = editor.serialize() as SceneData;
console.log(sceneData.objects);
```

---

#### 1.7.2 加载编辑数据（统一API）

```typescript
async load(data: ModelEditorData | SceneData): Promise<void>;
```

**说明**：
- 根据数据类型自动判断并加载
- 自动切换到对应的编辑模式

**参数**：
- `data`: 模型配置数据或场景数据

**判断逻辑**：
- 如果数据包含 `skus` 字段 → 模型配置数据
- 如果数据包含 `objects` 字段 → 场景数据

**抛出异常**：
- `Error('Invalid data format')` - 数据格式不正确

**示例**：
```typescript
// 加载模型配置
const modelData: ModelEditorData = {
  version: '1.0',
  modelId: 'M3EF684585',
  materials: { /* ... */ },
  skuIds: ['sku_001', 'sku_002']
};
await editor.load(modelData);
// → 自动进入模型编辑模式

// 加载场景数据
const sceneData: SceneData = {
  sceneId: 'SCN001',
  sceneName: '产品展示场景',
  version: '1.0',
  objects: [ /* ... */ ]
};
await editor.load(sceneData);
// → 自动进入场景编辑模式
```

---

#### 1.7.3 保存到后端（统一API）

```typescript
async save(options?: SaveOptions): Promise<string>;
```

**说明**：
- 序列化当前数据并保存到后端
- 根据模式选择不同的保存端点

**参数**：
```typescript
interface SaveOptions {
  overwrite?: boolean;  // 是否覆盖已有数据
  comment?: string;     // 保存备注
}
```

**返回值**：保存后的ID

**保存端点**：
- 模型编辑模式 → `/api/models/config`
- 场景编辑模式 → `/api/scenes`

**抛出异常**：
- `Error('Not in edit mode')`
- `Error('Failed to save')` - 网络或服务器错误

**注意**：需要应用层配置后端API地址

**示例**：
```typescript
const savedId = await editor.save({
  overwrite: true,
  comment: '更新产品颜色配置'
});
console.log('Saved with ID:', savedId);
```

---

### 1.8 模型编辑模式专用API

以下API仅在模型编辑模式下有效，其他模式调用会抛出错误。

#### 1.8.1 创建SKU

```typescript
createSKU(name: string, description?: string): SKU;
```

**说明**：基于当前模型状态创建新的SKU

**参数**：
- `name`: SKU名称
- `description`: 可选的描述

**返回值**：创建的SKU对象

**抛出异常**：
- `Error('createSKU is only available in model edit mode')`

**示例**：
```typescript
// 修改材质
const materials = editor.modelEditor.getMaterials();
materials[0].color.set(0x000000);

// 创建黑色SKU
const blackSKU = editor.createSKU('黑色款', '经典黑色配色');
console.log(blackSKU.skuId); // 'sku_xxx'
```

---

#### 1.8.2 切换SKU

```typescript
switchSKU(skuId: string): void;
```

**说明**：切换到指定SKU，应用其材质配置

**参数**：
- `skuId`: 要切换到的SKU ID

**抛出异常**：
- `Error('switchSKU is only available in model edit mode')`
- `Error('SKU not found: xxx')` - SKU不存在

**触发事件**：`skuChanged`

**示例**：
```typescript
editor.switchSKU('sku_black');
// → 应用黑色SKU的材质配置
```

---

#### 1.8.3 获取所有SKU

```typescript
getSKUs(): SKU[];
```

**返回值**：所有SKU列表

**抛出异常**：
- `Error('getSKUs is only available in model edit mode')`

**示例**：
```typescript
const skus = editor.getSKUs();
skus.forEach(sku => {
  console.log(`${sku.name} (${sku.skuId})`);
});
```

---

#### 1.8.4 删除SKU

```typescript
deleteSKU(skuId: string): boolean;
```

**参数**：
- `skuId`: 要删除的SKU ID

**返回值**：
- `true`: 删除成功
- `false`: SKU不存在或无法删除（如默认SKU）

**抛出异常**：
- `Error('deleteSKU is only available in model edit mode')`

**触发事件**：`skuDeleted`

**示例**：
```typescript
const deleted = editor.deleteSKU('sku_old');
if (deleted) {
  console.log('SKU deleted successfully');
}
```

---

#### 1.8.5 保存默认相机视角

```typescript
saveDefaultCameraView(): CameraParams;
```

**说明**：保存当前相机视角为模型的默认视角

**返回值**：保存的相机参数

**抛出异常**：
- `Error('saveDefaultCameraView is only available in model edit mode')`

**示例**：
```typescript
// 调整相机到合适位置
editor.cameraManager.setCameraView(CameraView.FRONT);
editor.cameraManager.zoomToFit();

// 保存为默认视角
const cameraParams = editor.saveDefaultCameraView();
console.log('Saved camera:', cameraParams);
```

---

#### 1.8.6 获取当前编辑的模型

```typescript
getCurrentModel(): ModelObject | null;
```

**返回值**：
- `ModelObject`: 当前编辑的模型
- `null`: 不在模型编辑模式或未加载模型

**示例**：
```typescript
const model = editor.getCurrentModel();
if (model) {
  console.log('Editing model:', model.id);
}
```

---

### 1.9 场景编辑模式专用API

以下API仅在场景编辑模式下有效，其他模式调用会抛出错误。

#### 1.9.1 移除对象

```typescript
removeObject(objectId: string): void;
```

**参数**：
- `objectId`: 要移除的对象ID（Object3D.uuid）

**抛出异常**：
- `Error('removeObject is only available in scene edit mode')`
- `Error('Object not found: xxx')` - 对象不存在

**触发事件**：`objectRemoved`

**示例**：
```typescript
const object = await editor.loadModel('M3EF684585');
// ... 使用对象 ...
editor.removeObject(object.uuid);
// → 从场景中移除
```

---

#### 1.9.2 创建组合

```typescript
createGroup(name: string): Group;
```

**说明**：创建空的Group节点用于组织层级

**参数**：
- `name`: 组合名称

**返回值**：创建的Group对象

**抛出异常**：
- `Error('createGroup is only available in scene edit mode')`

**触发事件**：`objectAdded`

**示例**：
```typescript
const productGroup = editor.createGroup('产品组合');
editor.reparent(productA.uuid, productGroup.uuid);
editor.reparent(productB.uuid, productGroup.uuid);
```

---

#### 1.9.3 重新组织层级

```typescript
reparent(childId: string, newParentId: string | null): void;
```

**说明**：将对象移动到新的父节点下

**参数**：
- `childId`: 子对象ID
- `newParentId`: 新父对象ID，`null`表示移到根级别

**抛出异常**：
- `Error('reparent is only available in scene edit mode')`
- `Error('Child object not found')` - 子对象不存在
- `Error('Parent object not found')` - 父对象不存在
- `Error('Cannot reparent: would create cycle')` - 会造成循环依赖

**触发事件**：`hierarchyChanged`

**示例**：
```typescript
// 将productA移到productGroup下
editor.reparent(productA.uuid, productGroup.uuid);

// 将productA移到根级别
editor.reparent(productA.uuid, null);
```

---

#### 1.9.4 获取场景树

```typescript
getSceneTree(): SceneNode[];
```

**说明**：获取场景层级结构的扁平化表示

**返回值**：
```typescript
interface SceneNode {
  id: string;
  type: 'object' | 'group';
  name: string;
  parentId: string | null;
  children: string[];
  visible: boolean;
  expanded?: boolean;  // UI展示用
}
```

**抛出异常**：
- `Error('getSceneTree is only available in scene edit mode')`

**示例**：
```typescript
const sceneTree = editor.getSceneTree();
sceneTree.forEach(node => {
  console.log(`${node.name} (${node.type})`);
  console.log(`  Children: ${node.children.join(', ')}`);
});
```

---

#### 1.9.5 选中对象

```typescript
selectObject(object: Object3D): void;
```

**说明**：选中场景中的对象，显示变换控制器

**参数**：
- `object`: 要选中的对象

**抛出异常**：
- `Error('selectObject is only available in scene edit mode')`

**触发事件**：`objectSelected`

**示例**：
```typescript
const object = await editor.loadModel('M3EF684585', { autoSelect: false });
// ... 延迟选中
editor.selectObject(object);
```

---

#### 1.9.6 清空选择

```typescript
clearSelection(): void;
```

**说明**：取消当前对象的选中状态，隐藏变换控制器

**抛出异常**：
- `Error('clearSelection is only available in scene edit mode')`

**触发事件**：`objectDeselected`

**示例**：
```typescript
editor.clearSelection();
```

---

#### 1.9.7 获取选中对象

```typescript
getSelectedObject(): Object3D | null;
```

**返回值**：
- `Object3D`: 当前选中的对象
- `null`: 无选中对象

**抛出异常**：
- `Error('getSelectedObject is only available in scene edit mode')`

**示例**：
```typescript
const selected = editor.getSelectedObject();
if (selected) {
  console.log('Selected:', selected.name);
}
```

---

#### 1.9.8 设置变换模式

```typescript
setTransformMode(mode: 'translate' | 'rotate' | 'scale'): void;
```

**说明**：设置变换控制器的模式

**参数**：
- `mode`: 变换模式
  - `'translate'`: 移动
  - `'rotate'`: 旋转
  - `'scale'`: 缩放

**抛出异常**：
- `Error('setTransformMode is only available in scene edit mode')`

**示例**：
```typescript
editor.setTransformMode('translate');  // 切换到移动模式
editor.setTransformMode('rotate');     // 切换到旋转模式
editor.setTransformMode('scale');      // 切换到缩放模式
```

---

#### 1.9.9 设置变换空间

```typescript
setTransformSpace(space: 'local' | 'world'): void;
```

**说明**：设置变换控制器的坐标空间

**参数**：
- `space`: 坐标空间
  - `'local'`: 本地坐标系
  - `'world'`: 世界坐标系

**抛出异常**：
- `Error('setTransformSpace is only available in scene edit mode')`

**示例**：
```typescript
editor.setTransformSpace('local');   // 本地坐标系（相对于对象自身）
editor.setTransformSpace('world');   // 世界坐标系（绝对坐标）
```

---

### 1.10 相机视角管理API（统一API）

> **重要说明**：相机视角管理API在模型编辑模式和场景编辑模式下都可用。

#### 1.10.1 保存相机视角

```typescript
saveCameraView(name: string, description?: string): CameraView;
```

**说明**：保存当前相机视角为命名视角

**参数**：
- `name`: 视角名称
- `description`: 视角描述（可选）

**返回值**：创建的相机视角对象

**抛出异常**：
- `Error('Not in edit mode')` - 未进入编辑模式

**行为差异**：
- **模型编辑模式**：保存到模型的 `cameraViews` 列表
- **场景编辑模式**：保存到场景的 `cameraViews` 列表

**示例**：
```typescript
// 调整相机到合适位置
editor.cameraManager.setCameraView(CameraView.FRONT);
editor.cameraManager.zoomToFit();

// 保存为命名视角
const frontView = editor.saveCameraView('正面视角', '展示产品正面的最佳角度');
console.log('Created view:', frontView.id);
```

---

#### 1.10.2 删除相机视角

```typescript
deleteCameraView(viewId: string): void;
```

**说明**：删除指定的相机视角

**参数**：
- `viewId`: 视角ID

**抛出异常**：
- `Error('Not in edit mode')` - 未进入编辑模式
- `Error('Camera view not found')` - 视角不存在

**示例**：
```typescript
editor.deleteCameraView('view_123');
```

---

#### 1.10.3 获取所有相机视角

```typescript
getCameraViews(): CameraView[];
```

**说明**：获取当前编辑对象的所有相机视角列表

**返回值**：相机视角数组

**抛出异常**：
- `Error('Not in edit mode')` - 未进入编辑模式

**示例**：
```typescript
const views = editor.getCameraViews();
views.forEach(view => {
  console.log(`${view.name}: ${view.description || '无描述'}`);
});
```

---

#### 1.10.4 切换相机视角

```typescript
switchCameraView(viewId: string): void;
```

**说明**：切换到指定的相机视角

**参数**：
- `viewId`: 视角ID

**抛出异常**：
- `Error('Not in edit mode')` - 未进入编辑模式
- `Error('Camera view not found')` - 视角不存在

**示例**：
```typescript
// 获取所有视角
const views = editor.getCameraViews();

// 切换到第一个视角
if (views.length > 0) {
  editor.switchCameraView(views[0].id);
}
```

---

#### 1.10.5 更新相机视角

```typescript
updateCameraView(viewId: string, updates: Partial<CameraView>): void;
```

**说明**：更新现有相机视角的信息

**参数**：
- `viewId`: 视角ID
- `updates`: 要更新的字段

**抛出异常**：
- `Error('Not in edit mode')` - 未进入编辑模式
- `Error('Camera view not found')` - 视角不存在

**示例**：
```typescript
// 更新视角名称和描述
editor.updateCameraView('view_123', {
  name: '新的视角名称',
  description: '更新后的描述'
});

// 更新视角的相机参数（保存当前相机位置）
const camera = editor.cameraManager.getCamera();
const controls = editor.cameraManager.controls;
editor.updateCameraView('view_123', {
  camera: {
    position: camera.position.toArray() as [number, number, number],
    target: controls.target.toArray() as [number, number, number],
    zoom: camera.zoom,
    fov: (camera as THREE.PerspectiveCamera).fov
  }
});
```

---

#### 1.10.6 生成视角缩略图

```typescript
generateCameraViewThumbnail(viewId: string, width?: number, height?: number): Promise<string>;
```

**说明**：为指定视角生成缩略图

**参数**：
- `viewId`: 视角ID
- `width`: 缩略图宽度（默认256）
- `height`: 缩略图高度（默认144）

**返回值**：Promise<Base64编码的图片数据>

**抛出异常**：
- `Error('Not in edit mode')` - 未进入编辑模式
- `Error('Camera view not found')` - 视角不存在

**示例**：
```typescript
// 生成缩略图并更新视角
const thumbnail = await editor.generateCameraViewThumbnail('view_123', 512, 288);
editor.updateCameraView('view_123', { thumbnail });
```

---

## 2. ModelEditorComponent API

ModelEditorComponent由EditorViewer内部管理，通过`editor.modelEditor`访问。

### 2.1 材质管理

#### 2.1.1 获取所有材质

```typescript
getMaterials(): Record<string, Material>;
```

**返回值**：材质映射表，key为材质路径

**示例**：
```typescript
const materials = editor.modelEditor.getMaterials();
Object.entries(materials).forEach(([path, material]) => {
  console.log(`${path}:`, material.type);
});
```

---

#### 2.1.2 更新材质属性

```typescript
updateMaterial(
  materialPath: string,
  properties: Partial<MaterialProperties>
): void;
```

**参数**：
- `materialPath`: 材质路径，如 "body/base"
- `properties`: 要更新的属性（见 sku-data-structure.md）

**触发事件**：`materialChanged`

**示例**：
```typescript
editor.modelEditor.updateMaterial('body/base', {
  color: '#ff0000',
  metalness: 0.8,
  roughness: 0.2
});
```

---

#### 2.1.3 批量更新材质

```typescript
batchUpdateMaterials(
  updates: Record<string, Partial<MaterialProperties>>
): void;
```

**参数**：
- `updates`: 材质路径到属性的映射

**触发事件**：`materialChanged`（每个材质触发一次）

**示例**：
```typescript
editor.modelEditor.batchUpdateMaterials({
  'body/base': { color: '#ff0000' },
  'body/accent': { color: '#000000' },
  'logo': { emissive: '#ffffff', emissiveIntensity: 0.5 }
});
```

---

### 2.2 SKU管理

#### 2.2.1 获取SKUManager

```typescript
get skuManager(): SKUManager;
```

**返回值**：SKUManager实例

**示例**：
```typescript
const skuManager = editor.modelEditor.skuManager;
const allSKUs = skuManager.getAllSKUs();
```

---

#### 2.2.2 SKUManager API

```typescript
class SKUManager {
  // 创建SKU
  createSKU(params: CreateSKUParams): SKU;

  // 获取SKU
  getSKU(skuId: string): SKU | null;
  getAllSKUs(): SKU[];
  getDefaultSKU(): SKU | null;

  // 切换SKU（使用Diff算法优化）
  switchTo(skuId: string): void;

  // 删除SKU
  deleteSKU(skuId: string): boolean;

  // 设置默认SKU
  setDefaultSKU(skuId: string): void;

  // 当前SKU
  get currentSKU(): SKU | null;
}
```

**详细定义见 sku-data-structure.md**

---

### 2.3 环境与背景

#### 2.3.1 加载环境贴图

```typescript
async loadEnvironment(url: string): Promise<void>;
```

**参数**：
- `url`: 环境贴图URL

**示例**：
```typescript
await editor.modelEditor.loadEnvironment('studio.hdr');
```

---

#### 2.3.2 设置环境强度

```typescript
setEnvironmentIntensity(intensity: number): void;
```

**参数**：
- `intensity`: 强度值，范围 0-10

---

#### 2.3.3 设置环境旋转

```typescript
setEnvironmentRotation(x: number, y: number, z: number): void;
```

**参数**：
- `x, y, z`: 欧拉角旋转（弧度）

---

#### 2.3.4 设置背景

```typescript
setBackground(background: string | Color | Texture): void;
useEnvironmentAsBackground(enable: boolean): void;
```

---

### 2.4 渲染参数

```typescript
setToneMapping(type: ToneMappingType): void;
setToneMappingExposure(exposure: number): void;
```

---

### 2.5 相机视角

#### 2.5.1 保存默认视角

```typescript
saveDefaultCameraView(): CameraParams;
```

**返回值**：保存的相机参数

**示例**：
```typescript
const cameraParams = editor.modelEditor.saveDefaultCameraView();
console.log(cameraParams.position, cameraParams.target);
```

---

### 2.6 数据序列化

#### 2.6.1 序列化

```typescript
serialize(): ModelEditorData;
```

**返回值**：模型编辑配置数据（见 model-editor-data-structure.md）

**示例**：
```typescript
const data = editor.modelEditor.serialize();
localStorage.setItem('modelConfig', JSON.stringify(data));
```

---

#### 2.6.2 加载配置

```typescript
async load(data: ModelEditorData): Promise<void>;
```

**参数**：
- `data`: 模型编辑配置数据

**示例**：
```typescript
const data = JSON.parse(localStorage.getItem('modelConfig'));
await editor.modelEditor.load(data);
```

---

## 3. SceneEditorComponent API

SceneEditorComponent由EditorViewer内部管理，通过`editor.sceneEditor`访问。

### 3.1 对象管理

#### 3.1.1 添加对象

```typescript
async addObject(
  modelId: string,
  options?: AddObjectOptions
): Promise<Object3D>;
```

**参数**：
```typescript
interface AddObjectOptions {
  skuId?: string;
  transform?: TransformData;
  name?: string;
  visible?: boolean;
  useGlobalSettings?: boolean;
  localSettings?: {
    environment?: EnvironmentConfig;
    background?: BackgroundConfig;
    rendering?: RenderingConfig;
  };
}
```

**返回值**：添加的对象

**触发事件**：`objectAdded`

**示例**：
```typescript
const obj = await editor.sceneEditor.addObject('M3EF684585', {
  skuId: 'sku_black',
  transform: {
    position: [0, 0, 0],
    rotation: [0, 0, 0],
    scale: [1, 1, 1]
  },
  name: '产品A'
});
```

---

#### 3.1.2 移除对象

```typescript
removeObject(objectId: string): void;
```

**参数**：
- `objectId`: 对象ID（Object3D.uuid）

**触发事件**：`objectRemoved`

---

#### 3.1.3 获取对象

```typescript
getObject(objectId: string): Object3D | null;
getAllObjects(): Object3D[];
```

**示例**：
```typescript
const obj = editor.sceneEditor.getObject('uuid_xxx');
const allObjects = editor.sceneEditor.getAllObjects();
```

---

### 3.2 选择管理

```typescript
class SelectionManager {
  // 选择对象
  select(object: Object3D): void;

  // 取消选择
  deselect(): void;

  // 获取选中对象
  getSelectedObject(): Object3D | null;

  // 检查是否选中
  isSelected(object: Object3D): boolean;
}
```

**访问方式**：
```typescript
editor.sceneEditor.selectionManager.select(object);
```

---

### 3.3 变换控制

```typescript
class TransformController {
  // 设置变换模式
  setMode(mode: 'translate' | 'rotate' | 'scale'): void;

  // 设置坐标空间
  setSpace(space: 'local' | 'world'): void;

  // 启用/禁用
  setEnabled(enabled: boolean): void;

  // 获取当前模式
  getMode(): 'translate' | 'rotate' | 'scale';
  getSpace(): 'local' | 'world';
}
```

**访问方式**：
```typescript
editor.sceneEditor.transformController.setMode('rotate');
```

---

### 3.4 层级管理

#### 3.4.1 创建组合

```typescript
createGroup(name: string): Group;
```

**触发事件**：`objectAdded`

---

#### 3.4.2 重新组织层级

```typescript
reparent(childId: string, newParentId: string | null): void;
```

**触发事件**：`hierarchyChanged`

---

#### 3.4.3 获取层级管理器

```typescript
get hierarchyManager(): HierarchyManager;
```

```typescript
class HierarchyManager {
  // 获取场景树
  getSceneTree(): SceneNode[];

  // 获取节点的子节点
  getChildren(nodeId: string): SceneNode[];

  // 获取节点的父节点
  getParent(nodeId: string): SceneNode | null;

  // 获取节点路径
  getPath(nodeId: string): string[];

  // 验证是否会造成循环
  wouldCreateCycle(childId: string, parentId: string): boolean;
}
```

---

### 3.5 全局设置

#### 3.5.1 设置全局环境

```typescript
async loadEnvironment(url: string): Promise<void>;
setEnvironmentIntensity(intensity: number): void;
setEnvironmentRotation(x: number, y: number, z: number): void;
```

---

#### 3.5.2 设置全局背景

```typescript
setBackground(background: string | Color | Texture): void;
useEnvironmentAsBackground(enable: boolean): void;
```

---

#### 3.5.3 设置全局渲染参数

```typescript
setToneMapping(type: ToneMappingType): void;
setToneMappingExposure(exposure: number): void;
```

---

### 3.6 数据序列化

#### 3.6.1 序列化

```typescript
serialize(): SceneData;
```

**返回值**：场景数据（见 scene-data-structure.md）

**示例**：
```typescript
const sceneData = editor.sceneEditor.serialize();
```

---

#### 3.6.2 加载场景

```typescript
async load(data: SceneData): Promise<void>;
```

**参数**：
- `data`: 场景数据

**示例**：
```typescript
await editor.sceneEditor.load(sceneData);
```

---

## 4. 事件系统

### 4.1 EditorViewer事件

#### 4.1.1 模式变化事件

```typescript
interface EditorModeChangedEvent {
  type: 'modeChanged';
  oldMode: EditorMode;
  newMode: EditorMode;
  timestamp: number;
}
```

**监听示例**：
```typescript
editor.addEventListener('modeChanged', (e: EditorModeChangedEvent) => {
  console.log(`Mode changed: ${e.oldMode} → ${e.newMode}`);

  // 更新UI状态
  if (e.newMode === EditorMode.MODEL_EDIT) {
    showModelEditToolbar();
  } else if (e.newMode === EditorMode.SCENE_EDIT) {
    showSceneEditToolbar();
  }
});
```

---

#### 4.1.2 性能警告事件

```typescript
interface PerformanceWarningEvent {
  type: 'performanceWarning';
  message: string;
  currentFPS: number;
  suggestedAction: 'reduceRenderQuality' | 'removeObjects';
}
```

**监听示例**：
```typescript
editor.addEventListener('performanceWarning', (e: PerformanceWarningEvent) => {
  showNotification({
    type: 'warning',
    message: e.message,
    action: {
      label: '降低渲染质量',
      handler: () => editor.reduceRenderQuality()
    }
  });
});
```

---

### 4.2 ModelEditor事件

#### 4.2.1 SKU变化事件

```typescript
interface SKUChangedEvent {
  type: 'skuChanged';
  oldSkuId: string | null;
  newSkuId: string;
  sku: SKU;
}
```

**监听示例**：
```typescript
editor.addEventListener('skuChanged', (e: SKUChangedEvent) => {
  console.log(`SKU changed: ${e.oldSkuId} → ${e.newSkuId}`);
  updateSKUSelector(e.newSkuId);
});
```

---

#### 4.2.2 SKU创建事件

```typescript
interface SKUCreatedEvent {
  type: 'skuCreated';
  sku: SKU;
}
```

---

#### 4.2.3 SKU删除事件

```typescript
interface SKUDeletedEvent {
  type: 'skuDeleted';
  skuId: string;
}
```

---

#### 4.2.4 材质变化事件

```typescript
interface MaterialChangedEvent {
  type: 'materialChanged';
  materialPath: string;
  material: Material;
  changedProperties: string[];
}
```

**监听示例**：
```typescript
editor.addEventListener('materialChanged', (e: MaterialChangedEvent) => {
  console.log(`Material ${e.materialPath} changed:`, e.changedProperties);
});
```

---

### 4.3 SceneEditor事件

#### 4.3.1 对象添加事件

```typescript
interface ObjectAddedEvent {
  type: 'objectAdded';
  object: Object3D;
  objectId: string;
}
```

---

#### 4.3.2 对象移除事件

```typescript
interface ObjectRemovedEvent {
  type: 'objectRemoved';
  objectId: string;
}
```

---

#### 4.3.3 对象选中事件

```typescript
interface ObjectSelectedEvent {
  type: 'objectSelected';
  object: Object3D;
  objectId: string;
}
```

---

#### 4.3.4 对象取消选中事件

```typescript
interface ObjectDeselectedEvent {
  type: 'objectDeselected';
  objectId: string | null;
}
```

---

#### 4.3.5 对象变换事件

```typescript
interface ObjectTransformedEvent {
  type: 'objectTransformed';
  object: Object3D;
  objectId: string;
  transform: TransformData;
}
```

**监听示例**：
```typescript
editor.addEventListener('objectTransformed', (e: ObjectTransformedEvent) => {
  console.log('Object transformed:', e.objectId);
  console.log('Position:', e.transform.position);
  updatePropertyPanel(e.transform);
});
```

---

#### 4.3.6 层级变化事件

```typescript
interface HierarchyChangedEvent {
  type: 'hierarchyChanged';
  childId: string;
  oldParentId: string | null;
  newParentId: string | null;
}
```

**监听示例**：
```typescript
editor.addEventListener('hierarchyChanged', (e: HierarchyChangedEvent) => {
  console.log(`${e.childId}: ${e.oldParentId} → ${e.newParentId}`);
  refreshHierarchyTree();
});
```

---

### 4.4 事件监听器管理

```typescript
// 添加事件监听器
editor.addEventListener(type: string, listener: (event: any) => void): void;

// 移除事件监听器
editor.removeEventListener(type: string, listener: (event: any) => void): void;

// 触发一次性事件监听器
editor.once(type: string, listener: (event: any) => void): void;
```

**示例**：
```typescript
const handler = (e: ObjectAddedEvent) => {
  console.log('Object added:', e.objectId);
};

// 添加监听器
editor.addEventListener('objectAdded', handler);

// 移除监听器
editor.removeEventListener('objectAdded', handler);

// 一次性监听器
editor.once('modeChanged', (e) => {
  console.log('Mode changed for the first time');
});
```

---

## 5. 类型定义

### 5.1 编辑器模式

```typescript
export enum EditorMode {
  IDLE = 'idle',
  MODEL_EDIT = 'model',
  SCENE_EDIT = 'scene'
}
```

---

### 5.2 变换数据

```typescript
export interface TransformData {
  position: [number, number, number];
  rotation: [number, number, number];  // 欧拉角（弧度）
  scale: [number, number, number];
}
```

---

### 5.3 相机参数

```typescript
export interface CameraParams {
  position: [number, number, number];
  target: [number, number, number];
  zoom?: number;
  fov?: number;
}
```

---

### 5.4 材质属性

详见 `sku-data-structure.md` 中的 `MaterialProperties` 定义

---

### 5.5 环境配置

```typescript
export interface EnvironmentConfig {
  url: string;
  intensity: number;
  rotation: [number, number, number];
}
```

---

### 5.6 背景配置

```typescript
export interface BackgroundConfig {
  type: 'color' | 'texture' | 'environment';
  value?: string;  // 颜色值或纹理URL
}
```

---

### 5.7 渲染配置

```typescript
export interface RenderingConfig {
  toneMapping: ToneMappingType;
  toneMappingExposure: number;
  shadowEnabled?: boolean;
  shadowIntensity?: number;
}
```

---

### 5.8 场景节点

```typescript
export interface SceneNode {
  id: string;
  type: 'object' | 'group';
  name: string;
  parentId: string | null;
  children: string[];
  visible: boolean;
  expanded?: boolean;
}
```

---

## 6. 使用示例

### 6.1 模型编辑完整流程

```typescript
import { EditorViewer, EditorMode } from 'anker3d';

// 1. 创建编辑器
const editor = new EditorViewer(container);

// 2. 监听事件
editor.addEventListener('modeChanged', (e) => {
  console.log(`Mode: ${e.newMode}`);
});

editor.addEventListener('skuChanged', (e) => {
  console.log(`SKU: ${e.newSkuId}`);
});

// 3. 加载模型
const model = await viewer.loadModel({ urlOrId: 'M3EF684585' });

// 4. 进入模型编辑模式
editor.enterModelEditMode(model);

// 5. 设置环境
await editor.loadEnvironment('studio.hdr', {
  intensity: 1.5,
  rotation: [0, Math.PI / 4, 0]
});

// 6. 设置背景
editor.setBackground('#242424');

// 7. 修改材质
const materials = editor.modelEditor.getMaterials();
editor.modelEditor.updateMaterial('body/base', {
  color: '#000000',
  metalness: 0.9,
  roughness: 0.1
});

// 8. 保存为SKU
const blackSKU = editor.createSKU('黑色款', '经典黑色配色');

// 9. 修改材质（白色）
editor.modelEditor.updateMaterial('body/base', {
  color: '#ffffff'
});

// 10. 保存另一个SKU
const whiteSKU = editor.createSKU('白色款');

// 11. 切换SKU预览
editor.switchSKU(blackSKU.skuId);
await new Promise(resolve => setTimeout(resolve, 2000));
editor.switchSKU(whiteSKU.skuId);

// 12. 保存默认相机视角
editor.cameraManager.setCameraView(CameraView.FRONT);
editor.saveDefaultCameraView();

// 13. 序列化并保存
const modelData = editor.serialize();
const savedId = await editor.save({
  comment: '添加黑白两个SKU配色'
});

console.log('Model config saved with ID:', savedId);

// 14. 退出编辑模式
editor.exitEditMode();
```

---

### 6.2 场景编辑完整流程

```typescript
// 1. 创建编辑器（复用上面的editor实例）
// const editor = new EditorViewer(container);

// 2. 进入场景编辑模式
editor.enterSceneEditMode();

// 3. 设置全局环境
await editor.loadEnvironment('outdoor.hdr', {
  intensity: 2.0
});
editor.setBackground('#87CEEB');  // 天蓝色背景

// 4. 添加第一个产品（黑色款）
const productA = await editor.loadModel('M3EF684585', {
  skuId: 'sku_black',
  worldPosition: [-2, 0, 0],
  autoSelect: true
});
productA.name = '产品A';

// 5. 添加第二个产品（白色款）
const productB = await editor.loadModel('M3EF684585', {
  skuId: 'sku_white',
  worldPosition: [2, 0, 0],
  autoSelect: false
});
productB.name = '产品B';

// 6. 添加第三个产品（使用屏幕坐标）
const productC = await editor.loadModel('M3EF684585', {
  skuId: 'sku_black',
  screenPosition: [100, 200],  // 屏幕坐标
  autoSelect: false
});
productC.name = '产品C';

// 7. 创建组合
const productGroup = editor.createGroup('产品组合');

// 8. 组织层级
editor.reparent(productA.uuid, productGroup.uuid);
editor.reparent(productB.uuid, productGroup.uuid);

// 9. 查看场景树
const sceneTree = editor.getSceneTree();
console.log('Scene hierarchy:');
sceneTree.forEach(node => {
  const indent = '  '.repeat(node.parentId ? 1 : 0);
  console.log(`${indent}- ${node.name} (${node.type})`);
});

// 10. 选择并变换对象
editor.selectObject(productA);
editor.setTransformMode('rotate');
editor.setTransformSpace('local');

// 监听变换事件
editor.addEventListener('objectTransformed', (e) => {
  console.log('Object transformed:', e.objectId);
  console.log('New position:', e.transform.position);
});

// 11. 序列化场景
const sceneData = editor.serialize();
console.log('Scene data:', sceneData);

// 12. 保存场景
const sceneId = await editor.save({
  comment: '产品展示场景，包含3个产品和1个组合'
});

console.log('Scene saved with ID:', sceneId);

// 13. 退出编辑模式
editor.exitEditMode();
```

---

### 6.3 模式切换

```typescript
// 从模型编辑切换到场景编辑
editor.enterModelEditMode(model);
// ... 编辑模型 ...
editor.enterSceneEditMode();  // 自动退出模型编辑模式
// ... 编辑场景 ...

// 返回IDLE状态
editor.exitEditMode();

// 检查当前模式
switch (editor.currentMode) {
  case EditorMode.IDLE:
    console.log('Not in edit mode');
    break;
  case EditorMode.MODEL_EDIT:
    console.log('Model editing mode');
    break;
  case EditorMode.SCENE_EDIT:
    console.log('Scene editing mode');
    break;
}
```

---

### 6.4 数据加载

```typescript
// 加载模型配置
const modelData: ModelEditorData = await fetch('/api/models/M3EF684585/config')
  .then(res => res.json());

await editor.load(modelData);
// → 自动进入模型编辑模式
// → 应用所有配置（材质、环境、SKU等）

// 加载场景数据
const sceneData: SceneData = await fetch('/api/scenes/SCN001')
  .then(res => res.json());

await editor.load(sceneData);
// → 自动进入场景编辑模式
// → 加载所有引用的模型
// → 重建场景层级结构
```

---

### 6.5 性能监控

```typescript
// 启用性能监控
editor.enablePerformanceMonitoring();

// 监听性能警告
editor.addEventListener('performanceWarning', (e: PerformanceWarningEvent) => {
  console.warn(`Low FPS detected: ${e.currentFPS}`);

  // 显示通知给用户
  showNotification({
    type: 'warning',
    message: '场景较复杂，可以通过降低渲染质量来提高流畅度',
    actions: [
      {
        label: '降低质量',
        handler: () => {
          editor.reduceRenderQuality();
          hideNotification();
        }
      },
      {
        label: '忽略',
        handler: () => hideNotification()
      }
    ]
  });
});

// 用户手动触发降低质量
function onReduceQualityClick() {
  editor.reduceRenderQuality();
  // → 降低像素比、禁用抗锯齿等
}
```

---

### 6.6 错误处理

```typescript
try {
  // 尝试在IDLE模式调用编辑API
  editor.loadModel('M3EF684585');
} catch (error) {
  if (error.message.includes('not in edit mode')) {
    console.error('Please enter edit mode first');
    editor.enterModelEditMode(model);
  }
}

try {
  // 尝试在场景编辑模式调用模型编辑API
  editor.createSKU('新SKU');
} catch (error) {
  if (error.message.includes('only available in model edit mode')) {
    console.error('This API is only for model editing');
  }
}

try {
  // 切换到不存在的SKU
  editor.switchSKU('sku_not_exist');
} catch (error) {
  if (error.message.includes('SKU not found')) {
    console.error('SKU does not exist');
    // 显示SKU列表供用户选择
    const skus = editor.getSKUs();
    console.log('Available SKUs:', skus.map(s => s.name));
  }
}
```

---

### 6.7 React集成示例

```typescript
import React, { useEffect, useRef, useState } from 'react';
import { EditorViewer, EditorMode } from 'anker3d';

function ModelEditor() {
  const containerRef = useRef<HTMLDivElement>(null);
  const editorRef = useRef<EditorViewer | null>(null);
  const [currentMode, setCurrentMode] = useState<EditorMode>(EditorMode.IDLE);
  const [skus, setSKUs] = useState<SKU[]>([]);

  useEffect(() => {
    if (!containerRef.current) return;

    // 创建编辑器
    const editor = new EditorViewer(containerRef.current);
    editorRef.current = editor;

    // 监听模式变化
    editor.addEventListener('modeChanged', (e) => {
      setCurrentMode(e.newMode);
    });

    // 监听SKU变化
    editor.addEventListener('skuChanged', (e) => {
      // 更新UI
    });

    // 加载模型并进入编辑模式
    (async () => {
      const model = await editor.loadModel({ urlOrId: 'M3EF684585' });
      editor.enterModelEditMode(model);
      setSKUs(editor.getSKUs());
    })();

    return () => {
      editor.dispose();
    };
  }, []);

  const handleCreateSKU = () => {
    if (!editorRef.current) return;
    const name = prompt('Enter SKU name:');
    if (name) {
      const sku = editorRef.current.createSKU(name);
      setSKUs(editorRef.current.getSKUs());
    }
  };

  const handleSwitchSKU = (skuId: string) => {
    editorRef.current?.switchSKU(skuId);
  };

  const handleSave = async () => {
    if (!editorRef.current) return;
    const savedId = await editorRef.current.save();
    alert(`Saved with ID: ${savedId}`);
  };

  return (
    <div>
      <div ref={containerRef} style={{ width: '100%', height: '600px' }} />

      <div className="toolbar">
        <div>Mode: {currentMode}</div>

        {currentMode === EditorMode.MODEL_EDIT && (
          <>
            <button onClick={handleCreateSKU}>Create SKU</button>
            <select onChange={(e) => handleSwitchSKU(e.target.value)}>
              {skus.map(sku => (
                <option key={sku.skuId} value={sku.skuId}>
                  {sku.name}
                </option>
              ))}
            </select>
            <button onClick={handleSave}>Save</button>
          </>
        )}
      </div>
    </div>
  );
}

export default ModelEditor;
```

---

## 附录

### A. 完整类型导出

```typescript
// 从主包导出
export {
  // Classes
  EditorViewer,
  ModelEditorComponent,
  SceneEditorComponent,
  SKUManager,
  SelectionManager,
  TransformController,
  HierarchyManager,

  // Enums
  EditorMode,
  ToneMappingType,

  // Interfaces
  EditorLoadModelOptions,
  EnvironmentLoadOptions,
  SaveOptions,
  ModelEditorData,
  SceneData,
  SKU,
  MaterialConfig,
  MaterialProperties,
  TransformData,
  CameraParams,
  SceneNode,

  // Events
  EditorModeChangedEvent,
  SKUChangedEvent,
  SKUCreatedEvent,
  SKUDeletedEvent,
  MaterialChangedEvent,
  ObjectAddedEvent,
  ObjectRemovedEvent,
  ObjectSelectedEvent,
  ObjectDeselectedEvent,
  ObjectTransformedEvent,
  HierarchyChangedEvent,
  PerformanceWarningEvent
};
```

---

### B. 版本兼容性

| API | 版本 | 状态 |
|-----|------|------|
| EditorViewer 基础功能 | v1.0 | ✅ 稳定 |
| 统一API路由 | v1.0 | ✅ 稳定 |
| SKU管理 | v1.0 | ✅ 稳定 |
| 场景编辑 | v1.0 | ✅ 稳定 |
| 性能监控 | v1.0 | ✅ 稳定 |
| 撤销/重做 | v2.0 | 📅 计划中 |
| 多视口 | v2.0 | 📅 计划中 |
| 动画编辑 | v3.0 | 📅 计划中 |

---

### C. 性能建议

#### C.1 SKU切换优化

- ✅ 使用Diff算法（已实现）
- ✅ 批量更新材质时使用`batchUpdateMaterials()`
- ✅ 避免频繁切换SKU

#### C.2 场景编辑优化

- ✅ 复用模型实例（通过ResourceManager自动处理）
- ✅ 使用LOD（Level of Detail）机制
- ✅ 监听性能警告事件并手动降低质量

#### C.3 内存管理

- ✅ 及时调用`exitEditMode()`清理资源
- ✅ 使用`editor.dispose()`销毁编辑器实例
- ✅ 避免在内存中缓存大量场景数据

---

### D. 调试技巧

```typescript
// 启用调试模式
editor.enableDebugMode();

// 监听所有事件
const allEvents = [
  'modeChanged',
  'skuChanged',
  'skuCreated',
  'skuDeleted',
  'materialChanged',
  'objectAdded',
  'objectRemoved',
  'objectSelected',
  'objectDeselected',
  'objectTransformed',
  'hierarchyChanged',
  'performanceWarning'
];

allEvents.forEach(eventType => {
  editor.addEventListener(eventType, (e) => {
    console.log(`[${eventType}]`, e);
  });
});

// 获取当前状态
console.log('Current mode:', editor.currentMode);
console.log('Current model:', editor.getCurrentModel());
console.log('Scene tree:', editor.getSceneTree());
```

---

## 结语

本文档提供了Anker3D编辑器的完整API参考。所有API都遵循以下设计原则：

1. **统一API**：相同功能在不同模式下使用相同的API
2. **智能路由**：EditorViewer根据状态自动路由到正确的实现
3. **类型安全**：完整的TypeScript类型定义
4. **事件驱动**：所有状态变化都触发相应事件
5. **错误提示**：清晰的错误消息和异常处理

如有任何问题或建议，请参考：
- [编辑器设计文档](./editor-viewer-design.md)
- [决策记录](./editor-decision-log.md)
- [SKU数据结构](./sku-data-structure.md)
- [模型编辑数据结构](./model-editor-data-structure.md)
- [场景数据结构](./scene-data-structure.md)
