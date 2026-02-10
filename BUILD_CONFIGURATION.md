# Anker3D SDK 构建配置文档

本文档说明 Anker3D SDK 的构建配置架构、构建流程和输出产物。

## 🚀 快速开始

```bash
# 开发模式
npm run dev

# 构建（测试环境）- 一次性完成 SDK + 应用构建
npm run build

# 构建（生产环境）
npm run build:prod
```

**一条命令完成所有构建**：
- ✅ SDK 库文件（anker3d.module.js 等）
- ✅ 应用入口（index.html + assets/）
- ✅ 示例页面（自动替换 SDK 路径为 CDN）
- ✅ 文档和静态资源

## 📂 构建配置文件结构

构建配置已模块化拆分为以下文件：

```
├── vite.config.ts           # 主配置文件（~70 行）
├── scripts/
│   ├── build-config.ts      # 构建环境配置管理
│   ├── vite-plugins.ts      # Vite 插件配置
│   └── rollup-config.ts     # Rollup 输出配置
```

### 1. `vite.config.ts` - 主配置文件

简洁的主配置文件，负责整合各模块配置：

```typescript
import { defineConfig } from 'vite';
import { createBuildConfig, getDefineConfig, logBuildConfig } from './scripts/build-config';
import { getPlugins } from './scripts/vite-plugins';
import { getAllOutputs } from './scripts/rollup-config';

export default defineConfig(({ mode, command }) => {
    const buildConfig = createBuildConfig(mode, command);
    logBuildConfig(buildConfig);
    
    return {
        base: buildConfig.basePath,
        define: getDefineConfig(buildConfig),
        plugins: getPlugins(),
        build: {
            lib: {
                entry: 'src/index.ts',
                name: 'Anker3D',
            },
            rollupOptions: {
                output: getAllOutputs()
            }
        }
    };
});
```

### 2. `scripts/build-config.ts` - 构建环境配置

管理构建环境和配置参数：

```typescript
export type BuildEnv = 'dev' | 'test' | 'prod';

export interface BuildConfig {
    env: BuildEnv;           // 构建环境
    command: string;         // Vite 命令（serve/build）
    basePath: string;        // 基础路径
    isProduction: boolean;   // 是否生产环境
    sourcemap: boolean;      // 是否生成 Source Map
}

// 创建构建配置
export function createBuildConfig(mode: string, command: string): BuildConfig;

// 获取基础路径
export function getBasePath(env: BuildEnv): string;

// 获取 define 配置
export function getDefineConfig(config: BuildConfig): Record<string, string>;

// 打印构建配置信息
export function logBuildConfig(config: BuildConfig): void;
```

### 3. `scripts/vite-plugins.ts` - Vite 插件配置

统一管理所有 Vite 插件：

```typescript
// Docs 路由重定向插件
export function docsRedirectPlugin(): Plugin;

// 静态文件复制插件
export function staticCopyPlugin(): Plugin;

// 获取所有插件
export function getPlugins(): Plugin[];
```

**插件功能：**
- `docsRedirectPlugin()` - 将 `/docs` 重定向到 `/docs/api/index.html`
- `staticCopyPlugin()` - 复制文档、示例和静态资源到 `dist/`

### 4. `scripts/rollup-config.ts` - Rollup 输出配置

定义 SDK 的输出格式和配置：

```typescript
// ES Module 输出（未压缩）
export function getESModuleOutput(): OutputOptions;

// ES Module 输出（压缩）
export function getESModuleMinOutput(): OutputOptions;

// UMD 输出（压缩，用于浏览器）
export function getUMDMinOutput(): OutputOptions;

// 获取所有输出配置
export function getAllOutputs(): OutputOptions[];
```

---

## 🚀 构建流程说明

### 开发模式
```bash
npm run dev
```
启动 Vite 开发服务器，支持热重载，端口 8012。

### 构建流程（一键完成）

运行 `npm run build` 会自动执行以下步骤：

**步骤 1：SDK 库构建**
```bash
tsc -b && vite build --mode test
```
- ✅ TypeScript 编译
- ✅ 构建 SDK 文件（anker3d.module.js 等）
- ✅ 复制静态资源（examples, docs 等）
- ✅ 自动替换 examples 中的 SDK 路径为 CDN 地址

**步骤 2：应用构建**
```bash
set BUILD_APP=true && vite build --mode test
```
- ✅ 构建 index.html 应用
- ✅ 打包应用代码到 assets/
- ✅ 生成 Source Map（测试模式）

**构建结果**：
```
dist/
├── anker3d.module.js          # SDK（未压缩）
├── anker3d.module.min.js      # SDK（压缩）
├── anker3d.min.js             # SDK（UMD）
├── index.html                 # 应用入口
├── assets/                    # 应用代码
│   └── index-*.js
├── examples/                  # 示例（SDK 路径已替换）
└── docs/                      # 文档
```

### 环境模式

| 模式 | 命令 | Source Map | 用途 |
|------|------|-----------|------|
| **test** | `npm run build` | ✅ 开启 | 测试环境 |
| **prod** | `npm run build:prod` | ❌ 关闭 | 生产环境 |

---

## 📦 输出产物说明

### SDK 文件

