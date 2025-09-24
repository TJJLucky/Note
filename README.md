基于 uni-app 开发的跨平台应用，支持微信小程序、H5、App 等多端部署。

## 技术栈

### 核心技术

- **uni-app 3.x** - 跨平台开发框架
- **Vue 3** - 前端框架（使用 Composition API）
- **TypeScript** - 类型安全的 JavaScript
- **Vite** - 现代构建工具
- **Pinia** - 状态管理（支持数据持久化）
- **uv-UI** - uni-app生态专用的UI框架

### 开发工具

- **ESLint** - 代码语法检查
- **Prettier** - 代码格式化
- **Husky** - Git hooks 管理
- **commitlint** - 提交信息规范检查
- **lint-staged** - 暂存区文件检查

### 特性支持

- ✅ 多端适配（微信小程序、H5、App）
- ✅ TypeScript 类型支持
- ✅ 分包优化
- ✅ 响应式布局
- ✅ 状态管理与持久化
- ✅ 统一请求封装
- ✅ 工具函数库
- ✅ 代码规范检查

## 目录结构

```
front/
├── src/                          # 源代码目录
│   ├── App.vue                   # 应用入口文件
│   ├── main.ts                   # 应用启动文件
│   ├── manifest.json             # 应用配置文件
│   ├── pages.json                # 页面配置文件
│   ├── uni.scss                  # 全局样式文件
│   ├── env.d.ts                  # 环境变量类型声明
│   │
│   ├── pages/                    # 主包页面
│   │   └── index/                # 首页
│   │       └── index.vue
│   │
│   ├── subpackage1/              # 分包1
│   │   └── example/              # 示例页面
│   │       └── index.vue
│   │
│   ├── components/               # 组件目录
│   │
│   ├── stores/                   # 状态管理
│   │   ├── index.ts             # Pinia 配置
│   │   └── modules/             # 状态模块
│   │       └── user.ts          # 用户状态
│   │
│   ├── http/                     # HTTP 请求封装
│   │   ├── README.md            # 使用说明
│   │   ├── request/             # 请求工具
│   │   │   └── index.ts         # 请求封装
│   │   └── apiConfig/           # API 配置
│   │       └── index.ts         # 接口配置
│   │
│   ├── utils/                    # 工具函数库
│   │   ├── index.ts             # 工具函数导出
│   │   └── debounce.ts          # 防抖函数
│   │
│   └── static/                   # 静态资源
│       └── logo.png
│
├── .vscode/                      # VS Code 配置
├── .husky/                       # Git hooks
├── dist/                         # 构建输出目录
├── node_modules/                 # 依赖包
│
├── package.json                  # 项目配置
├── tsconfig.json                 # TypeScript 配置
├── vite.config.ts                # Vite 配置
├── eslint.config.mjs             # ESLint 配置
├── .prettierrc.json              # Prettier 配置
├── commitlint.config.js          # 提交信息规范配置
├── .gitignore                    # Git 忽略配置
└── readme.md                     # 项目说明文档
```

## 项目启动

### 首次启动

克隆项目后，代码检查会自动配置（通过 postinstall 脚本）：

```bash
# 安装依赖时会自动执行 husky install
npm install
```

如果遇到钩子不生效的问题，可以手动执行：

```bash
# 重新安装 Git hooks
npx husky install
```

### 开发调试

建议使用 VS Code 作为编辑器，也可以使用 HBuilderX。
VS Code 环境建议安装 uni-app 相关插件：

- **uni-create-view** - 快速创建 uni-app 页面
- **uni-helper** - uni-app 代码提示
- **uniapp小程序扩展** - 鼠标悬停查看文档

#### 修改 uni-create-view 配置

![配置图片](image.png)

### 开发命令

```bash
# 微信小程序（热更新）
npm run dev:mp-weixin
```

### 构建命令

```bash
# 构建微信小程序
npm run build:mp-weixin
```

### 代码规范

项目已配置完整的代码质量检查流程，包括：

- **ESLint** - 代码语法和规范检查
- **Prettier** - 代码格式化
- **lint-staged** - 只检查暂存区文件
- **commitlint** - 提交信息规范检查

