# DineOS (식당 운영 통합 시스템)

식당 체인을 위한 옴니채널 주문 통합 플랫폼 — 매장·배달·관리·홈페이지 All-in-One

---

## 📅 프로젝트 정보

| 항목 | 내용 |
|---|---|
| 기간 | 2025.10 ~ 진행 중 |
| 팀 구성 | 1인 (개인 프로젝트) |
| 역할 | 풀스택 개발자 |
| 형태 | 모노레포 (pnpm Workspace + Turborepo) |

---

## 🛠 기술 스택

- Frontend
    
    | 분류 | 기술 |
    |---|---|
    | Framework | Next.js 15 (App Router), React 19, TypeScript 5 |
    | Styling | Tailwind CSS 4, Shadcn UI |
    | State | Zustand 5 (클라이언트), TanStack Query 5 (서버) |
    | PWA / 앱 | Next-PWA, Capacitor 6 (iOS/Android) |
    | Test | Vitest, Testing Library |
    
- Backend
    
    | 분류 | 기술 |
    |---|---|
    | Framework | NestJS 10, TypeScript 5 |
    | ORM | Prisma 5 |
    | DB | PostgreSQL 14 (Supabase) |
    | Queue | pgmq (PostgreSQL 기반 메시지 큐) |
    | Cache / Rate Limit | Redis (ioredis) — Vercel 다중 인스턴스 Rate Limiting 공유 저장소 |
    | Security | Helmet.js, Rate Limiting, CORS, JWT |
    | Logger | Winston |
    | API Docs | Swagger |
    
- DevOps
    
    | 분류 | 기술 |
    |---|---|
    | Monorepo | pnpm Workspace + Turborepo |
    | Deployment | Vercel (Frontend + Backend Serverless) |
    | CI/CD | GitHub Actions |
    | Monitoring | Vercel Analytics, Sentry |
    

---

## 🏗 시스템 구조

**모노레포 4 Apps + 1 Backend + 4 Shared Packages**

```
apps/
├── table-order       # 매장 태블릿/QR 주문
├── delivery-customer # 배달 주문 (PWA + iOS/Android)
├── admin             # 주방 화면 + 관리자 대시보드
└── brand-website     # 브랜드 마케팅 홈페이지 (SSG)

backend/              # NestJS 통합 API (Vercel Serverless)

packages/
├── @order/shared     # 공통 타입, 유틸, 상수
├── @order/ui         # 공통 UI 컴포넌트 (Shadcn UI)
├── @order/order-core # 주문 비즈니스 로직 (프론트)
└── @order/config     # ESLint, TSConfig 공통 설정
```

- 1️⃣ Table Order (매장 태블릿/QR 주문)
    
    고객이 매장 내 태블릿 또는 본인 스마트폰(QR 스캔)으로 직접 주문하는 시스템
    
    **주요 기능**
    - 카테고리별 메뉴 브라우징 및 옵션 선택
    - 실시간 장바구니 (수량 조절, 삭제)
    - QR 스캔 → 테이블 번호 자동 설정 → 3초 카운트다운 후 주문
    - 직원 호출 버튼 → 관리자 화면 실시간 알림
    
    **기술 포인트**
    - Container/Presenter 패턴으로 UI ↔ 비즈니스 로직 분리
    - Zustand: 장바구니·테이블 번호 클라이언트 상태
    - TanStack Query: 메뉴·주문 서버 상태 캐싱 및 자동 갱신
    - MSW Mock API로 백엔드 없이 프론트 독립 개발
    
    **배포**: Vercel / 개발 포트 3000
    
