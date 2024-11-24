# 동작 방식

## Hand Shake

`Hand Shake`란 클라이언트와 서버 간에 Web Socket 연결을 설정하는 초기 프로세스이다.  
HTTP 요청으로 시작하여 연결을 Web Socket으로 업그레이드한다.

이 때의 request header는 다음과 같다.  
다시 한번 말하지만, Web Socket 연결을 수립할 때는 HTTP 프로토콜을 사용한다.  
바로 떡하니 Web Socket으로 연결!이 아니다.

```HTTP
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

### `Upgrade`

현재 클라이언트, 서버, 전송 프로토콜 연결에서 다른 프로토콜로 업그레이드 또는 변경하기 위한 규칙

### `Connection`

Upgrade 헤더 필드가 명시되었을 경우, 송신자는 반드시 Upgrade 옵션을 지정한 Connection 헤더 필드도 전송해야한다.

### `Sec-WebSocket-Key`

WebSocket을 연결 수립을 위해 사용되는 request header.  
이 key 값은 보안을 제공하지는 않는다. 대신, non-WebSocket 클라이언트가 WebSocket 연결을 요청하는 것을 막아준다.  
이 헤더는 user agents에 의해 자동으로 추가된다. 따라서, `fetch()`나 `XMLHttpRequest.setRequestHeader()`를 사용해 임의로 추가할 수 없다.  
서버의 response header인 `Sec-WebSocket-Accept`는 반드시 이 key를 사용하여 computed 된 값을 포함해야한다.  
아래는 HTTP Response Header의 예시이다.

```HTTP
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### `Origin`

모든 browsers는 `Origin` 헤더를 보낸다.  
따라서, 이를 checking for same origin이나 automatically allowing or denying 등의 security 기능에 활용할 수 있다.(403 Forbidden을 보낸다던지)  
이는 `Cross Site WebSocket Hijacking(CSWH)`을 대비하는데 효과적이다.  
하지만 non-browser agents는 faked `Origin`을 보낼 수 있기 때문에 주의해야한다.

### `Sec-WebSocket-Protocol`

HTTP `Sec-WebSocket-Protocol` request and response header는 WebSocket Handshake에서 통신에 사용하기 위한 sub-protocol을 조정하는데 사용된다.

Request 시에, header는 web application이 사용하고자하는 하나 이상의 WebSocket sub-protocols를 명시한다.  
이들은 여러 headers에 protocol values로 추가될 수 있다.  
혹은 하나의 header에 comma(`,`) separate values로 추가할 수도 있다.

```HTTP
Sec-WebSocket-Protocol: <sub-protocols>
```

Response는 서버에 의해 선택된 sub-protocol을 명시한다.  
이는 반드시 request header에 명시된 sub-protocol list 중 첫 번째 것이어야 한다.

```HTTP
Sec-WebSocket-Protocol: <selected-sub-protocol>
```

Request header는 `WebSocket()`의 `protocols`에서 애플리케이션이 지정한 값을 사용하여 browser에 의해 자동으로 추가되고 채워진다.  
서버에서 선택한 하위 프로토콜은 `WebSocket.protocol`에서 웹 애플리케이션에 제공된다.

Request:

```HTTP
GET /chat HTTP/1.1
Host: example.com:8000
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Sec-WebSocket-Protocol: soap, wamp
```

Reponse:

```HTTP
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: soap
```

따라서, 이는 서버에서 여러 프로토콜 혹은 프로토콜 버전을 나눠서 서비스 할 경우 필요한 정보라고 할 수 있다.

### `Sec-WebSocket-Version`

즉, Socket.io를 활용한 통신은 정확한 표현이 아니다.
Socket.io를 활용한 WebSocket 통신이라고 해야한다.
