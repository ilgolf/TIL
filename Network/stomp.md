# STOMP 

STOMP는 메시지 지향 미들웨어에서 메시지를 전달하기 위해 사용되는 프로토콜이다. 주로 웹소켓을 통해 메시지를 전송할 때 사용되며, 메시지 브로커와 클라이언트 간의 통신을 단순화하는 역할을 한다. 
STOMP는 메시지큐 시스템과 상호작용 하기 위해 설계된 텍스트 기반 프로토콜이다.

## 주요 특징 

1. 텍스트기반: STOMP 메시지는 텍스트 형식으로 표현되므로 사람이 읽고 이해하기 쉽다.
2. 프레임: 메시지는 프레임 단위로 구성, 각 프레임은 특정 명령과 함께 헤더와 본문을 포함한다.
3. 호환성: 다양한 메시지 브로커와 호환된다.
4. 간단함: 구현이 간단하여 빠르게 적용이 가능하다. 

클라이언트가 서버에 연결하고 메시지를 보내는 과정은 다음과 같다. 

1. CONNECT 프레임: 클라이언트가 서버에 연결을 요청하는 프레임


```shell
CONNECT
accept-version:1.2
host:example.com

\0
```

2. SEND 프레임: 클라이언트가 특정 목적지로 메시지를 보내는 프레임

```shell
SEND
destination:/queue/test

Hello, World!\0
```

3. SUBSCRIBE 프레임: 클라이언트가 특정 목적지의 메시지를 수신하기 위해 구독을 요청하는 프레임

```shell
SUBSCRIBE
id:sub-0
destination:/queue/test

\0
```

## Kotlin Example

StompServer.kt

```kotlin
fun main() {
    val server = StompServer()
    server.start()
}

class StompServer(
    private val port: Int = 8080
) {

    fun start() {
        val socket = ServerSocket(port) // Create a server socket

        println("STOMP server started on port $port")

        // Accept incoming connections in a loop
        while (true) {
            val client = socket.accept()
            println("Client connected: ${client.inetAddress.hostAddress}")

            // Handle each client in a separate thread (single Thread)
            thread { handleClient(client) }
        }
    }

    private fun handleClient(client: Socket) {
        val reader = BufferedReader(InputStreamReader(client.getInputStream()))
        val writer = client.getOutputStream().bufferedWriter()

        while (true) {
            // Read a frame from the client
            val frame = readFrame(reader)

            // Process the frame
            if (frame != null) {
                println("Received frame: $frame")
                processFrame(frame, writer, client)
            }
        }
    }

    // Read a frame from the client
    private fun readFrame(reader: BufferedReader): String? {
        val frame = StringBuilder()
        var line: String?

        // Read until an empty line is encountered
        while (reader.readLine().also { line = it } != null) {
            if (line!!.isBlank()) break
            frame.append(line).append("\n")
        }

        // Return the frame if it's not empty
        return if (frame.isNotBlank()) frame.toString() else null
    }

    private fun processFrame(frame: String, writer: BufferedWriter, clientSocket: Socket) {
        val lines = frame.lines()

        when (val command = lines[0]) {
            "CONNECT" -> {
                val response = "CONNECTED\nversion:1.2\n\n\u0000"
                writer.write(response)
                writer.flush()
            }
            "SEND" -> {
                val destination = lines.find { it.startsWith("destination:") }?.substringAfter(":")
                val message = lines.last()

                println("Message to $destination: $message")
                // Echo the message back for simplicity
                val response = "MESSAGE\ndestination:$destination\n\n$message\u0000"
                writer.write(response)
                writer.flush()
            }
            "DISCONNECT" -> {
                clientSocket.close()
                println("Client disconnected")
            }
            else -> {
                println("Unknown command: $command")
            }
        }
    }
}
```

StompClient.kt
```kotlin
fun main() {
    val client = StompClient()
    client.connect()
    client.send("/topic/test", "Hello, world!")
    client.disconnect()
}

class StompClient(
    private val host: String = "localhost",
    private val port: Int = 8080
) {

    private lateinit var socket: Socket
    private lateinit var reader: BufferedReader
    private lateinit var writer: OutputStream

    fun connect() {
        socket = Socket(host, port)
        reader = BufferedReader(InputStreamReader(socket.getInputStream()))
        writer = socket.getOutputStream()

        val connectFrame = "CONNECT\naccept-version:1.2\n\n\u0000"
        writer.write(connectFrame.toByteArray())
        writer.flush()

        val response = readFrame()
        println("Received response: $response")
    }

    fun send(destination: String, message: String) {
        val sendFrame = "SEND\ndestination:$destination\n\n$message\u0000"
        writer.write(sendFrame.toByteArray())
        writer.flush()
    }

    fun disconnect() {
        val disconnectFrame = "DISCONNECT\n\n\u0000"
        writer.write(disconnectFrame.toByteArray())
        writer.flush()
        socket.close()
    }

    private fun readFrame(): String? {
        val frame = StringBuilder()
        var line: String?

        while (reader.readLine().also { line = it } != null) {
            if (line!!.isEmpty()) break
            frame.append(line).append("\n")
        }

        return if (frame.isNotEmpty()) frame.toString() else null
    }
}
```
