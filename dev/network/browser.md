## DNS 조회
- DNS 조회

브라우저는 DNS 서버에 www.naver.com의 IP 주소를 요청, DNS 서버는 도메인 이름을 해당 IP주소로 변환

- TCP 연결 

DNS 서버가 IP 주소를 반환하면 브라우저는 해당 IP주소로 TCP 연결을 시도

일반적으로 80번 포트 (HTTP), 443번 포트 (HTTPS) 로 연결 

TCP 핸드셰이크 브라우저와 웹 서버 간에 TCP 3-way 핸드셰이크 (SYN, SYN-ACK, ACK) 가 이루어진다. 

- HTTP/HTTPS 요청

TCP 연결이 완료되면, 브라우저는 HTTP 또는 HTTPS 요청을 웹 서버로 전송한다. www.naver.com 입력할 때 기본적으로 GET 요청을 전송한다. 

```java
GET / HTTP/1.1
Host: www.naver.com
```

- 서버처리

웹 서버에서 요청을 수신하고 처리한다. 

- 어플리케이션 서버와 데이터베이스 상호작용 

- 응답 생성 및 전송 
- 클라이언트 렌더링

 DNS 조회 (Layer 7: Application Layer) 
- 사용자 입력: 사용자가 브라우저 주소창에 www.naver.com을 입력하고 엔터를 누릅니다.
- DNS 조회: 브라우저는 DNS 서버에 www.naver.com의 IP 주소를 요청합니다. 이 과정은 애플리케이션 계층에서 이루어집니다.

TCP 연결 (Layer 4: Transport Layer)
- IP 주소 반환: DNS 서버가 IP 주소를 반환하면, 브라우저는 해당 IP 주소로 TCP 연결을 시도
- TCP 핸드셰이크: 브라우저와 웹 서버 간에 TCP 3-way 핸드셰이크 (SYN, SYN-ACK, ACK)가 이루어집니다. 이 과정은 전송 계층 

 HTTP/HTTPS 요청 (Layer 7: Application Layer)
 - TCP 연결이 완료되면, 브라우저는 HTTP 또는 HTTPS 요청을 웹 서버로 전송합니다. 예를 들어, 사용자가 www.naver.com을 입력했을 때 기본적으로 GET 요청이 전송


 서버 처리 (Layer 7: Application Layer)
 - 요청 처리: 웹 서버 (예: Nginx, Apache)에서 요청을 수신하고 처리

 어플리케이션 서버와 데이터베이스 상호작용 (Layer 7: Application Layer)

 응답 생성 및 전송 (Layer 7: Application Layer)

 클라이언트 렌더링 (Layer 7: Application Layer)

OSI 7계층과의 매핑
물리 계층 (Layer 1): 실제 물리적 네트워크 매체 (케이블, 광섬유 등) 위에서 데이터 전송.

데이터 링크 계층 (Layer 2): 네트워크 상의 노드 간 데이터 프레임 전송 (MAC 주소 사용).

네트워크 계층 (Layer 3): 데이터 패킷이 여러 네트워크를 통해 전달되도록 라우팅 (IP 주소 사용).

전송 계층 (Layer 4): 호스트 간 신뢰성 있는 데이터 전송 (TCP/UDP 사용).

세션 계층 (Layer 5): 응용 프로그램 간 세션 설정, 관리, 종료.

표현 계층 (Layer 6): 데이터의 형식, 암호화, 압축을 처리.

응용 계층 (Layer 7): 최종 사용자와 상호작용하는 응용 프로그램 및 서비스 (HTTP, DNS, FTP 등).

----------------------------------------------------------

다시 OSI 7계층으로 설명을 해보자.

데이터 전달할 때 상위 계층에서 하위 계층으로 내려가고

반대로 데이터를 수신할 때 하위 계층에서 상위 계층으로 올라간다. 

<요약>

1. 응용 계층 
- 사용자가 브라우저에 www.naver.com 입력하고 엔터를 누른다. 
- 브라우저가 HTTP/HTTPS 요청을 생성한다. 

2. 표현 계층
- 브라우저는 HTTP/HTTPS 데이터를 인코딩 및 암호화 한다. 

3. 세션 계층
- 브라우저는 서버와의 세션을 관리한다. 

4. 전송 계층
- 브라우저는 TCP 3-way 핸드셰이크를 통해 서버와 TCP 연결을 설정 

5. 네트워크 계층
- TCP 세그먼트는 IP패킷으로 캡슐화된다. 
- IP주소를 기반으로 패킷이 목적지로 라우팅 된다. 

6. 데이터 링크 계층 
- IP 패킷은 프레임으로 캡슐화된다. 
- 물리적 네트워크(와이파이) 를 통해 프레임이 전송된다. 

7. 물리 계층
- 프레임은 실제 물리적 매체를 통해 비트로 전송된다. 

## 서버에서 처리
- 물리 계층
    - 서버는 물리적 매체를 통해 수신된 비트를 수신한다.
- 데이터 링크 계층
    - 수신된 비트는 프레임으로 재구성된다. 
    - 프레임으로 분석하여 상위 계층으로 전달한다.
- 네트워크 계층
    - 프레임에서 IP 패킷을 추출
    - IP 패킷을 분석하여 상위 계층으로 전달
- 전송 계층
    - IP 패킷에서 TCP 세그먼트를 추출
    - TCP 세그먼트를 분석하고, 세그먼트 순서를 재조합
    - TCP 연결을 통해 데이터의 신뢰성을 보장
- 세션 계층
    - 세션을 관리하고, 세션 정보에 따라 데이터를 전달
- 표현 계층
    - 데이터 인코딩 및 암호화를 해제합니다(HTTPS의 경우).
- 응용 계층
    - HTTP/HTTPS 요청을 분석하여 해당 요청을 처리
    - 데이터베이스에 접근하여 필요한 데이터를 조회
    - 조회한 데이터를 JSON 등의 형식으로 응답
    - HTTP/HTTPS 응답을 생성하여 클라이언트로 전송

클라이언트에서의 처리

물리 계층 (Physical Layer): 물리적 매체로부터 비트를 수신.

데이터 링크 계층 (Data Link Layer): 비트를 프레임으로 재구성.

네트워크 계층 (Network Layer): 프레임에서 IP 패킷을 추출.

전송 계층 (Transport Layer): IP 패킷에서 TCP 세그먼트를 추출하고 재조합.

세션 계층 (Session Layer): 세션 정보 관리.

표현 계층 (Presentation Layer): 데이터 디코딩 및 복호화.

응용 계층 (Application Layer): 브라우저가 HTTP/HTTPS 응답을 처리하여 웹 페이지를 렌더링.