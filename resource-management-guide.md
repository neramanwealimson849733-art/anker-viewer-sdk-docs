# Anker3D SDK - 资源管理指南

> **版本**: v1.0  
> **日期**: 2026-01-30  
> **重要性**: 🔴 必读 - 避免内存泄漏的关键

---

## 📋 概述

Anker3D SDK 基于 Three.js 构建，需要手动管理 WebGL 资源（Geometry, Material, Texture 等）。
**不正确的资源管理会导致内存泄漏**，特别是在单页应用中频繁创建/销毁 Viewer 的场景。

本文档提供：
- 资源管理最佳实践
- 常见内存泄漏场景及解决方案
- 调试工具使用指南

---

## 🎯 核心原则

### 1. **谁创建，谁释放**

```typescript
// ✅ 正确
const viewer = new Viewer(container);
// ... 使用 viewer
viewer.dispose(); // 创建者负责释放

// ❌ 错误
const viewer = new Viewer(container);
// ... 使用后忘记释放 → 内存泄漏
```

### 2. **释放前移除引用**

```typescript
// ✅ 正确
this.viewer?.dispose();
this.viewer = null; // 移除引用，帮助 GC

// ⚠️ 可以，但不推荐
this.viewer?.dispose();
// 仍持有引用，GC 需要等到 this 被回收
```

### 3. **父对象负责释放子对象**

```typescript
// Viewer 的 dispose 会自动释放：
viewer.dispose();
  ├─ sceneManager.dispose()
  │   └─ 所有场景和模型
  ├─ renderManager.dispose()
  ├─ cameraManager.dispose()
  └─ componentManager.dispose()
      └─ 所有组件
```

---

## 🔧 API 使用指南

### IDisposable 接口

所有持有资源的对象都实现了 `IDisposable` 接口：

```typescript
interface IDisposable {
  dispose(): void;          // 释放资源
  readonly isDisposed: boolean;  // 是否已释放
}
```

### 核心类的资源管理

#### 1. **Viewer / EditorViewer**

```typescript
const viewer = new Viewer(container, params);

// 使用完毕后必须释放
viewer.dispose();

// 检查是否已释放
if (viewer.isDisposed) {
  console.log('Viewer 已释放');
}

// ⚠️ 释放后不可再使用
viewer.loadModel({...}); // 错误！会抛出异常或产生未定义行为
```

**释放内容**：
- ✅ 渲染器（WebGLRenderer）
- ✅ 所有场景和模型
- ✅ 所有纹理和材质
- ✅ 所有组件
- ✅ 事件监听器
- ✅ 控制器

#### 2. **ModelObject**

```typescript
const model = await viewer.loadModelAsync({
  urlOrId: '/models/product.glb'
});

// 从场景移除并释放
scene.removeModel(model);
model.dispose();

// 或者直接释放场景（会自动释放所有模型）
scene.dispose();
```

**释放内容**：
- ✅ Geometry
- ✅ Material 及其所有贴图
- ✅ 动画
- ✅ 底部阴影

#### 3. **ResourceManager（全局单例）**

```typescript
import { ResourceManager } from 'anker3d-sdk';

// 删除单个缓存资源
ResourceManager.delete('model_id'); // 会自动调用 dispose()

// 清空所有缓存
ResourceManager.clear(); // 释放所有缓存的模型和纹理

// ⚠️ 注意：清空缓存后，已加载的模型仍可使用
// 只是缓存中不再保留副本
```

#### 4. **组件（Component）**

```typescript
const bloomPass = new BloomPassComponent({...});
viewer.addComponent(bloomPass);

// 方式1: 移除但不释放（可以重新添加）
viewer.removeComponent(bloomPass);

// 方式2: 移除并释放（推荐）
viewer.destroyComponent(bloomPass);

// 方式3: 手动释放
bloomPass.dispose();
```

---

## ⚠️ 常见内存泄漏场景

### 场景 1：SPA 路由切换

**问题**：
```typescript
// React 示例
function ModelViewer() {
  useEffect(() => {
    const viewer = new Viewer(containerRef.current);
    
    return () => {
      // ❌ 忘记释放！
    };
  }, []);
}
```

**解决方案**：
```typescript
function ModelViewer() {
  useEffect(() => {
    const viewer = new Viewer(containerRef.current);
    
    return () => {
      viewer.dispose(); // ✅ 组件卸载时释放
    };
  }, []);
}
```

### 场景 2：频繁加载/切换模型

**问题**：
```typescript
async function loadProduct(productId: string) {
  const model = await viewer.loadModelAsync({
    urlOrId: productId
  });
  viewer.scene.addModel(model);
  // ❌ 旧模型未释放
}

// 调用多次后内存累积
loadProduct('product1');
loadProduct('product2'); // product1 未释放
loadProduct('product3'); // product1, product2 未释放
```

