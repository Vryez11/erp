# CLAUDE.md

이 파일은 Claude Code(및 협업 AI)가 이 저장소에서 작업할 때 참고하는 프로젝트 가이드입니다.

## 프로젝트 개요

- 여수의 PVC 소기업 사내 **ERP** 시스템. 사무실 직원 약 3명이 사용.
- 사무실 PC에서만 접근(외부 미노출). 보안은 공유기 고정 IP 통제 수준으로 충분.
- 목적: **실무 사용**. 하지만, 코드를 같이 보며 이해하는 것을 중시함.

## 기술 스택

- Java 17 + Spring Boot 3.5.x + Gradle (Groovy DSL)
- 화면: **Thymeleaf**(서버사이드 렌더링) + **AdminLTE v4**(Bootstrap 5 기반 관리자 템플릿) + Thymeleaf Layout Dialect
  - AdminLTE/Bootstrap은 **로컬 설치**(번들)로 사용한다. 사무실 내부망에서 인터넷 없이도 동작해야 하므로 CDN을 쓰지 않는다.
  - CSS를 직접 작성하지 않고 AdminLTE/Bootstrap 클래스를 활용한다.
- 인증: 추후 **Spring Security(세션 기반)** 추가 예정. SSR이므로 JWT 아님. (현재 미적용)
- DB: 추후 **PostgreSQL** 예정. **현재는 DB 없이 시작**.

## DB / 영속성 전략 (중요)

초기에는 DB를 쓰지 않고 **`ConcurrentHashMap` + `AtomicLong`** 으로 메모리에 데이터를 저장한다.
(앱을 끄면 데이터가 날아가도 무방한 단계.)

단, 나중에 구현체만 Map → JPA로 갈아끼울 수 있도록 **Repository 인터페이스로 추상화**한다.

반드시 지킬 3원칙:
1. **ID 채번은 Repository 내부에서** 한다. (서비스/컨트롤러가 ID를 만들지 않는다)
2. Repository **인터페이스는 단순 CRUD 수준**으로 유지한다. (save / findById / findAll / deleteById 등)
3. **도메인 객체는 POJO**로 둔다. (JPA 어노테이션을 붙이지 않는다)

## 패키지 구조 (도메인별)

기본 패키지: `com.yeosu.pvc.erp`
(Gradle group은 하이픈 불가라 `com.yeosu.pvc`)

```
com.yeosu.pvc.erp
├── common/    공통 (config, BaseEntity, exception, response 등)
├── auth/      로그인/인증 (추후)
└── partner/   거래처 — 첫 모듈
```

각 도메인 모듈은 세로 슬라이스로 구성: `domain` → `repository`(인터페이스 + Map 구현) → `service` → `controller` → Thymeleaf 화면.

## 진행 순서

1. 공통 뼈대 + 로그인
2. **거래처(Partner) 모듈** 세로 슬라이스 ← 현재 작업 시작 지점.
   (마스터 데이터라 독립적이고 CRUD가 단순해 **표준 패턴 확립용**으로 적합)
3. 이후 실무를 보며 모듈(품목/재고/매출 등)을 하나씩 추가

## 빌드 / 실행 명령

저장소 루트에서 (PowerShell은 `.\gradlew`, Git Bash는 `./gradlew`):

```bash
./gradlew bootRun     # 개발 서버 실행 (기본 http://localhost:8080)
./gradlew build       # 빌드 (테스트 포함)
./gradlew test        # 테스트만
```

## 협업 규칙

- **임의로 진행하지 말 것.** 코드 작성·의존성 추가 등은 먼저 방향을 합의한 뒤 진행한다.
- 학습 병행이 목적이므로, 가능하면 한 번에 쏟아내기보다 **단계적으로 설명하며** 진행한다.
- 커밋/푸시는 사용자가 직접 하거나, 요청이 있을 때만 수행한다.
