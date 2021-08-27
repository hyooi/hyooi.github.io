---
layout: post
title:  "마이크로서비스 서비스 분리 전략"
---

# 작고 분리가 쉬운 서비스로 워밍업하라
- 작은 서비스를 먼저 분리하며 역량을 내재화함
- cloud, deployment pipeline, container, monitoring등의 프로세스 구축
- 서비스 선택 조건
  - 신규 개발되는 작은 기능 혹은, 기존 monolith에서 분리 가능한 작은 기능
  - 내부적인 의존성이 적은 기능
  - 데이터의 크기, 테이블의 갯수가 적은 기능
  - 장애가 발생해도 전체 시스템에 영향이 적은 기능


# 핵심 기능의 분리
- 상품 전치, 결제, 회원, 추천 등의 도메인을 발견해 분리 대상 선정
- 비즈니스 팀 구조를 기반으로 분리
- 도메인 주도 설계 적용


# 데이터의 분리
- msa 전환 초기에는 워밍업을 위해 코드만 분리
- 그러나 결국 공유db는 안티패턴이므로, 분리해야 함


# 코드의 재사용 vs 재개발
- 일반적으로는 코드를 재사용하는 것이 효율적으로 보이나, 오히려 비효율을 초래할 가능성이 높음
- 오랜 기간 유지보수된 코드로 기술부채가 많이 쌓여있을 가능성 높음
- 기능을 재작성해 쌓여있던 기술 부채를 해결하고 비즈니스 도메인을 명확화할 수 있음


# 진화적인 서비스 분리
- 우선 크게 분리하고, 필요한 경우 재설계를 통해 더 작게 분리


# 반복/점진적 분리
- 한번에 하나씩 단계적으로 분리


# 서비스 분리와 조직
- 기존 기능을 유지보수 하면서 MSA분리하기는 어려움
- 따라서 새로운 서비스를 만들 전담 팀이 필요함