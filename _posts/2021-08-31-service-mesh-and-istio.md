---
layout: post
title:  "Service mesh와 Istio"
---

# Service mesh
- msa에서 서비스 간 통신을 관리, 제어하는 layer
- pod간 직접 호출이 아닌, proxy를 통해 다양한 기능 추가 가능함
- 언어나 프레임워크에 독립적
- spring cloud는 자바+스프링 기반으로 어플리케이션 레벨에서 처리해야 했으나, 
    istio + kubernates의 service mesh로 인프라 레벨에서 처리할 수도 있음
- ex. kubernates에서 pod간의 통신을 제어함. pod간의 직접통신X


# istio
- service mesh를 구현한 구현체
- google이 개발해 kubernates와 사용하기 쉬움
- application code의 수정없이 기능추가 가능
- EX. 로드밸런싱(헤더에 따라 밸런싱 등), 트래픽제어, 접근제어 등의 정책 및 매트릭, 로그, 트레이스 수집, 인증 및 인가


# envoy
- istio에서 proxy역할을 함
- istio의 핵심!
- 사이드카 패턴으로 모든 pod에 함께 배포됨
- istio의 observability기능 등 대부분의 기능을 envoy가 처리함
