# HTTPS

## SSL
대칭키와 공개키를 혼용해서 사용하는 암호화 기법

### 대칭키
암호화 복호화를 같은 키를 사용해서 처리하는 기법

### 공개키
암호화와 복호화를 서로 다른 키를 사용해서 처리하는 기법
1. 비밀키로 암호화한 것은 공개키로 복호화할 수 있다. (인증-비밀키를 보유한 사람임을 확인)
2. 공개키로 암호화한 것은 비밀키로 복호화할 수 있다. (암호화-복호화)

### 인증서
1. 서버가 신뢰 할 수 있는 서버임을 보증
2. SSL 통신에 사용할 공캐키 제공

#### CA(Certificate authority)
인증서를 발급하는 기관, HTTPS를 제공하려면 CA를 통해 인증서를 구입하여야 함  
브라우저 밴더들은 CA목록을 보유하고 서버가 제공하는 인증서가 유효한지 확인함
CA가 발급한 인증서에는 다음의 내용을 포함함.  
1. 서비스의 정보(인증서를 발급한 CA, 도메인 정보 등)
2. 서버의 공개키(공개키의 내용과 암호화 방법)

브라우저가 서버에 접속하면 서버는 인증서를 전송한다. 브라우저는 인증서의 CA를 리스트에서 확인한다.    
인증서가 유효하다면 해당 인증서에 제공된 공개키를 이용해서 인증서를 복호화한다.    
인증서를 복호화 할 수 있다는 것은 CA의 비밀키로 암호화 된 것을 의미한다.
이는 해당 CA의 비밀키를 가지고 있는 CA는 그 CA밖에 없으므로 서버가 제공한 인증서가 CA에 발급된 것을 확인시켜준다.

### 동작방식
공개키를 통해 암호화하는 방식은 컴퓨팅 자원을 많이 소비한다. 따라서 대칭키와 공개키 방식을 혼합하여
대칭키 전송은 공개키 방식으로 안전하게 전달하고, 안전하게 전달된 대칭키를 통해 데이터를 주고 받는다.

#### 1. 악수
1. 클라이언트가 서버에 접속(Client Hello) - 정보 전송
- 클라이언트가 생성한 랜덤 데이터 
- 클라이언트가 사용 가능한 암호화 방식
- 세션 아이디(이미 SSL 핸드쉐이킹을 했다면 비용 절감을 위해 재사용, 이를 전송해 새로운 연결 방지)

2. 서버는 Client Hello를 Server Hello로 응답
- 서버에서 생성한 랜덤 데이터
- 서버가 선정한 암호화 방식
- 인증서

3. 클라이언트는 인증서의 CA를 확인하고 유효한지 확인 
- 유효하다면 CA에 내장된 공개키를 사용해
- 인증서를 복호화하고 성공하면 이 인증서가 CA의 비밀키로 암호화된 문서임이 인증

4. 서버의 랜덤데이터와 클라이언트의 랜덤 데이터를 조합하며 pre master secret키(대칭키) 생성
- 클라이언트는 공캐키를 사용해 pre master secret값을 암호화하여 서버로 전송
- 서버는 비밀키로 복호화 해서 대칭키 생성, 이를 통해 서버, 클라이언트 모두 대칭키 보유
- pre master secret키로 master secret 생성, master secret로 session key 생성

5. 클라이언트와 서버는 핸드쉐이크 단계 종류를 서로에게 전송

#### 2. 전송(세션)
서버와 클라이언트가 데이터를 주고 받는 단계, 데이터는 session key(대칭키)로 암호화하여 전송

#### 3. 세션종료
SSL 통신이 종료되면 서로에게 신호하고, 세션키를 폐기