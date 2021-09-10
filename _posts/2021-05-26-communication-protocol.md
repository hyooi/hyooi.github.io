---
layout: post
title:  "통신 프로토콜"
published: false
---

# TCP(Transmission Control Protocol)
- TCP는 IP에 연결 지향 기능 및 신뢰성을 추가함
- TCP는 신뢰할 수 있는 스트림 전달 서비스
- 중복되거나 손실된 데이터 없이 한 호스트에서 전송한 데이터 스트림을 다른 호스트로 전달할 수 있도록 보장
- 패킷 전송은 신뢰할 수 없으므로, 재전송을 사용하는 긍정 수신확인이라고 하는 기술을 사용하여 패킷 전송의 신뢰성을 보장
- 발신인은 전송한 각 패킷의 레코드를 보존하고 다음 패킷을 전송하기 전에 수신확인을 기다림
- 또한 패킷 전송 당시의 타이머도 보존하여 타이머가 만료되면 패킷을 다시 전송합니다. 타이머는 패킷이 손실되었거나 손상되었을 경우에 필요합니다.
<br/><br/>
  


# UDP(User Datagram Protocol))
- 비연결형(따라서 신뢰할 수 없음) 전송 프로토콜
- 오류 검사를 거의 수행하지 않으며 호스트 간 통신 대신 프로세스 간 통신을 제공하는 것 이외에 IP 서비스에 아무것도 추가하지 않음
- 최소한의 오버헤드가 발생하는 단순 프로토콜
- 프로세스에서 소규모의 메시지를 전송하려고 하며 신뢰성이 중요하지 않은 경우에 UDP를 사용할 수 있음
- UDP를 사용하여 메시지를 전송할 경우 TCP를 사용하는 경우보다 시간이 덜 걸림
<br/><br/>
  

# HTTP
- HTML 문서와 같은 리소스들을 가져올 수 있도록 해주는 프로토콜
- 웹에서 이루어지는 모든 데이터 교환의 기초 
- TCP 혹은 암호화된 TCP 연결인 TLS를 통해 전송
- 클라이언트-서버 프로토콜
  - 요청은 사용자 에이전트에 의해 전송됨. EX. 브라우저, 웹을 돌아다니는 로봇
  - 통신 채널의 반대편에는 클라이언트에 의한 요청에 대한 문서를 제공하는 서버가 존재
- 기본적으로 stateless이므로 동일한 연결 상에 연속된 요청들은 연결고리가 없음.
- 그러나 쿠키, 세션 사용 가능
<br/><br/>
  

## HTTP1.1과 HTTP2의 차이
1. HTTP1.0
- HTTP는 OSI 7layer상 7계층이고, 연결은 4.전송계층에서 진행됨. 따라서 HTTP영역 밖
- 기본적으로 TCP를 이용하는데, HTTP는 각 요청/응답에 별도의 TCP연결을 사용함
- 따라서 여러 요청을 연속해서 보내는 경우, 단일 TCP연결보다 비효율
<br/><br/>
  

2. HTTP1.1
- 1번 단점 보완을 위해 파이프라이닝, (첫번째 응답을 완전히 수신할 때까지 기다리지 않고 여러 요청 가능. but구현어려움)
- keep alive개념 도입(이미 연결된 tcp연결을 재사용)
<br/><br/>
  

3. HTTP2
- 단일 연결 상에서 메시지를 multiplex할 수 있게됨
- HTTP Message: 사람이 읽을 수 있던 이전과는 달리 프레임 내부로 임베디드되어,
헤더의 압축과 다중화와 같은 최적화가 가능해짐
<br/><br/>
  

## HTTP흐름
1. TCP커넥션 오픈. 설정에 따라 새 커넥션을 열거나 기존 연결 재사용 가능
2. HTTP메시지 전송(HTTP2이전엔 사람이 읽기 가능)
{% highlight bash %}
-- 요청
GET / HTTP/1.1
Host: developer.mozilla.org
Accept-Language: fr

-- 응답
HTTP/1.1 200 OK
Date: Sat, 09 Oct 2010 14:28:02 GMT
Server: Apache
Last-Modified: Tue, 01 Dec 2009 20:18:22 GMT
ETag: "51142bc1-7449-479b075b2891b"
Accept-Ranges: bytes
Content-Length: 29769
Content-Type: text/html

<!DOCTYPE html... (here comes the 29769 bytes of the requested web page)
{% endhighlight %}
3. 연결을 닫거나 다른 요청을 위해 커넥션 재사용
