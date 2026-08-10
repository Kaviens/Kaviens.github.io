---
title: Electron基础示例
categories:
  - electron
tags:
  - electron
type: categories
comments: false
---

# Electron API 参考文档

本文档详细介绍了项目中使用的所有Electron API，包括其功能、参数、返回值和使用示例。

## 📋 目录

1. [BrowserWindow API](#browserwindow-api)
   - [new BrowserWindow(options)](#0-new-browserwindowoptions)
   - [窗口操作方法](#browserwindow-方法)
2. [Screen API](#screen-api)
3. [App API](#app-api)
4. [IpcMain API](#ipcmain-api)
5. [Tray API](#tray-api)
6. [Menu API](#menu-api)
7. [Dialog API](#dialog-api)
8. [事件监听 API](#事件监听-api)

---

## 🪟 BrowserWindow API

### 0. `new BrowserWindow(options)`

**功能**: 创建新的浏览器窗口

**参数**: `BrowserWindowConstructorOptions` - 窗口配置选项

**常用属性详解**:

```typescript
new BrowserWindow({
  // 窗口尺寸
  width: number,           // 窗口宽度（像素）
  height: number,          // 窗口高度（像素）
  minWidth?: number,       // 最小宽度
  minHeight?: number,      // 最小高度
  maxWidth?: number,       // 最大宽度
  maxHeight?: number,      // 最大高度

  // 窗口位置
  x?: number,              // 窗口左上角X坐标
  y?: number,              // 窗口左上角Y坐标
  center?: boolean,        // 是否居中显示

  // 窗口显示
  show?: boolean,          // 是否立即显示（默认true）
  title?: string,          // 窗口标题
  icon?: string | NativeImage, // 窗口图标

  // 窗口样式
  frame?: boolean,         // 是否显示窗口边框（默认true）
  titleBarStyle?: string,  // 标题栏样式（macOS）
  transparent?: boolean,   // 是否透明（默认false）
  backgroundColor?: string, // 背景颜色
  opacity?: number,        // 窗口透明度（0.0-1.0）

  // 窗口行为
  resizable?: boolean,     // 是否可调整大小（默认true）
  movable?: boolean,       // 是否可移动（默认true）
  minimizable?: boolean,   // 是否可最小化（默认true）
  maximizable?: boolean,   // 是否可最大化（默认true）
  closable?: boolean,      // 是否可关闭（默认true）
  alwaysOnTop?: boolean,   // 是否始终置顶（默认false）
  skipTaskbar?: boolean,   // 是否在任务栏显示（默认false）

  // 全屏相关
  fullscreen?: boolean,    // 是否全屏（默认false）
  fullscreenable?: boolean, // 是否允许全屏（默认true）

  // 窗口状态
  autoHideMenuBar?: boolean, // 自动隐藏菜单栏（默认false）
  menuBarVisible?: boolean,  // 菜单栏是否可见

  // 安全设置
  webSecurity?: boolean,   // 是否启用web安全（默认true）
  allowRunningInsecureContent?: boolean, // 允许运行不安全内容
  experimentalFeatures?: boolean, // 启用实验性功能

  // 其他
  hasShadow?: boolean,     // 是否显示窗口阴影（默认true）
  enableLargerThanScreen?: boolean, // 允许窗口大于屏幕
  webPreferences?: WebPreferences // Web内容偏好设置
})
```

**项目中的使用示例**:

```javascript
mainWindow = new BrowserWindow({
  width: Math.round(width * 0.9), // 宽度为屏幕的90%
  height: 100, // 初始高度100px
  show: false, // 先不显示
  frame: false, // 无边框窗口
  transparent: true, // 透明背景
  backgroundColor: '#00000000', // 透明背景色
  autoHideMenuBar: true, // 隐藏菜单栏
  fullscreen: false, // 非全屏
  skipTaskbar: true, // 不在任务栏显示
  alwaysOnTop: true, // 始终置顶
  hasShadow: false, // 禁用窗口阴影
  enableLargerThanScreen: false, // 禁止窗口大于屏幕
  movable: true, // 可移动
  resizable: false, // 不可调整大小
  webPreferences: {
    preload: join(__dirname, '../preload/index.js'),
    sandbox: false,
    webSecurity: false,
    allowRunningInsecureContent: true,
    experimentalFeatures: true，
    backgroundThrottling: false
  }
})
```

**WebPreferences 详细配置**:

```typescript
webPreferences: {
  preload?: string,                  // 预加载脚本路径
  nodeIntegration?: boolean,         // 是否启用Node.js集成
  contextIsolation?: boolean,        // 是否启用上下文隔离
  sandbox?: boolean,                 // 是否启用沙箱模式
  webSecurity?: boolean,             // 是否启用web安全
  allowRunningInsecureContent?: boolean, // 允许运行不安全内容
  experimentalFeatures?: boolean,    // 启用实验性功能
  devTools?: boolean,                // 是否显示开发者工具
  webgl?: boolean,                   // 是否启用WebGL
  plugins?: boolean,                 // 是否启用插件
  images?: boolean,                  // 是否加载图片
  textAreasAreResizable?: boolean,   // 文本区域是否可调整大小
  webgl2?: boolean,                  // 是否启用WebGL2
  offscreen?: boolean,               // 是否启用离屏渲染
  backgroundThrottling?: boolean,    // 后台节流
  spellcheck?: boolean,              // 拼写检查
  enableWebSQL?: boolean,            // 启用WebSQL
  enableRemoteModule?: boolean,      // 启用远程模块
  additionalArguments?: string[],    // 额外的命令行参数
  safeDialogs?: boolean,             // 安全对话框
  disableDialogs?: boolean,          // 禁用对话框
  navigateOnDragDrop?: boolean,      // 拖拽导航
  autoplayPolicy?: string,           // 自动播放策略
  disableHtmlFullscreenWindowResize?: boolean, // 禁用HTML全屏窗口调整
  accessibleTitle?: string,          // 辅助功能标题
  spellcheckerLanguage?: string,     // 拼写检查语言
  autoHideCursor?: boolean,          // 自动隐藏光标
  titleBarOverlay?: boolean | TitleBarOverlay, // 标题栏覆盖
  trafficLightPosition?: Point,      // 交通灯位置（macOS）
  roundedCorners?: boolean,          // 圆角（macOS）
  vibrancy?: string,                 // 活力效果（macOS）
  visualEffectState?: string,        // 视觉效果状态（macOS）
  titleBarStyle?: string,            // 标题栏样式（macOS）
  hiddenInMissionControl?: boolean,  // 在任务控制中隐藏（macOS）
  acceptFirstMouse?: boolean,        // 接受首次鼠标（macOS）
  disableAutoHideCursor?: boolean,   // 禁用自动隐藏光标（macOS）
  tabbingIdentifier?: string,        // 标签标识符（macOS）
  webContents?: WebContents,         // Web内容对象
  session?: Session,                 // 会话对象
  partition?: string,                // 分区
  affinity?: string,                 // 亲和性
  zoomFactor?: number,               // 缩放因子
  zoomLevel?: number,                // 缩放级别
  preloadURL?: string,               // 预加载URL
  preloadFile?: string,              // 预加载文件
  preloadErrorCallback?: (error: Error) => void, // 预加载错误回调
  preloadScripts?: string[],         // 预加载脚本数组
  additionalArguments?: string[],    // 额外参数
  safeDialogs?: boolean,             // 安全对话框
  disableDialogs?: boolean,          // 禁用对话框
  navigateOnDragDrop?: boolean,      // 拖拽导航
  autoplayPolicy?: string,           // 自动播放策略
  disableHtmlFullscreenWindowResize?: boolean, // 禁用HTML全屏窗口调整
  accessibleTitle?: string,          // 辅助功能标题
  spellcheckerLanguage?: string,     // 拼写检查语言
  autoHideCursor?: boolean,          // 自动隐藏光标
  titleBarOverlay?: boolean | TitleBarOverlay, // 标题栏覆盖
  trafficLightPosition?: Point,      // 交通灯位置（macOS）
  roundedCorners?: boolean,          // 圆角（macOS）
  vibrancy?: string,                 // 活力效果（macOS）
  visualEffectState?: string,        // 视觉效果状态（macOS）
  titleBarStyle?: string,            // 标题栏样式（macOS）
  hiddenInMissionControl?: boolean,  // 在任务控制中隐藏（macOS）
  acceptFirstMouse?: boolean,        // 接受首次鼠标（macOS）
  disableAutoHideCursor?: boolean,   // 禁用自动隐藏光标（macOS）
  tabbingIdentifier?: string,        // 标签标识符（macOS）
  webContents?: WebContents,         // Web内容对象
  session?: Session,                 // 会话对象
  partition?: string,                // 分区
  affinity?: string,                 // 亲和性
  zoomFactor?: number,               // 缩放因子
  zoomLevel?: number,                // 缩放级别
  preloadURL?: string,               // 预加载URL
  preloadFile?: string,              // 预加载文件
  preloadErrorCallback?: (error: Error) => void, // 预加载错误回调
  preloadScripts?: string[],         // 预加载脚本数组
}
```

**常用属性说明**:

- **`width/height`**: 设置窗口初始尺寸
- **`show: false`**: 创建窗口但不立即显示，等准备好后再显示
- **`frame: false`**: 创建无边框窗口，适合自定义标题栏
- **`transparent: true`**: 启用透明背景，配合 `backgroundColor: '#00000000'`
- **`skipTaskbar: true`**: 窗口不在任务栏显示，适合后台应用
- **`alwaysOnTop: true`**: 窗口始终置顶，适合字幕显示
- **`resizable: false`**: 禁止用户调整窗口大小
- **`webSecurity: false`**: 禁用web安全，允许跨域请求
- **`preload`**: 指定预加载脚本，用于主进程和渲染进程通信

### 1. `mainWindow.getBounds()`

**功能**: 获取窗口的边界信息（位置和尺寸）

**返回值**:

```typescript
Rectangle {
  x: number,      // 窗口左上角X坐标
  y: number,      // 窗口左上角Y坐标
  width: number,  // 窗口宽度
  height: number  // 窗口高度
}
```

**使用示例**:

```javascript
const bounds = mainWindow.getBounds();
console.log(`窗口位置: (${bounds.x}, ${bounds.y})`);
console.log(`窗口尺寸: ${bounds.width}x${bounds.height}`);
```

**项目中的使用**:

- 保存窗口原始尺寸
- 计算窗口位置变化
- 停靠功能的位置计算

### 2. `mainWindow.setBounds(options)`

**功能**: 设置窗口的边界（位置和尺寸）

**参数**:

```typescript
options: {
  x?: number,      // 窗口左上角X坐标
  y?: number,      // 窗口左上角Y坐标
  width?: number,  // 窗口宽度
  height?: number  // 窗口高度
}
```

**使用示例**:

```javascript
// 设置窗口位置和尺寸
mainWindow.setBounds({
  x: 100,
  y: 100,
  width: 800,
  height: 600,
});

// 只设置高度
mainWindow.setBounds({
  height: 300,
});
```

**项目中的使用**:

- 窗口停靠到顶部/底部
- 动态调整窗口高度
- 窗口拖拽后的位置恢复

### 3. `mainWindow.getPosition()`

**功能**: 获取窗口的当前位置

**返回值**: `[number, number]` - [x, y] 坐标数组

**使用示例**:

```javascript
const [winX, winY] = mainWindow.getPosition();
console.log(`窗口位置: (${winX}, ${winY})`);
```

### 4. `mainWindow.show()`

**功能**: 显示窗口

**使用示例**:

```javascript
mainWindow.show();
```

**项目中的使用**:

- 托盘菜单显示窗口
- 窗口初始化完成后显示
- 从最小化状态恢复

### 5. `mainWindow.hide()`

**功能**: 隐藏窗口

**使用示例**:

```javascript
mainWindow.hide();
```

**项目中的使用**:

- 托盘菜单隐藏窗口
- 窗口关闭时隐藏而不是退出

### 6. `mainWindow.close()`

**功能**: 关闭窗口

**使用示例**:

```javascript
mainWindow.close();
```

### 7. `mainWindow.focus()`

**功能**: 聚焦窗口（使窗口成为活动窗口）

**使用示例**:

```javascript
mainWindow.focus();
```

### 8. `mainWindow.reload()`

**功能**: 重新加载窗口内容

**使用示例**:

```javascript
mainWindow.reload();
```

### 9. `mainWindow.isVisible()`

**功能**: 检查窗口是否可见

**返回值**: `boolean`

**使用示例**:

```javascript
if (mainWindow.isVisible()) {
  mainWindow.hide();
} else {
  mainWindow.show();
}
```

### 10. `mainWindow.isMinimized()`

**功能**: 检查窗口是否最小化

**返回值**: `boolean`

**使用示例**:

```javascript
if (mainWindow.isMinimized()) {
  mainWindow.restore();
}
```

### 11. `mainWindow.restore()`

**功能**: 从最小化状态恢复窗口

**使用示例**:

```javascript
mainWindow.restore();
```

### 12. `mainWindow.setMovable(boolean)`

**功能**: 设置窗口是否可移动

**参数**: `boolean` - true允许移动，false禁止移动

**使用示例**:

```javascript
mainWindow.setMovable(false); // 禁止移动
mainWindow.setMovable(true); // 允许移动
```

### 13. `mainWindow.setResizable(boolean)`

**功能**: 设置窗口是否可调整大小

**参数**: `boolean` - true允许调整，false禁止调整

**使用示例**:

```javascript
mainWindow.setResizable(false); // 禁止调整大小
mainWindow.setResizable(true); // 允许调整大小
```

### 14. `mainWindow.setFullScreen(boolean)`

**功能**: 设置窗口全屏状态

**参数**: `boolean` - true全屏，false退出全屏

**使用示例**:

```javascript
mainWindow.setFullScreen(true); // 进入全屏
mainWindow.setFullScreen(false); // 退出全屏
```

### 15. `mainWindow.loadURL(url)`

**功能**: 加载指定URL

**参数**: `string` - 要加载的URL

**使用示例**:

```javascript
mainWindow.loadURL("https://example.com");
```

### 16. `mainWindow.loadFile(path)`

**功能**: 加载本地HTML文件

**参数**: `string` - 文件路径

**使用示例**:

```javascript
mainWindow.loadFile(join(__dirname, "../renderer/index.html"));
```

### 17. `mainWindow.webContents`

**功能**: 访问网页内容相关API

**常用方法**:

- `webContents.openDevTools()` - 打开开发者工具
- `webContents.send(channel, ...args)` - 向渲染进程发送消息
- `webContents.once(event, listener)` - 监听一次性事件
- `webContents.setWindowOpenHandler(handler)` - 设置窗口打开处理器

**使用示例**:

```javascript
// 打开开发者工具
mainWindow.webContents.openDevTools();

// 发送消息到渲染进程
mainWindow.webContents.send("snap-status", true);

// 监听页面加载完成
mainWindow.webContents.once("did-finish-load", () => {
  console.log("页面加载完成");
});
```

---

## 🖥️ Screen API

### 1. `screen.getPrimaryDisplay()`

**功能**: 获取主显示器信息

**返回值**:

```typescript
Display {
  id: number,
  bounds: Rectangle,     // 显示器边界
  workArea: Rectangle,   // 工作区域（排除任务栏等）
  scaleFactor: number,   // 缩放因子
  rotation: number,      // 旋转角度
  touchSupport: string,  // 触摸支持
  monochrome: boolean,   // 是否单色
  accelerometerSupport: string,
  colorSpace: string,
  colorDepth: number,
  depthPerComponent: number,
  displayFrequency: number,
  size: Size,           // 显示器尺寸
  workAreaSize: Size    // 工作区域尺寸
}
```

**使用示例**:

```javascript
const primaryDisplay = screen.getPrimaryDisplay();
const { width, height } = primaryDisplay.workAreaSize;
console.log(`主显示器工作区域: ${width}x${height}`);
```

**项目中的使用**:

- 获取屏幕宽度设置窗口初始大小
- 获取工作区域计算停靠位置

### 2. `screen.getAllDisplays()`

**功能**: 获取所有显示器信息

**返回值**: `Display[]` - 显示器数组

**使用示例**:

```javascript
const displays = screen.getAllDisplays();
displays.forEach((display, index) => {
  console.log(
    `显示器 ${index}: ${display.bounds.width}x${display.bounds.height}`,
  );
});
```

**项目中的使用**:

- 多屏幕环境下查找指定显示器
- 窗口跨屏移动时的显示器切换

### 3. `screen.getDisplayNearestPoint(point)`

**功能**: 获取最接近指定点的显示器

**参数**:

```typescript
point: {
  x: number,
  y: number
}
```

**返回值**: `Display`

**使用示例**:

```javascript
const windowCenter = {
  x: winX + windowWidth / 2,
  y: winY + windowHeight / 2,
};
const currentScreen = screen.getDisplayNearestPoint(windowCenter);
```

**项目中的使用**:

- 确定窗口当前所在的屏幕
- 多屏幕环境下的窗口停靠

### 4. `screen.getCursorScreenPoint()`

**功能**: 获取鼠标光标的屏幕坐标

**返回值**: `Point` - {x: number, y: number}

**使用示例**:

```javascript
const cursorPos = screen.getCursorScreenPoint();
console.log(`鼠标位置: (${cursorPos.x}, ${cursorPos.y})`);
```

**项目中的使用**:

- 窗口拖拽时跟随鼠标位置
- 取消停靠时的窗口位置计算

---

## 🚀 App API

### 1. `app.whenReady()`

**功能**: 当Electron初始化完成时触发

**返回值**: `Promise<void>`

**使用示例**:

```javascript
app.whenReady().then(() => {
  createWindow();
  createTray();
});
```

### 2. `app.quit()`

**功能**: 退出应用程序

**使用示例**:

```javascript
app.quit();
```

### 3. `app.on(event, listener)`

**功能**: 监听应用程序事件

**常用事件**:

- `activate` - 应用被激活（macOS）
- `window-all-closed` - 所有窗口关闭
- `browser-window-created` - 创建浏览器窗口
- `open-url` - 打开URL协议
- `second-instance` - 第二个实例启动

**使用示例**:

```javascript
app.on("window-all-closed", () => {
  if (process.platform !== "darwin") {
    app.quit();
  }
});

app.on("activate", () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow();
  }
});
```

### 4. `app.getAppPath()`

**功能**: 获取应用程序路径

**返回值**: `string`

**使用示例**:

```javascript
const appPath = app.getAppPath();
const iconPath = join(appPath, "resources/icon.png");
```

### 5. `app.setAsDefaultProtocolClient(protocol)`

**功能**: 设置应用程序为默认协议客户端，//往注册表里面填写，但是只有打开软件的时候才会往注册表注册

**参数**: `string` - 协议名称

**使用示例**:

```javascript
app.setAsDefaultProtocolClient("ketang-clients");
```

### 6. `app.requestSingleInstanceLock()`

**功能**: 请求单实例锁

**返回值**: `boolean` - 是否成功获取锁

**使用示例**:

```javascript
const gotTheLock = app.requestSingleInstanceLock();
if (!gotTheLock) {
  app.quit();
}
```

---

## 📡 IpcMain API

### 1. `ipcMain.on(channel, listener)`

**功能**: 监听渲染进程发送的消息

**参数**:

- `channel: string` - 消息频道
- `listener: (event, ...args) => void` - 监听器函数

**使用示例**:

```javascript
ipcMain.on("set-height", (_event, newHeight) => {
  if (mainWindow) {
    mainWindow.setBounds({
      height: newHeight.data,
    });
  }
});
```

**项目中的使用场景**:

- `set-initial-position` - 设置窗口初始位置
- `set-height` - 设置窗口高度
- `set-window-visibility` - 设置窗口显示/隐藏
- `close-window` - 关闭窗口
- `set-full-screen` - 设置全屏状态
- `refresh-main-window` - 刷新主窗口
- `show-warning-dialog` - 显示警告对话框

### 2. `ipcMain.handle(channel, listener)`

**功能**: 处理渲染进程的异步请求

**参数**:

- `channel: string` - 消息频道
- `listener: (event, ...args) => Promise<any>` - 处理器函数

**使用示例**:

```javascript
ipcMain.handle("load-settings", () => {
  return store.get("userSettings");
});
```

---

## 🗂️ Tray API

### 1. `new Tray(iconPath)`

**功能**: 创建系统托盘图标

**参数**: `string` - 图标文件路径

**使用示例**:

```javascript
const tray = new Tray(iconPath);
```

### 2. `tray.setContextMenu(menu)`

**功能**: 设置托盘右键菜单

**参数**: `Menu` - 菜单对象

**使用示例**:

```javascript
const contextMenu = Menu.buildFromTemplate(menuTemplate);
tray.setContextMenu(contextMenu);
```

### 3. `tray.destroy()`

**功能**: 销毁托盘图标

**使用示例**:

```javascript
if (tray) {
  tray.destroy();
  tray = null;
}
```

---

## 📋 Menu API

### 1. `Menu.buildFromTemplate(template)`

**功能**: 从模板创建菜单

**参数**: `MenuItemConstructorOptions[]` - 菜单项数组

**返回值**: `Menu`

**使用示例**:

```javascript
const menuTemplate = [
  {
    label: "显示/隐藏",
    click: () => {
      if (mainWindow.isVisible()) {
        mainWindow.hide();
      } else {
        mainWindow.show();
      }
    },
  },
  {
    label: "行数模式",
    submenu: [
      {
        label: "单行",
        click: () => setLineMode(1),
      },
      {
        label: "双行",
        click: () => setLineMode(2),
      },
    ],
  },
];

const contextMenu = Menu.buildFromTemplate(menuTemplate);
```

### 2. `MenuItemConstructorOptions`

**功能**: 菜单项配置选项

**属性**:

```typescript
{
  label?: string,           // 菜单项标签
  click?: () => void,       // 点击事件处理器
  type?: 'normal' | 'separator' | 'submenu' | 'checkbox' | 'radio',
  submenu?: MenuItemConstructorOptions[], // 子菜单
  enabled?: boolean,        // 是否启用
  visible?: boolean,        // 是否可见
  checked?: boolean,        // 是否选中（checkbox/radio类型）
  accelerator?: string,     // 快捷键
  icon?: string | NativeImage // 图标
}
```

---

## 💬 Dialog API

### 1. `dialog.showMessageBox(options)`

**功能**: 显示消息对话框

**参数**:

```typescript
options: {
  type?: 'none' | 'info' | 'error' | 'question' | 'warning',
  buttons?: string[],       // 按钮数组
  defaultId?: number,       // 默认按钮索引
  title?: string,          // 对话框标题
  message?: string,        // 主要消息
  detail?: string,         // 详细消息
  icon?: NativeImage,      // 图标
  cancelId?: number,       // 取消按钮索引
  noLink?: boolean,        // 按钮是否为链接样式
  normalizeAccessKeys?: boolean
}
```

**使用示例**:

```javascript
dialog.showMessageBox({
  type: "info",
  title: "温馨提示",
  message: "这是一个提示消息",
  buttons: ["OK"],
});
```

**项目中的使用**:

- 显示警告提示
- 用户确认操作

---

## 🎧 事件监听 API

### BrowserWindow 事件

#### 1. `ready-to-show`

**功能**: 窗口准备显示时触发

**使用示例**:

```javascript
mainWindow.on("ready-to-show", () => {
  originalBounds = mainWindow.getBounds();
  // 设置初始位置等操作
});
```

**项目中的使用**:

- 保存窗口初始尺寸
- 设置窗口默认停靠位置

#### 2. `move`

**功能**: 窗口移动时触发

**使用示例**:

```javascript
mainWindow.on("move", () => {
  const [winX, winY] = mainWindow.getPosition();
  // 处理窗口位置变化
});
```

**项目中的使用**:

- 实现窗口自动停靠功能
- 检测窗口是否需要吸附到屏幕边缘

#### 3. `close`

**功能**: 窗口关闭时触发

**使用示例**:

```javascript
mainWindow.on("close", (event) => {
  event.preventDefault(); // 阻止关闭
  mainWindow.hide(); // 隐藏窗口
});
```

#### 4. `show` / `hide`

**功能**: 窗口显示/隐藏时触发

**使用示例**:

```javascript
mainWindow.on("show", () => {
  console.log("窗口显示");
});

mainWindow.on("hide", () => {
  console.log("窗口隐藏");
});
```

#### 5. `enter-full-screen` / `leave-full-screen`

**功能**: 进入/退出全屏时触发

**使用示例**:

```javascript
mainWindow.on("enter-full-screen", () => {
  console.log("进入全屏");
});

mainWindow.on("leave-full-screen", () => {
  console.log("退出全屏");
});
```

### WebContents 事件

#### 1. `did-finish-load`

**功能**: 页面加载完成时触发

**使用示例**:

```javascript
mainWindow.webContents.once("did-finish-load", () => {
  console.log("页面加载完成");
});
```

---

## 🔧 坐标系统说明

### 屏幕坐标系

- **原点**: 屏幕左上角 (0, 0)
- **X轴**: 从左到右递增
- **Y轴**: 从上到下递增

### 窗口坐标计算

```javascript
// 窗口边界
const bounds = {
  x: 100, // 窗口左上角X坐标
  y: 50, // 窗口左上角Y坐标
  width: 800, // 窗口宽度
  height: 600, // 窗口高度
};

// 窗口右下角坐标
const rightBottom = {
  x: bounds.x + bounds.width, // 900
  y: bounds.y + bounds.height, // 650
};

// 窗口中心坐标
const center = {
  x: bounds.x + bounds.width / 2, // 500
  y: bounds.y + bounds.height / 2, // 350
};
```

### 多屏幕环境

```javascript
// 获取所有显示器
const displays = screen.getAllDisplays();

// 主显示器
const primaryDisplay = screen.getPrimaryDisplay();

// 根据窗口位置找到所在屏幕
const windowCenter = {
  x: bounds.x + bounds.width / 2,
  y: bounds.y + bounds.height / 2,
};
const currentScreen = screen.getDisplayNearestPoint(windowCenter);
```

---

## 📝 使用建议

1. **窗口边界操作**: 使用`getBounds()`获取当前状态，`setBounds()`设置新状态
2. **屏幕适配**: 使用`screen.getPrimaryDisplay()`获取屏幕信息，适配不同分辨率
3. **事件监听**: 合理使用`on()`持续监听和`once()`单次监听
4. **多屏幕支持**: 使用`getDisplayNearestPoint()`处理多屏幕环境
5. **进程通信**: 使用`ipcMain.on()`处理渲染进程消息
6. **托盘功能**: 使用`Tray`和`Menu`创建系统托盘交互

---

## 🔗 相关链接

- [Electron 官方文档](https://www.electronjs.org/docs)
- [BrowserWindow API](https://www.electronjs.org/docs/api/browser-window)
- [Screen API](https://www.electronjs.org/docs/api/screen)
- [App API](https://www.electronjs.org/docs/api/app)
- [IpcMain API](https://www.electronjs.org/docs/api/ipc-main)