| 文件名 | 格式 | 压缩 | 大小 | 用途 |
|--------|------|------|------|------|
| `anker3d.module.js` | ES Module | ❌ | ~2.3MB | 开发/调试 |
| `anker3d.module.min.js` | ES Module | ✅ | ~1.1MB | 生产环境（支持 tree-shaking） |
| `anker3d.min.js` | UMD | ✅ | ~1.1MB | 浏览器直接引入 |

### 完整目录结构

构建完成后，`dist/` 目录结构如下：

```
dist/
├── index.html                  # ✨ 应用入口（Vite 构建）
├── assets/                     # ✨ 应用代码（打包后）
│   ├── index-*.js             # 应用主文件（~2.5MB）
│   └── index-*.js.map         # Source Map
├── anker3d.module.js          # SDK - ES Module（未压缩）
├── anker3d.module.min.js      # SDK - ES Module（压缩）
├── anker3d.min.js             # SDK - UMD（压缩）
├── anker3d.*.js.map           # Source Map（仅 test 模式）
├── examples/                  # 示例（✨ SDK 路径自动替换为 CDN）
│   ├── simple.html
│   ├── bloom.html
│   └── ...
├── docs/                      # 文档
│   ├── api/                   # TypeDoc API 文档
│   └── *.md                   # Markdown 文档
├── models/                    # 3D 模型资源
├── environments/              # HDR 环境贴图
├── draco/                     # Draco 解码器
├── animations/                # 动画资源
└── icons/                     # 图标资源
```

### 构建特性

✅ **自动化处理**：
1. index.html 由 Vite 构建，引用打包后的 assets/
2. examples/*.html 中的 SDK 路径自动替换为 CDN 地址
   - 开发环境：`import * as Anker3D from "../src/index.ts"`
   - 构建后：`import * as Anker3D from "https://studio3d-sit.anker-in.com/anker-viewer/anker3d.module.js"`

---

## 🔧 使用方式

### 方式一：ES Module（推荐）

```javascript
import { Viewer, CameraType } from './anker3d.module.min.js';

const viewer = new Viewer(container, {
    modelParams: {
        urlOrId: '/models/S1pro.glb'
    }
});
```

**优势**：
- 支持 tree-shaking
- 更小的打包体积
- 现代构建工具支持

### 方式二：UMD（浏览器直接引入）

```html
<script src="./anker3d.min.js"></script>
<script>
    const { Viewer, CameraType } = Anker3D;
    const viewer = new Viewer(container, {
        modelParams: {
            urlOrId: '/models/S1pro.glb'
        }
    });
</script>
```

**优势**：
- 无需构建工具
- 快速原型开发
- 兼容传统项目

---

## 📊 构建性能

### 构建速度
- **开发模式**：即时启动（Vite HMR）
- **测试构建**：~8-10 秒
- **生产构建**：~8-10 秒

### 产物大小

| 模式 | 未压缩 | Gzip | Brotli |
|------|--------|------|--------|
| ES Module | 2.3MB | 467KB | ~350KB |
| ES Module (min) | 1.1MB | 295KB | ~220KB |
| UMD (min) | 1.1MB | 295KB | ~220KB |

---

## ⚙️ 自定义配置

### 修改输出格式

编辑 `scripts/rollup-config.ts`：

```typescript
export function getCustomOutput(): OutputOptions {
    return {
        file: 'dist/anker3d.custom.js',
        format: 'iife',  // 或 'cjs', 'es', 'umd'
        name: 'Anker3D',
        // ... 其他配置
    };
}
```

### 添加新插件

编辑 `scripts/vite-plugins.ts`：

```typescript
import myPlugin from 'vite-plugin-xxx';

export function getPlugins() {
    return [
        docsRedirectPlugin(),
        staticCopyPlugin(),
        myPlugin()  // 添加新插件
    ];
}
```

### 修改构建环境

编辑 `scripts/build-config.ts`：

```typescript
export function getBasePath(env: BuildEnv): string {
    switch (env) {
        case 'prod': return 'https://cdn.example.com/anker3d/';
        case 'test': return './';
        default: return './';
    }
}
```

---

## 🐛 故障排查

### 构建失败

1. **检查 TypeScript 编译**：
   ```bash
   npm run type-check
   ```

2. **清理缓存**：
   ```bash
   rm -rf node_modules/.vite
   rm -rf dist
   npm run build
   ```

### Source Map 缺失

- 检查构建模式是否为 `test`
- 生产模式 (`prod`) 不生成 Source Map

### 静态资源未复制

- 检查 `vite-plugin-static-copy` 是否正确配置
- 确认 `public/` 目录下文件存在

---

## 📝 更新日志

### v1.0.1 (2026-01-30)
- ✅ 重构构建配置，拆分为模块化文件
- ✅ 移除 `app` 模式，统一构建流程
- ✅ 简化 `package.json` 脚本
- ✅ 优化构建速度和输出大小

### v1.0.0
- 初始构建配置

---

## 🤝 贡献

如需修改构建配置：

1. 修改相应的配置模块
2. 运行 `npm run build` 测试
3. 确保所有输出产物正确生成
4. 提交 Pull Request

---

**构建配置维护者**: Anker3D Team  
**最后更新**: 2026-01-30
