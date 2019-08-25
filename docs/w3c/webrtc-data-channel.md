?> 作者：小天同学 ，供稿日期：2019/08/23


## 为什么需要数据通道

在两个浏览器中，聊天、游戏、或是文件传输等需求发送信息是十分复杂的。通常情况下，我们需要建立一台服务器来转发数据，当然规模比较大的情况下，会扩展成多个数据中心。这种情况下很容易出现很高的延迟，同时难以保证数据的私密性。

这些问题可以通过WebRTC提供的`RTCDataChannel API`来解决，它能直接在点对点之间传输数据。

> 为了充分理解这篇文章，你可能需要去了解一些`RTCPeerConnection API`的相关知识，以及`STUN`，`TURN`、信道如何工作。强烈推荐[Getting Started With WebRTC](http://www.html5rocks.com/en/tutorials/webrtc/basics/)这篇文章。

我们已经有`WebSocket`、`AJAX`和`Server-Sent Events`了，为什么我们需要另外一个通信信道？WebSocket是全双工的，但这些技术的设计都是让浏览器与服务器之间进行通信。

`RTCDataChannel`则是一个完全不同的途径：

- 它通过`RTCPeerConnection API`，可以建立点对点互联。由于不需要中介服务器，中间的“跳数”减少，延迟更低。

- `RTCDataChannel`使用[SCTP](https://en.wikipedia.org/wiki/Stream_Control_Transmission_Protocol#Features)(Stream Control Transmission Protocol)协议，允许我们配置传递语义：我们可以配置包传输的顺序并提供重传时的一些配置。

> ```javascript
> // 配置示例
> var dataChannelOptions = {
>   ordered: false, //不保证到达顺序
>   maxRetransmitTime: 3000, //最大重传时间
> };
> /**
> * ordered: 数据通道是否保证按序传输数据
> * maxRetrasmitTime：在信息失败前的最大重传时间（强迫进入不可靠模式）
> * maxRetransmits：在信息失败前的最大重传次数（强迫进入不可靠模式）
> * negotiated：如果设为true，将移除对方的数据通道的自动设置，也就是说，将使用相同的id以自己配置的方式与对方建立数据通道
> * id：为数据通道提供一个自己定义的ID
> */
> ```

`RTCDataChannel API`支持灵活的数据类型。它的API是模仿`WebSocket`设计的，并且支持JavaScript中的二进制类型如`Blob`、`ArrayBuffer`和`ArrayBufferView`，另外还支持字符串。这些类型对于文件传输和多玩家的游戏来说意义重大。

`RTCDataChannel`在不可靠模式（类似于UDP）或可靠模式（类似于TCP）下都能够正常工作。

## 建立连接

当用户点击 "连接" 按钮,  `connectPeers()` 方法被调用。

> 尽管参与连接的两端都在同一页面，我们将启动连接的一端称为 "local" 端，另一端称为 "remote" 端。

```javascript
  function connectPeers() {
    // 建立该连接的 "local" 端，它是发起连接请求的一方
    localConnection = new RTCPeerConnection();
   
    // 创建RTCDataChannel并设置事件侦听以监视该数据通道
    sendChannel = localConnection.createDataChannel("sendChannel"); // 第二个参数就是options
    sendChannel.onopen = handleSendChannelStatusChange;
    sendChannel.onclose = handleSendChannelStatusChange;
    
    // 建立该连接的 "remote" 端，它是接收连接请求的一方，并设置事件侦听
    remoteConnection = new RTCPeerConnection();
    remoteConnection.ondatachannel = receiveChannelCallback;
    
    // 为每个连接建立 ICE 候选侦听处理， 当连接的一方出现新的 ICE 候选时该侦听逻辑将被调用以告知连接的另一方此消息
    localConnection.onicecandidate = e => !e.candidate || 
          remoteConnection.addIceCandidate(e.candidate).catch(handleAddCandidateError);

    remoteConnection.onicecandidate = e => !e.candidate || 
          localConnection.addIceCandidate(e.candidate).catch(handleAddCandidateError);
    
    // 创建一个连接offer
    localConnection.createOffer() 
    //创建 SDP (Session Description Protocol) 字节块用于描述我们期待建立的连接,该方法可选地接受一个描述连接限制的对象，例如连接是否必须支持音频、视频或者两者都支持。
    .then(offer => localConnection.setLocalDescription(offer)) 
    //将offer提供给local端,配置local端的连接
    .then(() => remoteConnection.setRemoteDescription(localConnection.localDescription)) 
    // 告知remote节点上述描述，将local 节点连接到到远程 
    .then(() => remoteConnection.createAnswer()) 
    //remote 节点予以回应，该方法生成一个SDP二进制块，用于描述 remote 节点愿意并且有能力建立的连接
    .then(answer => remoteConnection.setLocalDescription(answer)) 
    //该调用完成了remote端连接的建立
    .then(() => localConnection.setRemoteDescription(remoteConnection.localDescription))
    //本地连接的远端描述被设置为指向remote节点
    .catch(handleCreateDescriptionError);
  }
```

> 在现实场景，当参与连接的两节点运行于不同的上下文，建立连接的过程或稍微复杂些，每一次双方通过调用[`RTCPeerConnection.addIceCandidate()`](https://developer.mozilla.org/zh-CN/docs/Web/API/RTCPeerConnection/addIceCandidate)，提出连接方式的建议  (例如： UDP,、中继UDP 、 TCP之类的) ， 双方来回往复直到达成一致。本示例既然不涉及现实网络环境，因此我们假定双方接受首次连接建议。 



#### 数据通道（data channel）的连接

[`RTCPeerConnection`](https://developer.mozilla.org/zh-CN/docs/Web/API/RTCPeerConnection) 一旦open, 事件`datachannel` 被发送到远端以完成打开数据通道的处理， 该事件触发 `receiveChannelCallback()` 方法，如下所示：

```javascript
function receiveChannelCallback(event) {
    receiveChannel = event.channel;
    //每当remote节点接收到数据 handleReceiveMessage() 方法将被调用
    receiveChannel.onmessage = handleReceiveMessage;
  	//每当通道的连接状态发生改变 handleReceiveChannelStatusChange() 方法将被调用
    receiveChannel.onopen = handleReceiveChannelStatusChange;
    receiveChannel.onclose = handleReceiveChannelStatusChange;
  }
```



## 发送消息

当用户按下 "发送" 按钮，触发我们已建立的该按钮的 `click` 事件处理逻辑，在处理逻辑中调用sendMessage() 方法。

```javascript
function sendMessage() {
    var message = messageInputBox.value;
    sendChannel.send(message);
    
    messageInputBox.value = "";
    messageInputBox.focus();
  }
```



## 接收消息

当远程通道发生“message”事件时，我们的handleReceiveMessage（）方法被调用来处理事件。

```javascript
function handleReceiveMessage(event) {
    var el = document.createElement("p");
    var txtNode = document.createTextNode(event.data);
    
    el.appendChild(txtNode);
    receiveBox.appendChild(el);
  }
```



##  断开连接

当用户点击"Disconnect" 按钮。在之前我们设置的按钮事件处理逻辑中`disconnectPeers()`方法被调用。

```javascript
function disconnectPeers() {
 
    // Close the RTCDataChannels if they're open.
    
    sendChannel.close();
    receiveChannel.close();
    
    // Close the RTCPeerConnections
    
    localConnection.close();
    remoteConnection.close();

    sendChannel = null;
    receiveChannel = null;
    localConnection = null;
    remoteConnection = null;
    
    // Update user interface elements
    
    connectButton.disabled = false;
    disconnectButton.disabled = true;
    sendButton.disabled = true;
    
    messageInputBox.value = "";
    messageInputBox.disabled = true;
  }
```

该方法首先关闭每个节点的[`RTCDataChannel`](https://developer.mozilla.org/zh-CN/docs/Web/API/RTCDataChannel)，之后类似地关闭每个节点的 [`RTCPeerConnection`](https://developer.mozilla.org/zh-CN/docs/Web/API/RTCPeerConnection)。将所有对它们的指向置为`null` 以避免意外的复用。


## Demo

请访问[https://github.com/mdn/samples-server/tree/master/s/webrtc-simple-datachannel](https://github.com/mdn/samples-server/tree/master/s/webrtc-simple-datachannel)