- 2️⃣ Delivery Customer (배달 주문)
    
    웹 + iOS/Android 네이티브 앱으로 배달 주문하는 시스템. 하나의 코드베이스로 웹·앱 동시 지원.
    
    **주요 기능**
    - PWA (홈 화면 추가) + Capacitor 네이티브 앱 빌드
    - Toss Payments 결제 (카드, 카카오페이, 네이버페이)
    - 실시간 배달 상태 추적 + 푸시 알림 (ASSIGNED → PICKED_UP → DELIVERED)
    - 라이더 전화 연결
    
    **Native 플러그인 (12개)**
    Camera, Geolocation, Push Notifications, Status Bar, Haptics, Share, Browser, Toast, Network, Keyboard, App, Device
    
    **기술 포인트**
    - Next.js Static Export → Capacitor로 네이티브 앱 변환
    - 플랫폼 분기 처리 (`platform.isNative`)
    - Android: Windows 빌드 가능 / iOS: macOS + Xcode 필요
    
    **배포**: Vercel (웹) / Google Play Store (Android) / 개발 포트 3001
    
- 3️⃣ Admin Dashboard (관리자)
    
    주방 화면과 관리자 대시보드를 하나의 앱으로 제공
    
    **주요 기능**
    - 실시간 주문 알림 (Supabase Realtime) + 소리·Toast 알림
    - 주문 상태 관리 (접수 → 조리 중 → 완료)
    - 직원 호출 알림 수신
    - 매출 통계 및 차트 (일별/주별/월별)
    - 메뉴 관리 (등록·수정·품절 처리)
    - POS 동기화 실패 주문 수동 재시도 UI
    
    **기술 포인트**
    - Supabase Realtime (PostgreSQL LISTEN/NOTIFY, 지연 ~100ms)
    - Recharts 기반 매출 통계
    
    **배포**: Vercel / 개발 포트 3003
    
- 4️⃣ Brand Website (브랜드 홈페이지)
    
    브랜드 소개·마케팅용 정적 홈페이지
    
    **주요 기능**
    - 브랜드 스토리, 대표 메뉴 소개, 매장 위치 (지도 연동)
    - 이벤트·프로모션, 채용 정보
    
    **기술 포인트**
    - Next.js SSG (빌드 타임 정적 생성, CDN 캐싱)
    - SEO 최적화: Meta 태그, Open Graph, Sitemap, JSON-LD
    - Next.js Image로 이미지 최적화 + Lazy Loading
    
    **배포**: Vercel 정적 호스팅 / 개발 포트 3002
    
- 5️⃣ Backend API (NestJS)
    
    모든 프론트 앱이 사용하는 NestJS 통합 REST API 서버
    
    **주요 모듈**
    - MenusModule: 메뉴 CRUD, 카테고리, 옵션, 품절 처리
    - OrdersModule: 주문 생성·조회·상태 변경·통계
    - StoresModule: 다중 매장 지원 (menuManagementMode: TOSS_POS / ADMIN_DIRECT)
    - IntegrationsModule: Toss Order POS 연동, Toss Payments 결제
    - QueueModule: pgmq 기반 이벤트 큐 (발행·소비·재시도)
    - NotificationModule: 인앱 + 푸시 알림 (FCM)
    
    **주요 DB 모델**
    Menu, Category, Order (+ posSyncStatus), OrderItem, Store, User, Payment, QueueEventLog, NotificationLog
    
    **주요 API 엔드포인트**
    
    | Method | Path | 설명 |
    |---|---|---|
    | GET | /api/v1/stores/:storeId/menus | 메뉴 목록 |
    | POST | /api/v1/stores/:storeId/orders | 주문 생성 |
    | PATCH | /api/v1/stores/:storeId/orders/:id | 주문 상태 변경 |
    | PATCH | /api/v1/stores/:storeId/orders/:id/pos-sync/retry | POS 재전송 (관리자) |
    | GET | /api/v1/stores/:storeId | 매장 정보 |
    
    **배포**: Vercel Serverless (Node.js 20, 메모리 1024MB) / 개발 포트 4000
    

---

## 💡 기술적 도전 & 해결

