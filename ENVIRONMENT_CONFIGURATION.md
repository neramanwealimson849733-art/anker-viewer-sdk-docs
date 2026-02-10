# 环境配置指南

本文档说明 Anker3D SDK 的环境配置机制和使用方式。

## 🎯 配置架构

### 环境类型

SDK 支持三种环境：

| 环境 | 说明 | API 地址 |
|------|------|---------|
| `dev` | 开发环境 | http://localhost/api |
| `test` | 测试环境 | https://studio3d-sit.anker-in.com/api |
| `prod` | 生产环境 | https://studio3d-uat.anker-in.com/api |

### 构建模式

| 构建命令 | 环境 | 说明 |
|---------|------|------|
| `npm run dev` | dev | 开发服务器 |
| `npm run build` | test | 测试环境构建 |
| `npm run build:prod` | prod | 生产环境构建 |

---

## 📝 使用方式

### 方式 1：使用环境配置模块（推荐）⭐

```typescript
import { APP_ENV, API_CONFIG, isDev, isTest, isProd } from './env';

// 获取当前环境
console.log('当前环境:', APP_ENV);  // 'dev' | 'test' | 'prod'

// 环境判断
if (isDev) {
    console.log('开发环境');
}

// 获取 API 配置
const apiUrl = API_CONFIG.getBaseURL();
console.log('API 地址:', apiUrl);

// 修改配置
API_CONFIG.useRelativePath = true;  // 使用相对路径模式
```

### 方式 2：使用环境变量

```typescript
// 在构建后的代码中
console.log(__APP_ENV__);  // 'dev' | 'test' | 'prod'

// 或使用 Vite 环境变量
console.log(import.meta.env.VITE_APP_ENV);
console.log(import.meta.env.MODE);
```

---

## 🔧 环境配置模块 API

### `src/env.ts`

#### 导出变量

```typescript
// 当前环境
export const APP_ENV: AppEnv;  // 'dev' | 'test' | 'prod'

// 环境判断
export const isDev: boolean;   // APP_ENV === 'dev'
export const isTest: boolean;  // APP_ENV === 'test'
export const isProd: boolean;  // APP_ENV === 'prod'

// API 配置对象
export const API_CONFIG: {
    useRelativePath: boolean;
    baseURL: Record<AppEnv, string>;
    getBaseURL(): string;
};

// 环境信息（调试用）
export const ENV_INFO: {
    env: AppEnv;
    mode: string;
    dev: boolean;
    prod: boolean;
    baseURL: string;
};
```

#### 环境检测优先级

环境变量按以下优先级检测：

1. **构建时注入的全局变量** `__APP_ENV__`（最高优先级）
   - 通过 Vite 的 `define` 配置注入
   - 在生产构建中使用

2. **Vite 环境变量** `import.meta.env.VITE_APP_ENV`
   - 从 `.env` 文件或命令行读取
   - 开发模式下使用

3. **从 Mode 推断** `import.meta.env.MODE`
   - 从 Vite 构建模式推断
   - `test`/`test-app` → `test`
   - `prod`/`prod-app` → `prod`

4. **默认值** `'dev'`
   - 无法检测时使用

---

## 🔄 API 路径模式

### 模式 1：绝对路径模式（默认）

```typescript
API_CONFIG.useRelativePath = false;

// 根据环境使用对应的绝对 URL
// dev:  http://localhost/api
// test: https://studio3d-sit.anker-in.com/api
// prod: https://studio3d-uat.anker-in.com/api
```

**适用场景**：
- 开发环境（跨域代理）
- 独立部署的前端应用

### 模式 2：相对路径模式

```typescript
API_CONFIG.useRelativePath = true;

// 所有环境统一使用相对路径
// /api
```

**适用场景**：
- 前后端同域部署
- 使用 Nginx 代理
- 避免跨域问题

---

## ⚙️ 配置环境

### 开发环境配置

创建 `.env.development`（可选）：

```bash
# 开发环境
VITE_APP_ENV=dev
```

### 测试环境配置

创建 `.env.test`（可选）：

```bash
# 测试环境
VITE_APP_ENV=test
```

### 生产环境配置

创建 `.env.production`（可选）：

```bash
# 生产环境
VITE_APP_ENV=prod
```

> **注意**：这些 `.env` 文件是可选的，因为环境已经通过构建模式自动推断。

---

## 🔍 调试环境配置

### 查看当前环境信息

```typescript
import { ENV_INFO } from './env';

console.log(ENV_INFO);
// 输出：
// {
//   env: 'test',
//   mode: 'test',
//   dev: false,
//   prod: false,
//   baseURL: 'https://studio3d-sit.anker-in.com/api'
// }
```

### 浏览器控制台

打开浏览器控制台，查看自动打印的环境信息：

```
[Env] Environment Info: {env: 'test', mode: 'test', ...}
[ModelAPI] Environment Info: {env: 'test', baseURL: '...'}
```

---

## 🚨 常见问题

### Q1: 为什么环境变量获取不到？

**A**: 环境变量有多种来源，检查顺序：
1. 确认构建模式：`npm run build` = test, `npm run build:prod` = prod
2. 检查 `src/env.ts` 中的环境检测逻辑
3. 查看浏览器控制台的环境信息输出

### Q2: 如何在运行时切换 API 地址？

**A**: 修改 `API_CONFIG`：

```typescript
import { API_CONFIG } from './env';

// 切换到相对路径模式
API_CONFIG.useRelativePath = true;

// 或直接修改 baseURL
API_CONFIG.baseURL.test = 'https://new-api.example.com/api';
```

### Q3: 开发时如何使用本地 API？

**A**: 有两种方式：

**方式 1**：修改 `src/env.ts`：
```typescript
baseURL: {
    dev: 'http://localhost:3000/api',  // 修改为你的本地地址
    // ...
}
```

**方式 2**：使用相对路径 + Vite 代理：
```typescript
// vite.config.ts
export default defineConfig({
    server: {
        proxy: {
            '/api': {
                target: 'http://localhost:3000',
                changeOrigin: true
            }
        }
    }
});
```

---

## 📦 迁移指南

### 从旧配置迁移

如果你的代码使用了旧的 `GlobalConfig`：

```typescript
// ❌ 旧方式（已废弃）
import { GlobalConfig } from './config';
const apiUrl = GlobalConfig.APIUrl[env];

// ✅ 新方式
import { API_CONFIG } from './env';
const apiUrl = API_CONFIG.getBaseURL();
```

---

## 📚 相关文档

- [构建配置文档](./BUILD_CONFIGURATION.md)
- [Vite 环境变量文档](https://vitejs.dev/guide/env-and-mode.html)

---

**维护者**: Anker3D 开发团队  
**最后更新**: 2026-01-30
