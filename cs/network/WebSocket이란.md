# WebSocket이란?

> WebSocket은 **실시간 양방향 통신**을 위한 프로토콜입니다.    
> HTTP의 요청-응답 방식과 달리, **연결을 유지하면서 서버와 클라이언트가 자유롭게 데이터를 주고받을 수 있습니다.**  

---

## 1. WebSocket이란?

WebSocket은 **HTML5 표준**의 일부로, **TCP 기반의 양방향 통신 프로토콜**입니다.

### 주요 특징

- **실시간 통신**: 서버에서 클라이언트로 즉시 데이터 전송 가능
- **양방향 통신**: 클라이언트 ↔ 서버 양방향 데이터 전송
- **연결 유지**: 한 번 연결하면 계속 유지 (HTTP와 달리)
- **효율적**: HTTP 헤더 오버헤드 없음
- **표준 프로토콜**: RFC 6455로 표준화됨

---

## 2. 언제 사용하는가?

### WebSocket을 사용하는 경우

- **실시간 채팅**: 카카오톡, 슬랙, 디스코드
- **실시간 알림**: 푸시 알림, 주식 시세
- **실시간 게임**: 멀티플레이어 게임
- **실시간 협업**: 구글 문서, 피그마
- **IoT 데이터**: 센서 데이터 실시간 전송
- **라이브 스트리밍**: 실시간 댓글, 좋아요

### HTTP vs WebSocket 비교

| 항목 | HTTP | WebSocket |
|------|------|-----------|
| 통신 방식 | 요청-응답 | 양방향 |
| 연결 | 매번 새로 연결 | 연결 유지 |
| 실시간성 | 낮음 (폴링 필요) | 높음 |
| 오버헤드 | 높음 (헤더 포함) | 낮음 |
| 사용 예 | API 호출, 파일 다운로드 | 채팅, 실시간 알림 |

---

## 3. WebSocket 원리

### 3-1. 연결 과정 (Handshake)

1. **HTTP Upgrade 요청**
   ```
   GET /chat HTTP/1.1
   Host: server.example.com
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
   Sec-WebSocket-Version: 13
   ```

2. **서버 응답**
   ```
   HTTP/1.1 101 Switching Protocols
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
   ```

3. **WebSocket 연결 완료**
   - HTTP 연결이 WebSocket 프로토콜로 업그레이드
   - 이후부터는 WebSocket 프레임으로 통신

---

## 4. Android에서 WebSocket 사용하기

### 4-1. OkHttp WebSocket 사용법

```kotlin
import okhttp3.*
import okio.ByteString
import java.util.concurrent.TimeUnit

class WebSocketManager {
    private var webSocket: WebSocket? = null
    private val client = OkHttpClient.Builder()
        .readTimeout(30, TimeUnit.SECONDS)
        .build()

    fun connect(url: String) {
        val request = Request.Builder()
            .url(url)
            .build()

        webSocket = client.newWebSocket(request, object : WebSocketListener() {
            override fun onOpen(webSocket: WebSocket, response: Response) {
                println("WebSocket 연결 성공")
                // 연결 성공 후 처리
            }

            override fun onMessage(webSocket: WebSocket, text: String) {
                println("텍스트 메시지 수신: $text")
                // 메시지 처리
            }

            override fun onMessage(webSocket: WebSocket, bytes: ByteString) {
                println("바이너리 메시지 수신: ${bytes.hex()}")
                // 바이너리 메시지 처리
            }

            override fun onClosing(webSocket: WebSocket, code: Int, reason: String) {
                println("WebSocket 연결 종료 중: $code / $reason")
            }

            override fun onClosed(webSocket: WebSocket, code: Int, reason: String) {
                println("WebSocket 연결 종료됨: $code / $reason")
            }

            override fun onFailure(webSocket: WebSocket, t: Throwable, response: Response?) {
                println("WebSocket 연결 실패: ${t.message}")
                // 재연결 로직
            }
        })
    }

    fun sendMessage(message: String) {
        webSocket?.send(message)
    }

    fun close() {
        webSocket?.close(1000, "정상 종료")
    }
}
```

### 4-2. ViewModel에서 WebSocket 사용

```kotlin
class ChatViewModel : ViewModel() {
    private val _messages = MutableLiveData<List<ChatMessage>>()
    val messages: LiveData<List<ChatMessage>> = _messages

    private val webSocketManager = WebSocketManager()
    private val messageList = mutableListOf<ChatMessage>()

    fun connectToChat(roomId: String) {
        val url = "wss://your-server.com/chat/$roomId"
        webSocketManager.connect(url)
    }

    fun sendMessage(text: String) {
        val message = ChatMessage(
            id = System.currentTimeMillis(),
            text = text,
            isFromMe = true,
            timestamp = System.currentTimeMillis()
        )
        
        messageList.add(message)
        _messages.value = messageList.toList()
        
        // WebSocket으로 메시지 전송
        webSocketManager.sendMessage(text)
    }

    override fun onCleared() {
        super.onCleared()
        webSocketManager.close()
    }
}

data class ChatMessage(
    val id: Long,
    val text: String,
    val isFromMe: Boolean,
    val timestamp: Long
)
```

### 4-3. Activity에서 사용

```kotlin
class ChatActivity : AppCompatActivity() {
    private lateinit var binding: ActivityChatBinding
    private val viewModel: ChatViewModel by viewModels()
    private val adapter = ChatAdapter()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityChatBinding.inflate(layoutInflater)
        setContentView(binding.root)

        setupRecyclerView()
        setupObservers()
        setupSendButton()

        // 채팅방 연결
        val roomId = intent.getStringExtra("room_id") ?: ""
        viewModel.connectToChat(roomId)
    }

    private fun setupRecyclerView() {
        binding.recyclerView.adapter = adapter
        binding.recyclerView.layoutManager = LinearLayoutManager(this)
    }

    private fun setupObservers() {
        viewModel.messages.observe(this) { messages ->
            adapter.submitList(messages)
            binding.recyclerView.scrollToPosition(messages.size - 1)
        }
    }

    private fun setupSendButton() {
        binding.sendButton.setOnClickListener {
            val message = binding.messageInput.text.toString()
            if (message.isNotEmpty()) {
                viewModel.sendMessage(message)
                binding.messageInput.text.clear()
            }
        }
    }
}
```

---

## 5. WebSocket vs HTTP Long Polling

### HTTP Long Polling
```kotlin
// 서버에 계속 요청을 보내는 방식
fun startLongPolling() {
    lifecycleScope.launch {
        while (true) {
            try {
                val response = apiService.getMessages()
                // 메시지 처리
                delay(1000) // 1초 대기
            } catch (e: Exception) {
                delay(5000) // 에러 시 5초 대기
            }
        }
    }
}
```

### WebSocket
```kotlin
// 한 번 연결하면 실시간으로 메시지 수신
webSocketManager.connect("wss://server.com/chat")
// 서버에서 메시지가 오면 즉시 onMessage 호출
```



## 6. 라이브러리 추천

### 7-1. OkHttp WebSocket
- **장점**: OkHttp 기반, 안정적
- **단점**: 설정이 복잡할 수 있음

### 7-2. Socket.IO
- **장점**: 자동 재연결, 폴백 지원
- **단점**: 라이브러리 크기가 큼

### 7-3. Ktor WebSocket
- **장점**: Kotlin 네이티브, 코루틴 지원
- **단점**: 상대적으로 새로운 라이브러리
