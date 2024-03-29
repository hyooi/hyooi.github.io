---
layout: post
title:  "HTTPS? 디지털 인증서?"
categories:
- 보안
tags:
- 인증서
---

### 1. 개요
인터넷 서핑을 하다보면 브라우저에 보이는 자물쇠가 있다.
아래와 같은 버튼인데, 클릭해보면 **보안연결이 사용되었다**는 문구가 나타난다.

![http-lock](/assets/images/http-lock.PNG)

사용자에게는 해당 자물쇠의 유무가 인터넷 서핑에 그닥 영향을 미치지 않지만,
실제 네트워크 상에는 <ins>정보를 안전하게 주고받을 수 있는 장치가 추가되어 있는 것</ins>이다.

오늘은 해당 인증이 어떻게 진행되는지 정리할 예정이다.

<br/>

### 2. HTTPS
먼저 위에서 말하는 **보안연결(HTTPS)**란, 인터넷에서 정보를 교환할 때 사용하는 프로토콜인 HTTP에 
암호화 프로토콜(SSL, TLS)을 추가한 것이다.
> OSI 7Layer중 http는 응용계층에 해당되는데, SSL, TLS는 그 하위 레이어인 표현 계층에 해당한다.

SSL은 과거의 보안 기술로 최근에는 모두 TLS프로토콜을 사용한다고 생각하면 되는데,
이 때 필요한 것이 위의 캡쳐에서 확인했던 **인증서**이다.

<br/>

### 3. 디지털 인증서
아무 도메인이나 접속해서 자물쇠 버튼을 클릭하고, 인증서를 클릭해보자.

그러면 다음과 같은 팝업이 뜰 것이다.

![certificate](/assets/images/blog-certificate.png)

어디서 발급해주었고, 해당 인증서의 유효기간은 어떻게되며 어느 도메인을 통해 발급받았는지,
심지어는 <ins>서명 알고리즘에 인증서 내용까지</ins> 확인할 수 있다.

뭔가 중요한 정보들을 모조리 확인할 수 있는 것 같은데, 이걸 통해 어떻게 신뢰할 수 있는걸까?

인증서는 비대칭키 암호화 방식을 사용하는데, 해당 암호화 방식을 이해하면 그 과정을 이해할 수 있다.

<br/>

#### 3.1. 비대칭키 암호화방식(=공개키 암호화 방식)
비대칭키 암호화 방식은 기본적으로 **공개키와 개인키, 2개의 키를 생성**한다.
여기서 공개키는 방금 전 확인한 것처럼 누구나 사용할 수 있도록 공개하며, 비밀키는 안전하게 보관한다.
> 때문에 공개키 암호화 방식이라고도 불린다.

데이터를 요청했을 때, <ins>서버는 데이터를 개인키로 암호화해서 전송</ins>하고, 
사용자가 사용하는 <ins>브라우저는 가지고 있던 공개키(인증서)로 복호화해 데이터가 신뢰가능한지 판단</ins>한다.

위의 캡처에서 서명 알고리즘이 **SHA256RSA**라고 나와있는데,
여기서 SHA256은 해시 알고리즘이며 RSA는 비대칭키 암호화방식 중 한가지이다.


비대칭키 암호화 방식에는 RSA뿐만 아니라 DSA, 비트코인에서 사용하는 ECSDA가 있다.

<br/>

#### 3.2. HTTPS 통신 시 인증 과정(전자서명)
##### 3.2.1. 웹서버에 비밀키 등록
그럼 실제로 인터넷을 통해 <var>https://hyooi.github.io/</var>에 접근했을 때, 가져온 데이터를 어떻게 신뢰할 수 있는지 정리해보자.

먼저, github는 **비대칭키 암호화 방식에 따른 두 쌍의 키를 발급**받는다.

그리고 하나는 사용자에게 공개하고, 비밀키는 <var>https://hyooi.github.io/</var>를 호스팅하고 있는 github의 웹서버에 등록했을 것이다.