- 1. 장애 격리 & Circuit Breaker
    
    POS 연동 장애 시 서버 전체로 오류가 전파되는 문제를 3중 방어 구조로 해결
    
    - Circuit Breaker 상세 (`pos.resilience.ts`)
        
        **라이브러리**: `opossum` v9.0.0
        
        **설정값**
        
        | 옵션 | 값 | 설명 |
        |---|---|---|
        | timeout | 3,000ms | 개별 요청 최대 대기 시간 |
        | errorThresholdPercentage | 50% | 실패율 초과 시 OPEN 전환 |
        | resetTimeout | 10,000ms | OPEN → HALF-OPEN 대기 시간 |
        | rollingCountTimeout | 10,000ms (기본값) | 실패율 집계 슬라이딩 윈도우 |
        | volumeThreshold | 0 (기본값) | 최소 요청 수 제한 없음 |
        
        **3가지 상태 전이**
        
        | 상태 | 전환 조건 | 동작 |
        |---|---|---|
        | 🟢 CLOSED | 기본 상태 | POS API 실제 호출 |
        | 🔴 OPEN | 최근 10초간 실패율 50% 초과 | 즉시 fallback, POS 호출 차단 |
        | 🟡 HALF-OPEN | OPEN 후 10초 경과 | 요청 1건 시도 → 성공 시 CLOSED 복귀 |
        
        **동작 흐름**
        ```
        breaker.fire(order)
          └─ CLOSED: MockPosService.sendOrder() 실제 호출
          └─ OPEN:   fallback() → return false (즉시 반환)
        ```
        
    - Queue 기반 Exponential Backoff 재시도 (`queue-consumer.service.ts`)
        
        POS 전송 실패 시 큐에 재발행하여 자동 재시도
        
        | 시도 횟수 | 대기 시간 |
        |---|---|
        | 1차 실패 | 10초 후 재시도 |
        | 2차 실패 | 30초 후 재시도 |
        | 3차 실패 | 60초 후 재시도 |
        | 4차 실패 | 3분 후 재시도 |
        | 5차 실패 | 5분 후 재시도 |
        | 5회 초과 | 영구 아카이브 (포기) |
        
        실패 이력은 Order 테이블의 `posSyncStatus`, `posSyncAttemptCount`, `posSyncLastError`로 추적
        
    - 전체 흐름 요약
        
        ```
        [프론트] 주문 생성
          → OrdersService: DB 저장 (status: PAID)
          → QueueService: publishOrderPaid() 이벤트 발행
        
        [QueueConsumer] handleOrderPaid()
          → TOSS_POS 매장이면 publishPosSendOrder() 발행
          → ADMIN_DIRECT 매장이면 POS 전송 skip
        
        [QueueConsumer] handlePosSendOrder()
          → ResilientPosService.sendOrder() ← Circuit Breaker
            → 성공: posSyncStatus = 'SENT'
            → 실패: posSyncStatus = 'FAILED' → 큐 재시도 스케줄
        ```
        
        **멱등성 보장**: 이벤트마다 `idempotencyKey` 부여 → `QueueEventLog`에서 `SUCCEEDED` 확인 후 재처리 스킵
        

- 2. 실시간 주문 처리
    
    - Realtime + Polling 이중화
        
        | 레이어 | 방식 | 역할 |
        |---|---|---|
        | Primary | Supabase Realtime (postgres_changes 구독) | 실시간 주문 수신 |
        | Fallback | 연결 실패 시 5초 후 자동 재구독 | Realtime 채널 장애 대응 |
        | Backup | 30초 간격 polling | Realtime 전체 장애 시 안전망 |
        
    - 이벤트 큐 구조 (pgmq)
        
        백엔드 내부 이벤트를 PostgreSQL 기반 큐로 비동기 처리
        
        | 이벤트 | 설명 |
        |---|---|
        | order.paid | 결제 완료 → POS 전송 + 알림 트리거 |
        | pos.send_order | POS 주문 전송 |
        | notification.send | 인앱 / 푸시 알림 발송 |
        | payment.reconcile | Toss Payments 결제 상태 정합성 확인 |
        | delivery.status_changed | 배달 상태 변경 → 고객/매장 알림 |
        

