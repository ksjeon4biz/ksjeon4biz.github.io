---
layout: post
title:  "웹소켓"
date:   2024-08-12 22:20:00 +0900
categories: architecture
---

WebSocket은 웹 애플리케이션과 서버 간의 상호작용을 실시간으로 가능하게 해주는 통신 프로토콜입니다. HTTP와는 달리, WebSocket은 클라이언트와 서버 간의 지속적인 연결을 유지하며 양방향 통신을 지원합니다. 이를 통해 낮은 지연 시간과 효율적인 데이터 전송이 가능합니다.

### WebSocket의 주요 특징
1. #### 양방향 통신 (Bidirectional Communication):
클라이언트와 서버가 서로 데이터를 주고받을 수 있습니다. 이는 실시간 애플리케이션, 예를 들어 채팅 애플리케이션이나 실시간 게임에서 매우 유용합니다.

1. #### 실시간 데이터 전송 (Real-Time Data Transfer):
WebSocket 연결이 설정되면, 클라이언트와 서버는 실시간으로 데이터를 주고받을 수 있습니다. 이는 지연 시간이 매우 낮습니다.

1. #### 낮은 오버헤드 (Low Overhead):
HTTP 요청/응답 사이클의 오버헤드가 없기 때문에, 지속적인 연결을 유지하면서 데이터를 효율적으로 주고받을 수 있습니다.

1. #### 지속적인 연결 (Persistent Connection):
초기 핸드셰이크 이후, 연결이 끊어질 때까지 지속됩니다. 이는 상태 정보를 유지하고, 필요한 경우 즉시 데이터를 전송할 수 있게 합니다.

### WebSocket 프로토콜
WebSocket 프로토콜은 다음과 같이 작동합니다:

1. #### 핸드셰이크 (Handshake):
WebSocket 연결은 HTTP를 통해 초기화됩니다. 클라이언트는 서버에 HTTP 요청을 보내고, 서버는 WebSocket 연결을 설정하기 위한 응답을 보냅니다.

    ```http
    GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
    Sec-WebSocket-Version: 13
    ```

    서버의 응답:

    ```http
    HTTP/1.1 101 Switching Protocols
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
    ```

1. #### 데이터 프레임 (Data Frame) 전송:
WebSocket은 텍스트와 바이너리 데이터를 프레임으로 전송합니다. 각 프레임에는 데이터 타입, 길이, 기타 메타데이터가 포함됩니다.

1. #### 연결 종료 (Connection Close):
클라이언트와 서버는 언제든지 연결을 종료할 수 있습니다. 종료 시, 종료 프레임을 전송하여 연결을 안전하게 닫습니다.

### WebSocket의 활용 사례
1. #### 실시간 채팅 애플리케이션:
사용자가 메시지를 보내면, 메시지가 서버를 통해 즉시 다른 사용자에게 전달됩니다.

1. #### 실시간 데이터 피드:
주식 시세, 스포츠 경기 점수, 뉴스 업데이트 등 실시간으로 변화하는 데이터를 사용자에게 제공합니다.

1. #### 실시간 협업 도구:
여러 사용자가 동시에 문서, 코딩, 디자인 등을 협업할 수 있게 합니다.

1. #### 온라인 게임:
플레이어 간의 빠른 데이터 전송이 필요하며, WebSocket을 통해 실시간으로 상태를 동기화합니다.

### WebSocket을 사용하는 코드 예제
클라이언트 (JavaScript)

```javascript
const socket = new WebSocket('ws://server.example.com/socket');

socket.onopen = function(event) {
    console.log('WebSocket connection opened:', event);
    socket.send('Hello Server!');
};

socket.onmessage = function(event) {
    console.log('Message from server:', event.data);
};

socket.onclose = function(event) {
    console.log('WebSocket connection closed:', event);
};

socket.onerror = function(event) {
    console.error('WebSocket error observed:', event);
};
```

서버 (Node.js와 ws 라이브러리)

```javascript
const WebSocket = require('ws');

const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', function connection(ws) {
    ws.on('message', function incoming(message) {
        console.log('received:', message);
        ws.send(`Hello, you sent -> ${message}`);
    });

    ws.send('Welcome to the WebSocket server!');
});
```

### 결론
WebSocket은 클라이언트와 서버 간의 실시간, 양방향 통신을 가능하게 하는 강력한 프로토콜입니다. HTTP의 한계를 극복하고 실시간 애플리케이션의 요구사항을 충족시킬 수 있는 WebSocket을 사용하면, 보다 효율적이고 반응성이 뛰어난 웹 애플리케이션을 개발할 수 있습니다.


---
---


