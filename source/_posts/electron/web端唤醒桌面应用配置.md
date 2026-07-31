---
title: web端唤醒桌面应用配置
categories:
  - electron
tags:
  - electron
type: categories
comments: false
---

## electron主进程配置

### 1.自定义注册协议

```js
// Windows和mac的协议注册
if (process.defaultApp) {
  // 检查是否在开发环境下运行（通过electron命令运行）
  if (process.argv.length >= 2) {
    // 检查是否有足够的命令行参数
    app.setAsDefaultProtocolClient("ketang-clients", process.execPath, [
      join(__dirname, process.argv[1]),
    ]);
    // 注册协议处理器，包含执行路径和脚本路径
  }
} else {
  //但是不要用这个，因为后面我们用了nsh
  app.setAsDefaultProtocolClient("ketang-clients");
  // 生产环境下直接注册协议处理器
}

//注意ketang-clients是和电脑注册表，electron-build打包工具绑定在一起的，下面代码会讲
```

## 2.处理web点击按钮首次唤醒桌面应用参数处理

### window系统

```
// 处理首次启动时的协议参数
if (process.platform === 'win32') {
  // 检查当前运行平台是否为Windows系统
  const args = process.argv
  // 获取启动时传入的命令行参数数组
  if (args.length > 1) {
    // 检查是否有额外的启动参数（第一个参数是执行文件路径）
    const protocolUrl = args[args.length - 1]
    // 获取最后一个参数，通常是协议URL（如：ketang-clients://xxx?xxx）

    const params = parseCustomUrl(protocolUrl)
    // 调用解析函数，从协议URL中提取参数
    const result = {
      ipSegments: params.ipSegments!.split('.'), // "192.168.1.1"
      // 将IP地址字符串按点分割成数组
      roomId: params.roomId // "3"
      // 获取房间ID
    }

    store.set('userSettings', result)
    // 将解析的参数保存到electron-store中

    mainWindow?.webContents.once('did-finish-load', () => {
      // 监听主窗口渲染进程加载完成事件（只执行一次）
      mainWindow?.webContents.send('awaken', protocolUrl)
      // 向渲染进程发送'awaken'消息，传递原始协议URL
    })
  }
```

### mac系统

```
if (process.platform === 'darwin') {
  // macOS
  app.on('open-url', (event, url) => {  // 去掉下划线，使用url参数
    event.preventDefault()

    console.log('macOS收到协议URL:', url)  // 调试用

    // 解析参数
    const params = parseCustomUrl(url)
    if (params.ipSegments && params.roomId) {
      const result = {
        ipSegments: params.ipSegments.split('.'),
        roomId: params.roomId
      }

      // 保存参数
      store.set('userSettings', result)
      console.log('保存参数:', result)
    }

    // 处理窗口显示
    if (mainWindow) {
      if (mainWindow.isMinimized()) mainWindow.restore()
      mainWindow.show()
      mainWindow.focus()

      // 向渲染进程发送参数
      if (mainWindow.webContents.isLoading()) {
        mainWindow.webContents.once('did-finish-load', () => {
          mainWindow?.webContents.send('awaken', url)
        })
      } else {
        mainWindow.webContents.send('awaken', url)
      }
    }
  })
}




//个人项目处理参数方法parseCustomUrl（不同需求对应不同）
function parseCustomUrl(url: string) {
  const queryPart = url.split('?')[1] || ''
  const params = new URLSearchParams(queryPart)

  return {
    ipSegments: params.get('ipSegments'),
    roomId: params.get('roomId')
  }
}
```

## 单实例锁定和第二个实例处理（防止开启两个桌面应用）

```
// 处理第二个实例
const gotTheLock = app.requestSingleInstanceLock()

if (!gotTheLock) {
  app.quit()
} else {
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  app.on('second-instance', (_event) => {
    // 有人试图运行第二个实例，我们应该关注我们的窗口
    if (mainWindow) {
      if (mainWindow.isMinimized()) mainWindow.restore()
      // 获取最后一个参数，通常包含协议URL
      if (is.dev) {
        // 开发环境
        mainWindow.reload()
      } else {
        // 生产环境
        // 1. 先记住当前窗口的位置和大小
        const bounds = mainWindow.getBounds()

        // 2. 重新加载页面
        mainWindow
          .loadFile(join(__dirname, '../renderer/index.html'))
          .then(() => {
            // 3. 恢复窗口位置和大小
            mainWindow?.setBounds(bounds)
            // 4. 确保窗口可见
            if (!mainWindow?.isVisible()) {
              mainWindow?.show()
            }
          })
          .catch((err) => {
            console.error('重新加载页面失败:', err)
          })
      }
      mainWindow.show()
      mainWindow.focus()
    }
  })
}
```

## electron-builder.yml配置(根据不同需求配置不同)

### 1.全局配置

```
//告诉electron-builder我要注册这个协议，同时electron-builder会自动写入window注册表，mac要额外配置
protocols:
  - name: ketang-clients-protocol
    schemes:
      - ketang-clients
```

