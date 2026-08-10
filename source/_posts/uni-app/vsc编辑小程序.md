---
title: uni-app基础篇
categories:
  - uni-app
tags:
  - uni-app
---

## 搭建uni-app项目

### 前情提要

微信开发者工具编写代码已经是过去式，不方便且语法已经老旧了，现在使用vsc来进行编码，使其转换成微信小程序代码，方便实时调试。

### 核心原理

微信开发者工具内置了 **文件监听（File Watcher）** 机制，会自动检测项目目录下的文件变化并触发重新编译。因此只需要用 VS Code 编辑源码，微信开发者工具就会自动刷新预览，**无需安装任何 VS Code 插件**。

uni-app 官方脚手架自带 `@dcloudio/types` 类型定义，VS Code 能直接识别 `uni` 全局对象，`uni.navigateTo`、`uni.request` 等所有 API 都有完整的智能补全和类型提示。

### 环境搭建

#### 1. 创建 uni-app 项目

```bash
# Vue3 + Vite + TypeScript 模板
npx degit dcloudio/uni-preset-vue#vite-ts my-mini-app
cd my-mini-app
pnpm install
```

> 如果不使用 TypeScript，可以用 `npx degit dcloudio/uni-preset-vue#vite my-mini-app`

#### 2. 启动微信小程序编译

```bash
pnpm dev:mp-weixin
```

执行后会在 `dist/dev/mp-weixin` 目录下持续输出小程序代码。Vite 保持 watch 模式，源文件一变化就自动重新编译。

#### 3. 微信开发者工具导入项目

1. 打开微信开发者工具 → **导入项目**
2. 目录选择：`项目根目录/dist/dev/mp-weixin`
3. 填写 AppID（可使用测试号）
4. 点击确定

之后微信开发者工具会自动监听该目录的文件变化，保存代码后 1-2 秒即可看到更新。

#### 4. 开启服务端口（可选，推荐）

微信开发者工具 → **设置** → **安全设置** → 打开 **服务端口**

开启后 uni-app CLI 可以自动唤起和刷新开发者工具，体验更好。

### 开发工作流

用 VS Code 打开 **uni-app 项目根目录**（不是 dist 目录），编辑 `src/` 下的 `.vue` 文件。

**整条链路**：

```
.vue 文件保存 → Vite 自动编译 → dist/dev/mp-weixin 产物更新 → 微信开发者工具自动刷新
```

建议在 VS Code 中开启自动保存：**File → Auto Save**，或设置 `files.autoSave: "afterDelay"`，这样连 Ctrl+S 都省了。

### uni API 使用示例

在 VS Code 中可以直接使用所有 uni API，拥有完整类型提示：

```vue
<script setup lang="ts">
import { onLoad } from "@dcloudio/uni-app";

// 页面加载
onLoad((options) => {
  console.log("页面参数:", options);
});

// 页面跳转
const goDetail = () => {
  uni.navigateTo({ url: "/pages/detail/detail?id=1" });
};

// 返回上一页
const goBack = () => {
  uni.navigateBack({ delta: 1 });
};

// 网络请求
const fetchData = () => {
  uni.request({
    url: "https://api.example.com/data",
    method: "GET",
    success(res) {
      console.log(res.data);
    },
    fail(err) {
      console.error(err);
    },
  });
};

// 本地存储
uni.setStorageSync("token", "abc123");
const token = uni.getStorageSync("token");

// 提示框
uni.showToast({ title: "操作成功", icon: "success" });
uni.showModal({
  title: "提示",
  content: "确定删除吗？",
  success(res) {
    if (res.confirm) {
      console.log("用户点击确定");
    }
  },
});

// 分享
onShareAppMessage(() => ({
  title: "分享标题",
  path: "/pages/index/index",
}));
</script>
```

```vue
<template>
  <view class="container">
    <text>uni-app + VS Code 开发</text>
    <button @click="goDetail">跳转详情</button>
    <button @click="fetchData">请求数据</button>
  </view>
</template>
```

### rpx 与 px 单位说明

小程序开发中推荐使用 **rpx**（responsive pixel）作为布局单位，`750rpx` 始终等于屏幕宽度，不同机型自动等比缩放。

| 单位  | 适用场景                         | 示例                                |
| ----- | -------------------------------- | ----------------------------------- |
| `rpx` | 静态布局尺寸（间距、宽高、字号） | `padding: 20rpx; font-size: 28rpx;` |
| `px`  | JS 动态计算值（API 返回的数值）  | `height: statusBarHeight + 'px'`    |

> API 如 `getSystemInfoSync()`、`getMenuButtonBoundingClientRect()` 返回的都是 **px** 值，绑定到 style 时直接用 px 即可，不要转 rpx。

### 常见问题

#### 修改了文件但微信开发者工具没反应？

1. 确认文件确实保存了（Ctrl+S）
2. 确认编辑的目录和微信开发者工具打开的 `dist/dev/mp-weixin` 是对应关系
3. 在微信开发者工具中手动点击 **编译** 按钮触发一次
4. 终端中 Vite 进程是否还在运行（`pnpm dev:mp-weixin`）

#### uni 的类型提示不生效？

手动安装类型包：

```bash
pnpm add -D @dcloudio/types
```

确保 `tsconfig.json` 中包含：

```json
{
  "compilerOptions": {
    "types": ["@dcloudio/types"]
  }
}
```

#### 需要装 VS Code 插件吗？

**不需要任何插件**就能开始开发。如果想提升体验，可以选装：

- `Vue - Official`（Volar）：Vue 模板语法高亮和智能提示
