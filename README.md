## mcj-uni-socket.io

[![NPM version][npm-image]][npm-url]
[![npm download][download-image]][download-url]

[npm-image]: https://img.shields.io/npm/v/mcj-uni-socket.io.svg?style=flat-square
[npm-url]: https://www.npmjs.com/package/mcj-uni-socket.io
[download-image]: https://img.shields.io/npm/dm/mcj-uni-socket.io.svg?style=flat-square
[download-url]: https://www.npmjs.com/package/mcj-uni-socket.io

重写 socket.io-client 的 engin.io-client 处理件，h5 依旧使用原生 WebSocket，APP 与小程序使用 uni-app 的 WebSocket 协议，所以 h5 端任然可以支持长轮询等方式，APP 与小程序只能支持 WebSocket 协议。

### 安装

```
// 建议使用npm或yarn包形式引入以保证插件的更新迭代
npm i mcj-uni-socket.io --save
// yarn add mcj-uni-socket.io
```

#### 版本选择

使用本插件，请注意插件版本与服务端 socket.io **版本要匹配**，否则将会出现无法连接服务器的问题。

| uni-socket 版本 | 服务端 socket.io 版本 |
| --------------- | --------------------- |
| 1.x             | 2.x                   |
| 2.x             | 3.x                   |
| 3.x             | 4.x                   |

### 使用

```
import io from 'mcj-uni-socket.io';

const socket = io('your websocket path', {
  query: {},
  transports: [ 'websocket', 'polling' ],
  timeout: 5000,
});

socket.on('connect', () => {
  // ws连接已建立，此时可以进行socket.io的事件监听或者数据发送操作
  // 连接建立后，本插件的功能已完成，接下来的操作参考socket.io官方客户端文档即可
  console.log('ws 已连接');
  // socket.io 唯一连接id，可以监控这个id实现点对点通讯
  const { id } = socket;
  socket.on(id, (message) => {
    // 收到服务器推送的消息，可以跟进自身业务进行操作
    console.log('ws 收到服务器消息：', message);
  });
  // 主动向服务器发送数据
  socket.emit('send_data', {
    time: +new Date(),
  });
});

socket.on('error', (msg: any) => {
  console.log('ws error', msg);
});
```

更多使用方法，请参考[socket.io-client](https://github.com/socketio/socket.io-client)写法即可。

### API

参考[官网 API](https://socket.io/docs/client-api/)

### 常见问题

1. 为什么没有聊天室示例代码？

   本项目仅仅是将 socket.io 封装到 uni-app 使用，并非完整的聊天室。

2. Exception: ReferenceError: Can't find variable: window

   hbuilder x 2.6.3 版本中 v3 编译有 bug，升级 hbuilder x 即可。

3. 真机运行 TypeError: undefined is not an object (evaluating 'document.createElement')？  
   示例代码中：

   ```
   const socket = io('your websocket path', {
     query: {},
     transports: [ 'websocket', 'polling' ],
     timeout: 5000,
   });
   ```

   不要漏写`transports: [ 'websocket', 'polling' ]`，如果没有指定协议，貌似 socket.io 会默认走`JSONP Polling`请求，导致报错。

4. 无法连接服务端？
   如果是真机调试，请确保设备与服务端在同一个局域网内，此外注意，Socket.io 以及升级到 3.x，请注意版本匹配。Socket.io 3.x 请使用 mcj-uni-socket 2.x，Socket.io 2.x 请使用 uni-socket 1.x。

5. 报错：Converting circular stricture to JSON
  错误的意思是，无法序列化一个循环引用的对象，大概率出现原因是，尝试使用JSON.stringify()去序列化`socket`对象。
  使用vue的`mixin`功能并将socket对象挂载到mixin的`data`上，vue内部可能会自动去序列化这个`mixin`的`data`从而导致出错。
  使用vuex直接挂载socket对象同样会导致该问题，请勿将socket对象挂载到vuex中。

  以下是mixin的错误示例，请勿使用这种方式初始化socket.io：
  ```
  // page.vue
  <script>
    import {socketioMixin} from "../../mixins/socketio";
    export default {
      data() {
        return { }
      },
      mixins: [socketioMixin],
    }
  </script>
  ```
  ```
  // socket mixin
  import io from 'mcj-uni-socket.io';
  export const socketioMixin = {
    data() {
      return {
        socket: null,
      }
    },
    created() {
      this.init()
    },
    methods: {
      init() {
        const option = {
          query: {io : this.uuid},
          transports: ['websocket', 'polling'],
          timeout: 5000,
        }
        this.socket = io(SOCKET_API, option);	  
      },
    },
  }