기존 소켓(Socket)과 WebSocket은 둘 다 네트워크 통신을 위한 프로토콜이지만, 각각의 목적과 작동 방식에는 중요한 차이점이 있습니다. 여기서는 주요 차이점을 설명하겠습니다.

### 1. 프로토콜 및 표준
#### 기존 소켓 (Socket):
* 일반적으로 TCP(Transmission Control Protocol) 또는 UDP(User Datagram Protocol) 기반으로 동작합니다.
* 낮은 수준의 네트워크 프로그래밍 인터페이스를 제공합니다.
* OSI 모델의 전송 계층(4계층)에서 작동합니다.
* 다양한 애플리케이션에서 사용되며, 프로토콜이나 포트 번호에 구애받지 않습니다.

#### WebSocket:
* HTTP(S)를 업그레이드하여 WebSocket 프로토콜로 변환하는 과정을 거칩니다.
* OSI 모델의 애플리케이션 계층(7계층)에서 작동합니다.
* RFC 6455에 정의된 표준 프로토콜입니다.
* 주로 웹 애플리케이션에서 실시간 통신을 위해 사용됩니다.

### 2. 연결 및 통신 방식
#### 기존 소켓 (Socket):
* 서버와 클라이언트 간의 직접적인 연결을 통해 데이터를 전송합니다.
* 연결 설정 후 데이터를 주고받을 때마다 명시적으로 처리해야 합니다.
* 일반적으로 단방향 또는 요청-응답 패턴을 따릅니다.
* 데이터 프레임의 구조나 형식은 애플리케이션 레벨에서 결정됩니다.

#### WebSocket:
* 초기에는 HTTP를 통해 핸드셰이크(handshake) 과정을 거쳐 연결이 설정됩니다.
* 핸드셰이크 이후에는 지속적인 양방향 통신이 가능해집니다.
* 연결이 유지되는 동안 클라이언트와 서버가 자유롭게 데이터를 주고받을 수 있습니다.
* 데이터는 프레임으로 전송되며, 프레임에는 메시지의 형식과 길이가 포함됩니다.

### 3. 사용 사례
#### 기존 소켓 (Socket):
* 실시간 게임, VoIP(Voice over IP), 비디오 스트리밍, 파일 전송 등 다양한 애플리케이션에 사용됩니다.
* 네트워크 프로그래밍에 대한 깊은 이해가 필요합니다.
* 다양한 프로토콜을 직접 구현하거나 사용해야 합니다.

#### WebSocket:
* 실시간 웹 애플리케이션(예: 채팅 애플리케이션, 실시간 데이터 스트리밍, 협업 도구)에서 주로 사용됩니다.
* 브라우저와 서버 간의 실시간 통신을 쉽게 구현할 수 있습니다.
* 웹 개발자가 쉽게 접근할 수 있도록 설계되었습니다.

### 4. 구현의 복잡성
#### 기존 소켓 (Socket):
* 소켓 프로그래밍은 상대적으로 복잡하며, 데이터의 직렬화/역직렬화, 네트워크 에러 처리 등을 명시적으로 다뤄야 합니다.
* C/C++, Java, Python 등의 언어에서 소켓 라이브러리를 사용하여 구현합니다.

#### WebSocket:
* 비교적 간단하게 구현할 수 있으며, 브라우저 내장 API와 서버 라이브러리를 사용하여 쉽게 설정할 수 있습니다.
* JavaScript에서 WebSocket 객체를 사용하여 간단하게 통신할 수 있습니다.

### 5. 브라우저 지원
#### 기존 소켓 (Socket):
* 브라우저에서 직접 지원되지 않으며, 브라우저와 서버 간의 통신은 HTTP/HTTPS를 통해 이루어집니다.
* 브라우저에서 소켓 기능을 사용하려면 WebSocket을 사용하거나, 브라우저 플러그인, 웹 워커 등을 사용해야 합니다.

#### WebSocket:
* 대부분의 현대적인 웹 브라우저에서 기본적으로 지원됩니다.
* 웹 표준의 일부로, 클라이언트-서버 간의 실시간 통신을 쉽게 구현할 수 있습니다.

### 결론
기존 소켓과 WebSocket은 모두 네트워크 통신을 위해 사용되지만, 그 사용 목적과 방식에는 차이가 있습니다. 기존 소켓은 낮은 수준의 네트워크 통신을 제공하며, 다양한 애플리케이션에서 사용됩니다. 반면 WebSocket은 웹 애플리케이션의 실시간 통신을 위해 설계되었으며, 높은 수준의 API를 제공하여 브라우저와 서버 간의 실시간 양방향 통신을 쉽게 구현할 수 있습니다. 두 프로토콜의 특성을 이해하고, 특정 요구사항에 맞는 프로토콜을 선택하는 것이 중요합니다.