### mac配置

```
//mac配置
mac:
  target:
    - target: dmg
   //安装包名称
  artifactName: ketang-clients-mac.${ext}
  //额外配置注册
  protocols:
    - name: ketang-clients-protocol
      schemes:
        - ketang-clients
  extendInfo:
  //告诉 macOS 系统："嘿，当用户点击 ketang-clients://xxx 这种链接时，请启动我这个应用！
    CFBundleURLTypes:
      - CFBundleURLSchemes:
          - ketang-clients
        CFBundleURLName: com.ketang-clients.url-scheme
        //权限申请的理由书,当你的应用想用摄像头时，系统会弹窗问用户
    NSCameraUsageDescription: Application requests access to the device's camera.
    //文件夹访问权限
    NSMicrophoneUsageDescription: Application requests access to the device's microphone.
    NSDocumentsFolderUsageDescription: Application requests access to the user's Documents folder.
    NSDownloadsFolderUsageDescription: Application requests access to the user's Downloads folder.
  entitlementsInherit: build/entitlements.mac.plist
  notarize: false
```

### Windows 安装程序的配置

```
nsis:
  # NSIS 安装程序配置
  include: installer.nsh
  # 包含自定义的安装脚本文件，可以添加额外的安装步骤

  artifactName: ketang-clients-win-x64-setup.${ext}
  # 生成的安装包文件名，${ext} 会被替换为 .exe
  # 例如：ketang-clients-win-x64-setup.exe

  shortcutName: 语音转写字幕客户端
  # 开始菜单和桌面快捷方式的名称

  uninstallDisplayName: ${productName}
  # 在控制面板"程序和功能"中显示的名称
  # ${productName} 会被替换为 "语音转写字幕客户端"

  createDesktopShortcut: true
  # 是否在桌面创建快捷方式

  createStartMenuShortcut: true
  # 是否在开始菜单创建快捷方式

  oneClick: false
  # 是否使用一键安装模式
  # false = 显示安装向导，让用户选择安装选项
  # true = 直接安装，不显示选项

  allowToChangeInstallationDirectory: true
  # 是否允许用户修改安装目录
  # true = 用户可以选择安装位置
  # false = 使用默认位置
```

## 解决桌面应用下载后点击web点击按钮无反应问题和处理卸载后依旧在桌面的问题

```
!macro customInstall
  ; 注册 ketang-clients 协议到当前用户
  WriteRegStr HKCU "Software\Classes\ketang-clients" "" "智能课堂系统字幕客户端"
  WriteRegStr HKCU "Software\Classes\ketang-clients" "URL Protocol" ""
    //my-app.exe这代表着要关联的应用程序路径，也是设置的核心要和electron-builder.yml里面的executableName一致
  WriteRegStr HKCU "Software\Classes\ketang-clients\DefaultIcon" "" "$INSTDIR\\my-app.exe,1"
  WriteRegStr HKCU "Software\Classes\ketang-clients\shell" "" ""
  WriteRegStr HKCU "Software\Classes\ketang-clients\shell\open" "" ""
  //my-app.exe这代表着要关联的应用程序路径，也是设置的核心要和electron-builder.yml里面的executableName一致
  WriteRegStr HKCU "Software\Classes\ketang-clients\shell\open\command" "" '"$INSTDIR\\my-app.exe" "%1"'

  ; 注册到 HKCR 以防某些系统环境
  WriteRegStr HKCR "ketang-clients" "" "智能课堂系统字幕客户端"
  WriteRegStr HKCR "ketang-clients" "URL Protocol" ""
  WriteRegStr HKCR "ketang-clients\\DefaultIcon" "" "$INSTDIR\\my-app.exe,1"
  WriteRegStr HKCR "ketang-clients\\shell" "" ""
  WriteRegStr HKCR "ketang-clients\\shell\\open" "" ""
  WriteRegStr HKCR "ketang-clients\\shell\\open\\command" "" '"$INSTDIR\\my-app.exe" "%1"'
!macroend

!macro customUnInstall
  ; 删除注册表项
  DeleteRegKey HKCU "Software\\Classes\\ketang-clients"
  DeleteRegKey HKCR "ketang-clients"
    //my-app.exe这代表着要关联的应用程序路径，也是设置的核心要和electron-builder.yml里面的executableName一致
  ; 强制结束进程
  nsExec::ExecToLog 'taskkill /f /im "my-app.exe"'

  ; 删除启动项
  DeleteRegValue HKCU "Software\\Microsoft\\Windows\\CurrentVersion\\Run" "智能课堂系统字幕客户端"

  ; 删除桌面快捷方式
  Delete "$DESKTOP\\智能课堂系统字幕客户端.lnk"

  ; 删除开始菜单快捷方式
  RMDir /r "$SMPROGRAMS\\智能课堂系统字幕客户端"
!macroend

```

## web端使用

```
 window.location.href = "ceshi-clients://launch?ipSegments=114.17.19.113&roomId=1&BrowserId=313135&SeqId=213251"
```
