---
title: socket封装
categories:
  - vue
---

## 封装socket

```html
socket如何封装呢 const ws = new WebSocket('ws://example.com/socket');
ws分别有四个事件：onopen 、onmessage 、onclose 、onerror 。
代表着连接成功、收到消息、断开连接和发生错误，当某个事件发生时，就会触发对应的回调函数。
同时心跳机制是必须的，不然服务器会认为客户端已经断开连接。 那我们就开始封装吧
```

```vue
const defaultServer =
  import.meta.env.VITE_SERVER_IP !== ""
    ? `ws://${import.meta.env.VITE_SERVER_IP}/ws/audio`
    : `ws://${location.host}/ws/audio`// 默认的服务器地址

创建一个类
class Socket {
    private ws: WebSocket | null = null; // 私有属性，用于存储WebSocket实例,实例不能被外部访问
    private _url: string; // 私有属性，用于存储WebSocket的url
    private _heartbeatTimer: ReturnType<typeof setInterval> | null = null// 心跳定时器
      // 后端超过多久没响应，就断开连接 */
    private _heartbeatInterval: number
    // 短线重连时常
    private _reconnectInterval: number
      // 如果主动关闭的则不重连
    private _closeForce = false
      // 锁：是否正在等待心跳应答
    private _isWaitingAck: boolean
      // 后端返回超时时间
    private _heartbeatIntervalTime: number | NodeJS.Timeout | undefined
    // 后端返回超时时间
    private _heartbeatIntervalTime: number | NodeJS.Timeout | undefined
    //类的构造函数