그렇게되면 해당 웹서버를 통해 받아오는 데이터는 모두 **등록한 비밀키를 통해 서명된 상태로 전달**되게 된다.

<br/>

##### 3.2.2. 서버 -> 클라이언트 데이터 송신
클라이언트에서 요청한 데이터는 서버에서 처리 후 송신하게 되는데,

이 때 보낼 데이터는 먼저 해싱 알고리즘을 통해 해싱되고, 해싱한 결과를 비밀키로 암호화한다.
이를 **전자서명** 이라고 부르는데, 원본 데이터에 이 서명을 붙여서 보내게 된다.
> 비대칭키 암호화 방식을 통한 암호화는 복호화가 가능하나, 해싱은 복호화가 불가능하다.

<br/>

##### 3.2.3. 데이터 수신
그럼 사용자는 <var>원본 데이터 + 전자서명</var>된 데이터를 받게 된다.

사용자가 해당 데이터를 받아올 때는, 먼저 반대로 <ins>원본 데이터와 전자서명을 분리</ins>한다.
그리고 가지고 있는 공개키를 통해 전자서명을 복호화하고,
원본 데이터를 해싱해 방금 복호화한 전자서명과 동일한지 확인한다.

<ins>비밀키로 암호화한 데이터는 공개키로만 복호화 가능하므로,
값이 동일하다면 원본 데이터가 변조되지 않았음을 확인</ins>할 수 있다.

다음은 [위키](https://en.wikipedia.org/wiki/Electronic_signature)에서 가져온 그림인데, 
즉 데이터 송수신 시 다음 그림과 동일한 과정을 거친다고 생각하면 된다.

![digital-signature.png](/assets/images/digital-signature.png)

<br/>

##### 3.2.4. 인증서(공개키) 신뢰 가능 여부
이 때, 바이너리를 복호화한 공개키는 어떻게 신뢰할 수 있을까?
그건 인증서를 발급해주는 기관(CA)에 있다.

브라우저의 자물쇠 버튼을 클릭하고, 인증서 > 인증 경로를 확인해보면 다음과 같은 트리 구조를 확인할 수 있다.

![root-ca](/assets/images/root-ca.PNG)

내 블로그에 적용되어있는 인증서의 발급 대상은 <var>www.github.com</var>인데,
이 인증서의 상위 인증서는 <var>DigiCert SHA2 High Assurance Server CA</var>,
그리고 그 상위엔 <var>DigiCert</var>가 있는 것이다.

OS는 기본적으로 Root CA의 리스트를 가지고 있다. 
OS가 업데이트되면서 내부적으로 가지고 있는 이 리스트도 업데이트하는데,
내가 발급받은 인증 기관의 상위 CA, 즉 **Root CA가 내가 가진 리스트에 포함된다면 신뢰 가능하다고 판단**하는 것이다.

그러므로 직접 서명한 인증서(self-signed certificate)는 암만 웹서버에 등록해봤자
Root CA를 확인할 수 없으므로 접속이 차단된다.

또한, Root CA가 만료되는 케이스도 있는데([gitlab-ssl-2참조](https://hyooi.github.io/%EC%84%9C%EB%B2%84/2021/09/17/gitlab-ssl-2.html)) 이 경우에도 접속이 불가하다.
보통 OS가 자동으로 업데이트하니 해당 케이스로 인한 접속 불가현상은 잘 만나보기 힘들다.

<br/>

### 4. 결론
사실 이론으로만 아는 것보다는, 직접 인증서를 발급받고 웹서버에 등록해 https가 적용된 것을 확인하는 편이 직접 와닿기는 하는 것 같다.

관련해 직접 인증서를 발급받고, 웹서버에 등록하는 내용을 포스팅한 링크를 추가해둔다.
- [Lets encrypt인증서 발급하기](https://hyooi.github.io/%EC%84%9C%EB%B2%84/2021/09/17/gitlab-ssl-2.html)
- [Nginx reverse proxy 설정하기](https://hyooi.github.io/%EC%84%9C%EB%B2%84/2021/09/17/gitlab-ssl-3. html)
