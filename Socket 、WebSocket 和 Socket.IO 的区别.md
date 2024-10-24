# Socket 、WebSocket 和 Socket.IO 的区别
- Socket 是为了方便使用 TCP，UDP 而抽象出来的一层 API 接口（Java 里也提供了遵循 Socket 理念的 API 供开发者使用），Socket 算是传输控制层的接口，Socket 通信是通过 Socket 套接字来实现的
- WebSocket 是一种双向通信协议，基于 TCP 协议，算是应用层的协议，作为 Html5 的一部分，WebSocket 通信是通过 HTTP 的握手过程实现的
- Socket.IO 是基于 Node.js 的框架，不过还有 Java 等客户端的实现，它可以是基于 WebSocket 协议进行的上层封装（主要是形成一致的通用的实时通信接口，可以用 WebSocket、Polling 轮询和其他通信方式实现），而且在 WebSocket 不可用的情况下，能够提供长轮询作为备选方式处理数据（比如当前环境不支持 WebSocket 时，能够自动选择最佳的方式来实现网络实时通信）