- 3. 보안
    
    - Rate Limiting / Helmet / CORS 상세
        
        **Rate Limiting (DDoS 방어)**
        - 1초당 10개 요청 제한
        - 1분당 100개 요청 제한
        - 15분당 1,000개 요청 제한
        - IP 기반 제한
        - **저장소**: `REDIS_URL` 설정 시 Redis 분산 저장소 사용 (Vercel 다중 인스턴스 간 공유), 미설정 시 인메모리 폴백
        
        **Helmet.js**
        - XSS, Clickjacking 방어
        - Content Security Policy 적용
        
        **CORS**: 허용된 도메인만 접근, Credentials 지원
        
        **Input Validation**: class-validator DTO 검증, Whitelist 모드
        
        **에러 처리**: HttpExceptionFilter 일관된 응답, 프로덕션 스택 트레이스 숨김
        

- 4. DevOps & 비용 절감
    
    - 인프라 비용 비교
        
        **기존 (NCP 클라우드)**
        - Server 2vCPU 4GB: 월 65,000원
        - Cloud DB PostgreSQL: 월 30,000원
        - **합계: 월 95,000원**
        
        **현재 (Vercel + Supabase)**
        - Vercel Hobby: 무료
        - Supabase Free: 무료
        - **합계: 월 0원 (100% 절감)**
        
        **확장 시나리오**
        
        | 규모 | 비용 |
        |---|---|
        | ~150개 테이블 | 무료 Tier |
        | ~280개 테이블 | Vercel Pro $20/월 |
        | ~300개 테이블+ | Vercel Pro + Supabase Pro $45/월 |
        
    - CI/CD 파이프라인
        
        - GitHub Actions: 자동 테스트 → 빌드 → 배포
        - git push만으로 프로덕션 배포 완료 (배포 시간 90% 단축)
        - Turborepo 빌드 캐싱으로 불필요한 재빌드 방지
        - Sentry 에러 모니터링, Vercel Analytics 성능 추적
        

---

## 📊 성과

- 기술적 성과
    
    1. **모노레포 구조 확립** — 4개 프론트 + 1개 백엔드를 단일 코드베이스로 관리, 공유 패키지로 중복 제거
    2. **3중 장애 격리** — Circuit Breaker + Queue Exponential Backoff + 관리자 수동 재시도로 POS 장애 대응
    3. **멱등성 보장** — idempotencyKey + QueueEventLog로 중복 이벤트 처리 차단
    4. **프로덕션 레벨 보안** — Rate Limiting, Helmet, Input Validation, JWT 인증 전체 적용
    5. **DevOps 자동화** — git push → 자동 배포, 배포 시간 90% 단축
    6. **비용 절감** — 월 95,000원 → 0원 (100% 절감)
    7. **테스트** — Vitest 기반 단위 테스트 24개 통과, MSW API 모킹
    
- 비즈니스 성과
    
    1. **실제 도입 준비 중** — 가족 식당 체인 (약 7개 매장), T-Order 대체 시 연간 수백만 원 절감 예상
    2. **옴니채널 지원** — 태블릿 주문, QR 주문, 배달 웹/앱 통합
    3. **확장 용이** — 매장 추가 시 공유 패키지로 모든 앱에 자동 반영
    

---

## 📈 향후 계획

- Admin Dashboard 고도화 — 메뉴 추천 AI, 재고 자동 관리
- iOS 앱 출시 — App Store 제출 (Mac 환경 필요)
- 추가 매장 확장 — 7개 → 15개, 프랜차이즈 시스템
- 슬랙 알림 연동 — Critical 에러 실시간 알림
