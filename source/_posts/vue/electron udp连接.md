---
title: electron udp连接
categories:
  - vue
---
```vue
import dgram from 'dgram'   // 引入dgram模块(Node.js 原生模块，用于UDP通信)
const server = dgram.createSocket('udp4')// 创建UDP套接字
//bind失败后触发error
server.on('error', (err) => {
    log.error('[UDP] Server error:', err.message)
    mainWindow?.webContents.send('udp-status', { type: 'error', message: err.message })
    // Retry after 3 seconds
    setTimeout(() => {
      log.info('[UDP] Retrying bind...')
      server.bind(8805)
    }, 3000)
  })

//bind成功后触发listening事件
  server.on('listening', () => {
    const addr = server.address()
    const msg = `[UDP] Server listening on ${addr.address}:${addr.port}`
    log.info(msg)
    mainWindow?.webContents.send('udp-status', { type: 'listening', message: msg })
  })


// 收到 UDP 数据包：匹配 IP → 发送渲染进程预构建好的二进制包
server.on('message', (_msg, rinfo) => {
  if (!mainWindow) return
  const sourceIp = rinfo.address
  console.log(`source.address: ${sourceIp}`)
  log.info(`source.address: ${sourceIp}`)
  mainWindow.webContents.send('source.address', sourceIp)
  // 查找该 IP 对应的预构建响应包
  const responseData = responseMap.get(sourceIp)
  console.log(`responseData: ${responseData}`)
  if (!responseData) {
    return
  }
  // 直接发送渲染进程构建好的二进制数据
  const buf = Buffer.from(responseData)
// 发送数据包到客户端,第一个参数是要发送的数据，第二个和第三个分别是端口号和IP地址。最后一个回调函数会在数据包被成功发送后执行

  server.send(buf, rinfo.port, rinfo.address, (err) => {
    if (err) {
      console.error('回包失败:', err)
    } else {
      console.log(`响应已发送至 ${sourceIp}:${rinfo.port}${firstResponseSent}`)
      log.info(`setBuf:${sourceIp}:${rinfo.port}`)
      // 首次回包成功，通知渲染进程初始化音频
      if (!firstResponseSent) {
        firstResponseSent = true
        mainWindow?.webContents.send('first-response-success')
        console.log('[主进程] 首次回包成功，已通知渲染进程初始化音频')
      }
    }
  })
})
  server.bind(8805)

```