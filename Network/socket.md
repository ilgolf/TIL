# Socket이란?

socket은 process 간 통신을 하기위한 수단으로써 데이터를 보내거나 받을 때 process는 socket을 통해 받는다. 소켓은 주로 로컬 주소 포트 프로토콜 정보등으로
구성되어있으며 이를 통해 프로세스를 식별하여 데이터를 주고 받는다. 

# 동작 과정

1. 소켓 생성
socket 시스템 호출을 통해 생성 이는 네트워크 통신을 위한 소켓 파일 디스크립터를 반환한다.

```c
int sockfd = socket(AF_INET, SOCKET_STREAM, 0);
```

2. 바인딩
서버 소켓의 경우 특정 IP 주소와 포트 번호에 바인딩 되어야한다. `bind` system call을 통해 이루어진다.

```c
struct sockaddrin serv_addr;

serv_addr.sin_family = AF_INET;
serv_addr.sin_addr,s_addr = INADDR_ANY;
serv_addr.sin_port = htons(8000);
```

3. 연결 대기
서버 소켓은 연결 요청을 대기하기 위해 `listen` system call을 사용하고, 클라이언트 연결 요청을 수락하기 위해 `accept` system call을 사용

```c
listen(socketfd, 5);
int newsockfd = accept(sockfd, (struct sockaddr *)&cli_addr, &clilen);
```

4. 데이터 송수신
클라이언트와 서버는 `send`와 `recv` system call을 통해 데이터를 주고 받음

```c
send(sockfd, "Hello World!", 13, 0);
recv(sockfd, buffer, 1024, 0);
```

## 높은 동시성을 갖는 socket 

이벤트 기반 I/O를 이용하면 고성능 socket을 구축 할 수 있다. 

`select`, `poll`, `epoll` 등과 같은 function을 이용하면 비동기적으로 소켓 이벤트를 처리할 수 있다. 이는 높은 동시성을 요구하는 네트워크 서버에서
주로 사용된다. 

대표적인 예) reids, netty, ...


