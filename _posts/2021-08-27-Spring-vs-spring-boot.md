---
layout: post
title:  "스프링 VS 스프링부트"
---

# 스프링?
- 스프링과 스프링부트의 차이점을 알기 위해서는, 먼저 스프링을 알아야 한다.
- 스프링은 복잡한 EJB의 대안으로 개발된 java기반 프레임워크
- DI와 XML기반 설정으로 비즈니스 코드에 FW를 위한 별도의 코드가 들어가지 않음


# 스프링부트?
- 그러나 스프링도 다양한 설정 탓에 복잡도가 증가
- 이에 따라 컨벤션 기반 설정이 가능한 모던 프레임워크인 스프링부트가 등장하게 됨

## 스프링부트의 특징
### 1. auto configuration
- 특정 jar를 사용하면 해당 설정을 자동화
EX. Spring mvc가 있으면 dispatcher servlet자동 구성

- 컨벤션 기반 설정
  - 정해진 이름 혹은 폴더에 파일넣으면 자동으로 인식(ex. static, application.yml 등)
  
- 간편화된 어노테이션 설정
EX. @SpringBootApplication 
  = @EanbleAutoConfiguration + @ComponentScan + @Configuration


### 2. Spring boot starter projects
- 자주 사용하는 의존성을 패키지화
EX. Spring boot starter test = junit + assert j 등


### 3. embeedded server integration
- 기본적으로 내장was 사용


### 4. actuator
- 별도의 모니터링 환경 없이 운영중인 app의 health, metrics확인 가능
- rest api로 쉽게 확인
- 연동되어있는 db와 같은 미들웨어의 상황도 파악 가능