    constructor(options?: AudioSocketOptions) {//外部new的时候可以传入参数，不传就使用默认值

    this._url = options?.url || defaultServer
    this._heartbeatInterval = options?.heartbeatInterval ?? 3000
    this._reconnectInterval = options?.reconnectInterval ?? 3000
    this._isWaitingAck = false
      /** 当前连接状态 */
  get readyState(): SocketReadyState {
    return (this._ws?.readyState ?? SocketReadyState.CLOSED) as SocketReadyState
  }
    /**
   * 建立连接
   */
    connect() {
    if (this._ws && this._ws.readyState !== WebSocket.CLOSED) {
      console.warn("[AudioSocket] 连接已存在或正在连接中")
      return
    }
    this._closeForce = false
    this._ws = new WebSocket(this._url || defaultServer)
    // this._ws.binaryType = "arraybuffer"

    this._ws.onopen = this._onOpen.bind(this)// 连接成功回调

    this._ws.onmessage = this._onMessage.bind(this) // 收到消息回调

    this._ws.onerror = this._onError.bind(this)// 发生错误回调
    this._ws.onclose = this._onClose.bind(this)// 断开连接回调
  }
   /**
   * 发送数据
   * @param data 要发送的数据，支持 string / ArrayBuffer / Blob / object
   */
  send(data: string | ArrayBuffer | Blob) {
    if (!this._ws || this._ws.readyState !== WebSocket.OPEN) {
      console.warn("[AudioSocket] 连接未就绪，无法发送数据")
      return
    }
    this._ws.send(data)
  }
   /**
   * 建立心跳（IAP 二进制封包）
   * 每次心跳前检查上一次是否收到后端响应，连续未响应超过阈值则主动断开触发重连
   */
   //说下思路，当socket连接成功时，会触发这个方法，首先会先把_heartbeatTimer原本的定时器去除掉，因为假设后面又重新连接了，之前的定时器还在的话就会有问题
清除之后，建立一个新的，当查看是否有连接，知道连接后进入查看锁，这个锁的作用是，假设为true的状态下，就是之前已经发过心跳信息给后端了，等待后端回复，在这期间
都不会进行下一步动作，后续就是向后端发送心跳信息，然后设置一个定时器，如果在6秒内没有收到回复的话，就断开连接，并且清除心跳定时器和锁设置为false，以供后续socket重连。
  heartbeat() {
    this._clearHeartbeat()
    this._heartbeatTimer = setInterval(() => {
      if (!this._ws || this._ws.readyState !== WebSocket.OPEN) {
        return
      }
      if (this._isWaitingAck) return
      const heartBeatMsg = {
        strCmdEnum: "CMD_HEARTBEAT_REQ",
        Mac: "aa:aa:aa:00:00:01",
        Type: 1
      }
      const iapBody = jsonToArrayBuffer(heartBeatMsg)
      const iapHead = getIapHead(iapBody.byteLength)
      const iapBuffer = _concatBuffer(iapHead, iapBody)
      this.send(iapBuffer)
      this._isWaitingAck = true // ← 新增：加锁
      this._heartbeatIntervalTime = setTimeout(() => {
        // ← 新增：超时断开
        if (this._isWaitingAck) {
          console.warn("[AudioSocket] 后端未响应心跳，断开连接")
          this._ws?.close()
          this._clearHeartbeat()
          this._isWaitingAck = false
        }
      }, 6000)
      this._emit("heartbeatSend")
    }, this._heartbeatInterval)
  }

  //接下来，是多功能监听方法
    /**
   * 监听事件
   * @param event 事件名称
   * @param callback 回调函数
   */
   首先，一个vue组件一开始以on监听，例如监听一个就是叫message，其实on只是把监听后执行的那个方法push到一个map对象里面而已，然后再把这个整体的一个方法数组给set进去，当_emits触发时，例如是后端发送前端socket信息，这时候，接受信息的_onMessage就会触发这个emits方法，并告诉他是message的，然后emits方法就会从map数组里面找到message的方法数组，然后进行遍历，给每个方法传数据
  

  on(event: string, callback: EventCallback) {
    const callbacks = this._eventMap.get(event) || []
    if (!callbacks.some((fn) => fn === callback)) {
      callbacks.push(callback)
    }
    this._eventMap.set(event, callbacks)
  }

  /**
   * 取消监听事件
   * @param event 事件名称
   * @param callback 回调函数
   */
  off(event: string, callback: EventCallback) {
    const callbacks = this._eventMap.get(event)
    if (!callbacks) return
    const index = callbacks.indexOf(callback)
    if (index !== -1) {
      callbacks.splice(index, 1)
    }
  }
    /**
   * 触发事件
   */
  private _emit(event: string, ...args: any[]) {
    const callbacks = this._eventMap.get(event)
    if (callbacks) {
      callbacks.forEach((cb) => cb(...args))
    }
  }



    /**
   * 主动关闭连接
   */
  close() {
    this._closeForce = true
    this._clearHeartbeat()
    this._ws?.close()
    this._ws = null
  }

  /**
   * 销毁实例，关闭连接并移除所有监听
   */
  destroy() {
    clearInterval(this._timeout)
    this._timeout = undefined
    this.close()
    this._eventMap.clear()
  }

  /** 连接成功 */
  private _onOpen() {
    // 不再重置 _currentReconnectTimes，否则重连成功后又归零，导致无限重连
    this._emit("open")
    if (this._heartbeatEnabled) {
      this.heartbeat()
    }
  }

  /** 收到消息 */
  private _onMessage(ev: MessageEvent) {
    this._emit("message", ev.data)
    if (ev) {
      // 收到心跳应答，清除超时、释放锁
      clearTimeout(this._heartbeatIntervalTime)
      this._isWaitingAck = false
      return
    }
  }

  /** 连接错误 */
  private _onError(ev: Event) {
    this._emit("error", ev)
  }

  /** 连接关闭 */
  private _onClose(ev: CloseEvent) {
    this._clearHeartbeat()
    this._ws = null
    this._emit("close", ev)
    if (this._closeForce) return
    setTimeout(() => {
      this.connect()
    }, this._reconnectInterval)
  }

  /** 清除心跳 */
  private _clearHeartbeat() {
    if (this._heartbeatTimer) {
      clearInterval(this._heartbeatTimer)
      this._heartbeatTimer = null
    }
  }
}
```
