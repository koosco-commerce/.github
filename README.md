# Commerce Platform

이벤트 기반 아키텍처, 데이터 일관성, 장애 처리 등 분산 시스템의 핵심 과제를 학습하고 실험하기 위한 프로젝트입니다.

## 개요

이 프로젝트는 프로덕션 수준의 완성도보다는 백엔드 개발 실무 학습에 중점을 둔 실습 플랫폼입니다. 비동기 통신 패턴, 서비스 간 트랜잭션 일관성, 부하 상황에서의 시스템 동작 분석 등을 다룹니다.

## 아키텍처

### 핵심 서비스

| 서비스 | 역할 | 저장소 |
|---------|------|--------|
| **auth-service** | JWT 토큰 관리 및 인증 | [koosco-commerce/auth-service](https://github.com/koosco-commerce/auth-service) |
| **user-service** | 사용자 계정 및 프로필 관리 | [koosco-commerce/user-service](https://github.com/koosco-commerce/user-service) |
| **catalog-service** | 상품 카탈로그 및 카테고리 관리 | [koosco-commerce/catalog-service](https://github.com/koosco-commerce/catalog-service) |
| **inventory-service** | 재고 수준 추적 및 예약 | [koosco-commerce/inventory-service](https://github.com/koosco-commerce/inventory-service) |
| **order-service** | 주문 처리 워크플로우 | [koosco-commerce/order-service](https://github.com/koosco-commerce/order-service) |
| **payment-service** | 결제 트랜잭션 처리 | [koosco-commerce/payment-service](https://github.com/koosco-commerce/payment-service) |

### 공유 라이브러리

| 모듈 | 역할 | 저장소 |
|------|------|--------|
| **common-core** | 예외 처리, API 응답, 이벤트 스키마 | [koosco-commerce/common-core](https://github.com/koosco-commerce/common-core) |
| **common-security** | JWT 검증 및 인증 필터 | [koosco-commerce/common-security](https://github.com/koosco-commerce/common-security) |
| **common-observability** | 로깅 설정 및 MDC 관리 | [koosco-commerce/common-observability](https://github.com/koosco-commerce/common-observability) |

### 인프라

| 구성요소 | 저장소 |
|----------|--------|
| **infra** | Kubernetes 매니페스트, Kafka, 모니터링 설정 | [koosco-commerce/infra](https://github.com/koosco-commerce/infra) |
| **load-test** | k6 기반 성능 테스트 시나리오 | [koosco-commerce/load-test](https://github.com/koosco-commerce/load-test) |

## 기술 스택

### 백엔드 서비스
- **언어**: Kotlin 1.9.25
- **프레임워크**: Spring Boot 3.5.8
- **런타임**: Java 21
- **데이터베이스**: MariaDB (JPA, QueryDSL)
- **메시지 브로커**: Apache Kafka
- **데이터 일관성**: Kafka CDC (Change Data Capture)
- **스키마 마이그레이션**: Flyway
- **코드 포맷팅**: Spotless

### 인프라
- **로컬 개발**: Docker Compose
- **개발 환경**: k3d (Kubernetes in Docker)
- **프로덕션 환경**: k3s (경량 Kubernetes)
- **모니터링**: Prometheus, Grafana
- **통신 방식**: 비동기 이벤트 기반 통신

## 설계 원칙

### 클린 아키텍처
각 서비스는 클린 아키텍처 원칙을 따르며 명확한 관심사 분리를 유지합니다:
- **API 레이어**: 컨트롤러 및 DTO
- **Application 레이어**: 유스케이스 및 커맨드 핸들러
- **Domain 레이어**: 비즈니스 로직 및 엔티티
- **Infrastructure 레이어**: 데이터베이스, 메시징, 외부 연동

의존성은 내부로 흐릅니다: API → Application → Domain ← Infrastructure

### 이벤트 기반 통신
- 모든 서비스 간 통신은 Kafka를 통한 비동기 메시징 사용
- 서비스는 상태 변경을 나타내는 도메인 이벤트 발행
- 이벤트는 CloudEvents v1.0 스펙 준수
- 컨슈머는 멱등성을 보장하도록 설계

### 데이터 아키텍처
- 각 서비스는 독립적인 데이터베이스 스키마 소유 (논리적 분리)
- Kafka CDC로 데이터베이스 업데이트와 이벤트 간 트랜잭션 일관성 보장
- Redis는 AOF 영속성을 사용한 분산 캐싱 제공

## 시작하기

### 필수 요구사항
- Java 21
- Docker 및 Docker Compose
- Gradle 8.x
- GitHub Personal Access Token (공유 라이브러리 의존성용)

### 환경 설정

1. **GitHub 자격증명 설정**
   ```bash
   export GH_USER=your-github-username
   export GH_TOKEN=your-github-token
   ```

   또는 `~/.gradle/gradle.properties`에 추가:
   ```properties
   gpr.user=your-github-username
   gpr.token=your-github-token
   ```

2. **인프라 시작**
   ```bash
   cd infra/docker
   docker-compose up -d
   ```

3. **개별 서비스 실행**
   ```bash
   cd user-service
   ./gradlew bootRun
   ```

각 서비스의 상세한 설정 방법은 개별 저장소의 문서를 참조하세요.

## 관찰 가능성 (Observability)

### 모니터링 스택
- **Prometheus**: 서비스 및 부하 테스트 메트릭 수집
- **Grafana**: 시스템 동작 분석을 위한 시각화 대시보드
- **구조화된 로깅**: 모든 서비스에 걸친 MDC 기반 컨텍스트 로깅

모니터링 설정은 `infra/monitoring`에 중앙화되어 있습니다. 모든 서비스는 Prometheus 수집을 위한 메트릭 엔드포인트를 노출합니다.

### 부하 테스트
부하 테스트는 시스템 동작을 관찰하기 위한 의도적인 실험으로 설계되었습니다:
- **Smoke Test**: 기본 기능 검증
- **Baseline Test**: 성능 기준선 확립
- **Stress Test**: 시스템 한계 및 장애 모드 식별

부하 테스트는 반드시 명시적으로 실행해야 하며 CI/CD에서 자동으로 실행되지 않습니다.

## 개발 가이드라인

### 아키텍처 제약사항
- Application 및 Domain 레이어는 API나 Infrastructure 레이어에 의존하면 안 됨
- 서비스 간 동기 HTTP 호출 금지; Kafka 이벤트만 사용
- 공유 라이브러리 계약 변경 시 하위 호환성 고려 필요

### 코드 표준
- Application 서비스 클래스에 `@UseCase` 어노테이션 사용
- 기존 패키지 구조 및 네이밍 규칙 준수
- 핵심 비즈니스 로직에 대한 테스트 커버리지 유지
- 커밋 전 Spotless 포맷팅 적용

## 프로젝트 목표

이 프로젝트는 다음에 중점을 둡니다:
- **학습**: 실습을 통한 분산 시스템 패턴 이해
- **실험**: 일관성 및 장애 처리에 대한 다양한 접근 방식 테스트
- **관찰**: 다양한 부하 조건에서의 시스템 동작 분석
- **반복**: 메트릭과 성능 분석 기반 지속적 개선

이 프로젝트는 프로덕션 시스템이 아닌 학습 플랫폼입니다. 기능의 완성도보다는 트레이드오프 이해와 시스템 동작 분석에 초점을 맞추고 있습니다.

## 라이선스

이 프로젝트는 교육 및 포트폴리오 목적으로 제작되었습니다.

## 참고 자료

- [아키텍처 결정 기록](./docs/adr) (있는 경우)
- [API 문서](./docs/api) (있는 경우)
- [개발 가이드](./CLAUDE.md)

서비스별 상세 문서, 구현 세부사항, API 명세는 위에 링크된 개별 서비스 저장소를 참조하세요.
