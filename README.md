# The WebSocket handshake

## Client handshake request

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

만약, 헤더가 잘못됐거나 이해될 수 없는 것이라면 서버는 400 Bad Request 에러를 보낸다.  
만약 서버가 WebSocket의 버전을 이해하지 못한다면 `Sec-WebSocket-Version`을 사용하여 버전을 명시해서 보내줘야 한다.

## Server handshake response

서버가 클라이언트로부터 handshake request를 받았을 때, 서버는 다음과 같은 응답을 보낸다.

```HTTP
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

또한, 서버는 여기서 extension/sub-protocol을 정할 수 있다.

`Sec-WebSocket-Accept`는 클라이언트가 보낸 `Sec-WebSocket-Key`로부터 끌어낸다.  
`Sec-WebSocket-Key` 값과 magic string이라 불리는 `"258EAFA5-E914-47DA-95CA-C5AB0DC85B11"`를 이어 만든 `SHA-1 hash`의 결과값을 얻고, 이를 `base64`로 인코딩하여 만들어낸다.

서버가 이 응답을 보내면 그 때부터 handshake가 완료되고, 데이터를 주고받을 수 있게 된다.  
물론, 서버는 handshake에 응답하기 전에 다른 status code를 통해 인증, 리다이렉트 처리를 하거나 `Set-Cookie` 같은 헤더를 보낼 수 있다.

# Keeping track of clients

서버는 클라이언트의 소켓을 기억하고 있기 때문에, 이미 handshake를 완료한 상태라면 다시 할 필요가 없다.  
동일한 클라이언트 IP 주소는 여러 번 연결을 시도할 수 있다.  
하지만 너무 많은 연결을 시도하면 서버가 `Denial-of-Service attacks`로 부터 스스로를 지키기 위해 이를 거부할 수 있다.

# Exchanging data frames

# Socket.io

# Reference

- [WebSocket 프로토콜: 작동 방식에 대한 심층 분석 - AppMaster](https://appmaster.io/ko/blog/websocket-peurotokol-jagdong-bangsig)