**解决方案 A：手动管理**
```typescript
let currentModel: ModelObject | null = null;

async function loadProduct(productId: string) {
  // 释放旧模型
  if (currentModel) {
    viewer.scene.removeModel(currentModel);
    currentModel.dispose();
  }
  
  // 加载新模型
  currentModel = await viewer.loadModelAsync({
    urlOrId: productId
  });
  viewer.scene.addModel(currentModel);
}
```

**解决方案 B：清空场景**
```typescript
async function loadProduct(productId: string) {
  // 清空场景（释放所有模型）
  viewer.scene.clear();
  
  // 加载新模型
  const model = await viewer.loadModelAsync({
    urlOrId: productId
  });
  viewer.scene.addModel(model);
}
```

### 场景 3：动态创建纹理/材质

**问题**：
```typescript
function changeMaterialColor(color: string) {
  const material = new MeshStandardMaterial({
    color: color
  });
  mesh.material = material;
  // ❌ 旧材质未释放
}
```

**解决方案**：
```typescript
function changeMaterialColor(color: string) {
  // 释放旧材质
  const oldMaterial = mesh.material;
  if (oldMaterial) {
    oldMaterial.dispose();
  }
  
  // 应用新材质
  const material = new MeshStandardMaterial({
    color: color
  });
  mesh.material = material;
}

// 或者直接修改现有材质（更推荐）
function changeMaterialColor(color: string) {
  if (mesh.material instanceof MeshStandardMaterial) {
    mesh.material.color.set(color);
    // 不需要 dispose，没有创建新对象
  }
}
```

### 场景 4：事件监听器未清理

**问题**：
```typescript
class MyComponent implements IDisposable {
  constructor() {
    window.addEventListener('resize', this.onResize);
    viewer.on('modelLoaded', this.onModelLoaded);
  }
  
  dispose() {
    // ❌ 忘记移除事件监听器
  }
}
```

**解决方案**：
```typescript
class MyComponent implements IDisposable {
  private _isDisposed = false;
  
  constructor() {
    this.onResize = this.onResize.bind(this);
    this.onModelLoaded = this.onModelLoaded.bind(this);
    
    window.addEventListener('resize', this.onResize);
    viewer.on('modelLoaded', this.onModelLoaded);
  }
  
  get isDisposed() { return this._isDisposed; }
  
  dispose() {
    if (this._isDisposed) return;
    
    // ✅ 清理事件监听器
    window.removeEventListener('resize', this.onResize);
    viewer.off('modelLoaded', this.onModelLoaded);
    
    this._isDisposed = true;
  }
}
```

---

## 🛠️ 调试工具

### 1. ResourceTracker（开发模式）

用于追踪资源创建和释放，发现泄漏：

```typescript
import { ResourceTracker } from 'anker3d-sdk';

const tracker = new ResourceTracker();

// 追踪资源
const viewer = tracker.track(new Viewer(container));
const model = tracker.track(await viewer.loadModelAsync({...}));

// 检查活跃资源
console.log('活跃资源数量:', tracker.getActiveCount());
tracker.printDebugInfo();

// 使用完毕后释放
tracker.dispose(); // 会自动释放所有追踪的资源

// 检查是否有泄漏
if (tracker.getActiveCount() > 0) {
  console.warn('检测到资源泄漏！');
  console.log(tracker.getActiveResources());
}
```

### 2. Chrome DevTools Memory Profiler

**步骤**：

1. **打开 DevTools** → **Memory** 选项卡

2. **操作流程**：
   ```typescript
   // 1. 拍摄初始快照
   // DevTools: Take Snapshot 1
   
   // 2. 创建大量对象
   const viewers = [];
   for (let i = 0; i < 100; i++) {
     viewers.push(new Viewer(container));
   }
   
   // DevTools: Take Snapshot 2
   
   // 3. 释放所有对象
   viewers.forEach(v => v.dispose());
   viewers.length = 0;
   
   // 4. 强制垃圾回收
   // DevTools: Collect Garbage (🗑️ 图标)
   
   // DevTools: Take Snapshot 3
   ```

3. **分析结果**：
   - Snapshot 2 应该比 Snapshot 1 大很多
   - Snapshot 3 应该接近 Snapshot 1
   - 如果 Snapshot 3 远大于 Snapshot 1 → **内存泄漏**

4. **定位泄漏**：
   - 比较 Snapshot 2 和 Snapshot 3
   - 查找 "Detached DOM" 节点
   - 查找 Three.js 对象（WebGLTexture, WebGLBuffer 等）

### 3. Performance Monitor

Chrome DevTools → **More Tools** → **Performance Monitor**