#### 自动代码检查

项目使用 Husky 配置了 Git hooks，会在提交时自动执行代码检查：

1. **pre-commit 钩子** - 在提交前自动格式化和检查暂存区的代码
2. **commit-msg 钩子** - 检查提交信息是否符合规范

#### 手动执行检查

```bash
# 代码检查并自动修复
npm run lint:fix

# 代码格式化
npm run format

# 类型检查
npm run type-check

# 检查暂存区文件（lint-staged）
npm run lint-staged
```

## 核心功能

### 1. 状态管理

使用 Pinia 进行状态管理，支持数据持久化：

```typescript
// stores/modules/user.ts
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  // 状态定义
  // 动作定义
  // 持久化配置
})
```

### 2. HTTP 请求

统一的请求封装，支持：

- 自动 token 管理
- 全局加载状态
- 统一错误处理
- 文件上传

详细使用方法请参考：[HTTP 请求封装说明](src/http/README.md)

### 3. 工具函数

提供常用的工具函数：

- 防抖函数 (`debounce`)

统一导出方式：

```typescript
// utils/index.ts
export { default as debounce } from './debounce'
```

### 4. 分包优化

- 支持分包预加载
- 优化首屏加载速度
- 按需加载页面资源

## 开发规范

### 提交规范

格式：`<type>(<scope>): <subject>`

示例：`feat(auth): 增加用户登录功能`

| 类型     | 描述                              |
| -------- | --------------------------------- |
| feat     | 新增功能（feature）               |
| fix      | 修复缺陷（bug fix）               |
| docs     | 修改文档（documentation）         |
| style    | 格式变动（不影响代码逻辑）        |
| refactor | 代码重构（非修复或新增功能）      |
| perf     | 性能优化                          |
| test     | 增加或修改测试                    |
| build    | 构建相关的更改（如webpack配置）   |
| ci       | 持续集成配置或脚本变更            |
| chore    | 杂项改动（如修改 `.gitignore`） |
| revert   | 回滚历史提交                      |

### 自定义组件规范

组件文件夹命名使用 `kebab-case`，如 `user-info`。组件目录结构应包含 `index.vue` 文件。
使用组件时，需在组件文件夹的名字前加上 `my-` 前缀。

```vue
// 使用user-info 组件
<template>
  <my-user-info :userName="name" :avatarSrc="avatar" />
</template>
```

### 主题样式设置

首先，在 `uni.scss` 中设置全局主题样式：

```scss
// uni.scss
$primary-color: #42b983; // 主题色
$secondary-color: #35495e; // 次要色
$font-size-base: 16px; // 基础字体大小
```

然后，在vue文件中使用：

```vue
<style lang="scss">
.my-user-info {
  background-color: $primary-color;
  color: white;
  padding: 10px;
  border-radius: 5px;
}
</style>
```

### 单位设置

如果设计稿中的尺寸是 `16px`,若你需要使用rpx单位，可以直接使用封装好的scss函数’to-rpx‘进行转换：

```scss
.my-component {
  width: to-rpx(16px); // 转换为rpx单位
  height: to-rpx(16px);
}
```

## 注意事项

1. **环境配置**：根据实际需求配置 `manifest.json` 中的 `appid` 等信息
2. **权限配置**：Android 和 iOS 平台需要配置相应的权限
3. **分包策略**：合理使用分包减少首屏加载时间
4. **兼容性**：不同平台的 API 差异需要特别注意
5. **性能优化**：注意图片压缩、代码分割等优化措施

## 相关链接

- [uni-app 官方文档](https://uniapp.dcloud.net.cn/)
- [Vue 3 文档](https://v3.cn.vuejs.org/)
- [TypeScript 文档](https://www.typescriptlang.org/)
- [Pinia 文档](https://pinia.vuejs.org/)
- [Vite 文档](https://cn.vitejs.dev/)
- [uv-UI 文档](https://www.uvui.cn/components/intro.html)

## 许可证

本项目采用 MIT 许可证。
