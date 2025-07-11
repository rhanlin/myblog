+++
title = "WebRTC 101"
description = ""
date = 2023-10-20 19:00:00
author = "Rhan0"
tags=["webRTC"]
weight= 2
+++

WebRTC (Web Real-Time Communication) 是一個支援網頁瀏覽器進行即時語音對話或影像的API，有許多廣泛用途，例如 Zoom, google meet… 這類型視訊會議的實作，由兩項核心技術構成：

### 1. Media Capture and Streams API

就是 [MediaStreams API](https://developer.mozilla.org/en-US/docs/Web/API/Media_Capture_and_Streams_API)，可以藉由這個 API 擷取本地端的多媒體串流(e.g video, audio)，顯示在瀏覽器上或是進行更多處理/傳送。

### 2. **Peer-To-Peer Connection(P2P)**

點對點連線，意指用戶端(client)電腦之間能夠互相交換資料，無需透過主機端(server)來處理各用戶端的資料傳輸。

一般來說，我們最熟悉的網路溝通模式(HTTP超文本傳輸協定)如下圖：![webrtc-1](https://github.com/rhanlin/myblog/assets/54620681/9f7d09b0-3023-428c-bbfe-74fe6cc0b396)

P2P的溝通方式為，各自先連線到 server 確認彼此連線後，client A 跟 client B 就能彼此傳輸資料，如下圖：

![webrtc-2](https://github.com/rhanlin/myblog/assets/54620681/2496bc2e-51d4-44b8-a814-6e9ad421f0a4)

## WebRTC 三種架構 P2P, SFU, MCU

![webrtc-3](https://github.com/rhanlin/myblog/assets/54620681/155b797f-2235-4d5d-806a-7b05cb26d8ae)
圖片擷取自: [liveswitch](https://developer.liveswitch.io/liveswitch-server/index.html)

**P2P** 多個終端之間兩兩進行連接，形成一個網狀結構，每一端直接向另一端發送媒體或從另一端接收媒體，隨著參與方數量的增加，會難以管理

**MCU** 從各端點接收媒體，然後將其混合後一起廣播給所有其他端點

**SFU** 和 MCU不同的地方在於不會把媒體訊號混流，收到某個終端共享的音視頻流後，就直接將該音視頻流轉發給房間內的其他終端，通常用於有兩個以上的連線(e.g 多人會議)

## WebRTC 幾個核心的 API

- **getUserMedia( )**
    
    擷取 user 的 audio, video，使用時需傳入 `{audio: boolean, video: boolean}`
    
    ```javascript
    const constraints = { audio: true };
    navigator.mediaDevices.getUserMedia(constraints)
    ```
    
- **MediaRecorder( )**
    
    能夠存取 `MediaStream`，用來錄製多媒體的數據，例如可以搭配 `getUserMedia` 進行錄音
    
    ```javascript
    const mediaRecorder = new MediaRecorder(stream);
    ```
    
- **RTCPeerConnection( )**
    
    WebRTC 建立溝通最核心的功能，提供建立、維持連線、監控狀態、加密、關閉連線等方法
    
- **RTCDataChannel( )**
    
    建立雙方使用者之間的數據傳輸通道，可以接收任意數據(如文字、圖片等)，可以想像成類似 webSocket API 但不需要經過 Server
    

## MediaStream API

### 1. getUserMedia( )
[實作一: 取得用戶裝置](https://codepen.io/rhanlin/pen/WNWqXGg)

### 2. MediaStream Object

`getUserMedia` 之後 return `MediaStream`

- MediaStream.active : 顯示該stream的活動狀態。
- MediaStream.id : 一串uniq ID。
- Event handler:
    - onaddtrack: 當使用`addTrack` method，加入新的 `MediaStreamTrack` object時會觸發該事件
    - onremovetrack: 當使用`removeTrack` method，移除 `MediaStreamTrack` object時會觸發該事件

### 3. MediaStreamTrack Object

`MediaStream.getTracks()` return `MediaStreamTrack`

每個 `MediaStream` 包含了數個 `MediaStreamTrack`，分別是來自不同的影音輸入裝置;每個 `MediaStreamTrack` 則可能包含數個 channel ( e.g 左右聲道 )

## **RTCPeerConnection 實作 P2P**

![webrtc-5](https://github.com/rhanlin/myblog/assets/54620681/42d32e3e-e7e4-447b-bd5b-2f2ca4a3acf2)

圖片擷取自: [medium](https://medium.com/@jaysee.zhang/webrtc-for-dummies-395e07c90562)

[實作二: PeerA 單方面傳送媒體到 PeerB](https://codepen.io/rhanlin/pen/PogrOmQ)

### **Event handlers / methods**

1. 初始化：build RTCPeerConnection instance ( 初始化 )

```javascript
const pc = new RTCPeerConnection();
pc.onicecandidate = (e) => {};
pc.oniceconnectionstatechange = (e) => {};
pc.ontrack = (e) => {};
```

初始化時綁定的幾個事件：

- onicecandidate: 當查找到相對應的遠端端口時會做發送此事件，進行網路資訊的共享
- oniceconnectionstatechange: 每次 ICE 連切狀態變化時，都會向對方發送此事件，常用在需要觸發 ICE 重啟時(failed)
- ontrack: 完成連線後，透過該事件能夠在遠端傳輸多媒體檔案時觸發，並處理接收
- 延伸 ICE (STUN/TURN): 是為了解決網路 [NAT](https://en.wikipedia.org/wiki/Network_address_translation) 的複雜性，ICE 會嘗試找到連接對方的最佳途徑
![webrtc-4](https://github.com/rhanlin/myblog/assets/54620681/257c5f87-a93b-4ea8-a96c-744ae1fcefba)


1. 加入多媒體數據：add MediaStreams

```javascript
const stream = await navigator.mediaDevices.getUserMedia({
  audio: true,
  video: true,
});
// ....Skip
localStream.getTracks().forEach(track => localPeer.addTrack(track, localStream));
// ...Skip
remotePeer.addEventListener('track', gotRemoteStream);
function gotRemoteStream(e) {
  if (remoteVideo.srcObject !== e.streams[0]) {
    remoteVideo.srcObject = e.streams[0];
  }
}
```

### **SDP offer/answer (handshake)**

想要進行連線之前，必須要先處理兩個問題：

1. 必須要先知道要如何與對方連線
2. 必須了解彼此支援哪些媒體格式

使用  **[Session Description Protocol](https://en.wikipedia.org/wiki/Session_Description_Protocol) (**SDP**) 的 offer 和 answer 來交換多媒體數據資訊以及連線資訊，確認彼此身份**

- SDP more…
    1. 各端所支援的影音編解碼器
    2. 編解碼所設定的參數
    3. 所使用的傳輸協議
    4. ICE連接候選項等

```javascript
// ...
const offerDesc = await localPeer.createOffer();
try {
  await localPeer.setLocalDescription(offerDesc);

	// send offer to remote endpoint(Signaling Server)
  await remotePeer.setRemoteDescription(offerDesc);  
	// remote endpoint get the local pc offer(SDP)
  const answerDesc = await remotePeer.createAnswer();
  await remotePeer.setLocalDescription(answerDesc);

	// send answer to remote endpoint(Signaling Server)
  await localPeer.setRemoteDescription(answerDesc);
} catch (e) {
  // ... catch error
}
```

舉例來說就好像是雙方打電話時，彼此會先確認是否有聽到聲音之後，才開始進行這段通話

### WebRTC 關鍵字整理

- P2P, SFU, MCU
- WebRTC 架構
- Answer & Offer(SDP)
- MediaStream API
- Stream & Track
- WebRTC 幾個核心的 API

### 更深入研究...

- ICE (STUN/TURN)
- Signaling Server
- RTCDataChannel

參考資料：

- [Architecture](https://webrtc.github.io/webrtc-org/architecture/#)
- [Real time communication with WebRTC  |  Google Codelabs](https://codelabs.developers.google.com/codelabs/webrtc-web#0)
- [https://webrtc.github.io/samples/src/content/peerconnection/pc1](https://webrtc.github.io/samples/src/content/peerconnection/pc1/)