监控指标：
- **JS Heap Size**：应该在释放后下降
- **DOM Nodes**：注意异常增长
- **Event Listeners**：应该在 dispose 后减少

---

## ✅ 最佳实践检查清单

### 开发时

- [ ] 每个创建 Viewer 的地方，都有对应的 dispose 调用
- [ ] SPA 路由卸载时释放 Viewer
- [ ] 动态加载模型时释放旧模型
- [ ] 自定义组件实现 IDisposable 接口
- [ ] 事件监听器在 dispose 中清理
- [ ] 使用 ResourceTracker 辅助开发

### 代码审查时

- [ ] 检查 `new Viewer` → 是否有 `dispose`
- [ ] 检查 `loadModel` → 是否释放旧模型
- [ ] 检查 `new Material` → 是否释放旧材质
- [ ] 检查 `addEventListener` → 是否有 `removeEventListener`
- [ ] 检查 `setInterval/setTimeout` → 是否有 clear

### 测试时

- [ ] 运行 Memory Profiler 检查泄漏
- [ ] 长时间运行测试（如连续切换路由100次）
- [ ] 监控 JS Heap Size 是否持续增长

---

## 📚 进阶主题

### 自定义组件资源管理模板

```typescript
import { Component } from 'anker3d-sdk';
import type { IDisposable } from 'anker3d-sdk';

export class MyCustomComponent extends Component implements IDisposable {
    private _isDisposed = false;
    
    // Three.js 资源
    private geometry?: BufferGeometry;
    private material?: Material;
    private mesh?: Mesh;
    
    // DOM 资源
    private domElement?: HTMLElement;
    
    // 事件处理器（需要 bind）
    private onResize: () => void;
    
    constructor() {
        super();
        
        // Bind 事件处理器
        this.onResize = this.handleResize.bind(this);
        
        this.init();
    }
    
    get isDisposed(): boolean {
        return this._isDisposed;
    }
    
    private init(): void {
        // 创建 Three.js 对象
        this.geometry = new BoxGeometry(1, 1, 1);
        this.material = new MeshStandardMaterial({ color: 0xff0000 });
        this.mesh = new Mesh(this.geometry, this.material);
        
        // 创建 DOM 元素
        this.domElement = document.createElement('div');
        document.body.appendChild(this.domElement);
        
        // 添加事件监听
        window.addEventListener('resize', this.onResize);
    }
    
    private handleResize(): void {
        // 处理逻辑
    }
    
    /**
     * 释放所有资源
     */
    public dispose(): void {
        if (this._isDisposed) {
            console.warn('[MyCustomComponent] Already disposed');
            return;
        }
        
        // 1. 清理 Three.js 资源
        this.geometry?.dispose();
        this.material?.dispose();
        // mesh 不需要 dispose，但应该从父对象移除
        if (this.mesh?.parent) {
            this.mesh.parent.remove(this.mesh);
        }
        
        // 2. 清理 DOM 资源
        this.domElement?.remove();
        
        // 3. 清理事件监听器
        window.removeEventListener('resize', this.onResize);
        
        // 4. 清空引用（帮助 GC）
        this.geometry = undefined;
        this.material = undefined;
        this.mesh = undefined;
        this.domElement = undefined;
        
        // 5. 标记为已释放
        this._isDisposed = true;
        
        // 6. 调用父类 dispose（如果有）
        super.dispose();
    }
}
```

---

## 🔗 相关资源

- [Three.js 官方文档 - 如何释放资源](https://threejs.org/docs/#manual/introduction/How-to-dispose-of-objects)
- [Chrome DevTools Memory Profiler 指南](https://developer.chrome.com/docs/devtools/memory-problems/)
- [JavaScript 内存管理基础](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management)

---

## ❓ FAQ

**Q: 我需要手动释放 ResourceManager 缓存的资源吗？**

A: 不需要。`viewer.dispose()` 会自动调用 `ResourceManager.clear()`。
   如果你想手动清理特定缓存，可以调用 `ResourceManager.delete(key)`。

**Q: dispose 后对象可以重新使用吗？**

A: 不可以。dispose 后对象处于不可用状态，必须创建新对象。

**Q: 什么时候使用 ResourceTracker？**

A: 仅在开发/调试时使用，帮助发现泄漏。生产环境不需要。

**Q: 如何判断是否存在内存泄漏？**

A: 使用 Chrome Memory Profiler，观察：
   - 创建 100 个 Viewer → dispose → 强制 GC
   - 如果内存未回到初始水平 → 泄漏

**Q: SceneObject.clear() 和 destroy() 的区别？**

A: 
   - `clear()`: 移除所有子对象，但不释放资源
   - `destroy()`: 移除所有子对象 **并释放资源**

---

**最后更新**: 2026-01-30  
**维护者**: SDK 架构团队
