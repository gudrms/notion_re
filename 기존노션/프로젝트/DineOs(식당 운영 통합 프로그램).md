# **DineOS**(식당 운영 통합 시스템)

식당 체인을 위한 옴니채널 주문 통합 플랫폼 (매장·배달·관리·홈페이지 All-in-One)

## 📅 프로젝트 정보

**기간**: 2025.10 ~ 진행 중  

**팀 구성**: 1명 (개인 프로젝트)  

**역할**: 풀스택 개발자  

## 🛠 기술 스택

### Frontend

- **Framework**: Next.js 16 (App Router), React 19, TypeScript 5
- **Styling**: Tailwind CSS 4, Shadcn UI
- **State Management**: Zustand 5, TanStack Query 5
- **PWA**: Next-PWA, Capacitor 6 (iOS/Android 네이티브 앱)
- **Testing**: Vitest, Testing Library

### Backend

- **Framework**: NestJS 10, TypeScript 5
- **ORM**: Prisma 5
- **Database**: PostgreSQL 14 (Supabase)
- **Security**: Helmet.js, Rate Limiting, CORS, JWT
- **Logger**: Winston
- **API Docs**: Swagger

### DevOps

- **Monorepo**: pnpm Workspace + Turborepo
- **Deployment**: Vercel (Frontend + Backend Serverless)
- **CI/CD**: GitHub Actions
- **Monitoring**: Vercel Analytics, Sentry

## 📖 프로젝트 개요

식당 체인용 종합 주문 시스템을 모노레포 구조로 개발한 풀스택 프로젝트입니다. 태블릿 주문, 배달 주문, 관리자 대시보드, 브랜드 홈페이지 등 4개 앱을 하나의 코드베이스에서 관리하며, 공통 로직과 UI 컴포넌트를 패키지로 공유합니다.

### 🏗️ 모노레포 구조

**Apps (4개)**

- `table-order`: 매장 내 태블릿/QR 주문 시스템
- `delivery-customer`: 배달 주문 (PWA + iOS/Android 앱)
- `brand-website`: 브랜드 마케팅 홈페이지 (SSG)
- `admin`: 주방 화면 + 관리자 대시보드

**Backend (1개)**

- `backend`: NestJS 통합 API 서버 (Vercel Serverless)

**Shared Packages (4개)**

- `@order/shared`: 공통 타입, 유틸, 상수
- `@order/ui`: 공통 UI 컴포넌트 (Shadcn UI)
- `@order/order-core`: 주문 비즈니스 로직 (프론트엔드)
- `@order/config`: ESLint, TSConfig 공통 설정

## 📋 주요 업무 및 성과

### 🎯 풀스택 개발

- 모노레포 프로젝트 설계 및 구축 (pnpm workspace + Turborepo)
- 4개 프론트엔드 앱 + 1개 백엔드 API 개발
- Container/Presenter 패턴 적용으로 UI 컴포넌트와 비즈니스 로직 분리
- 공유 패키지를 통한 코드 재사용성 및 유지보수성 향상

### 🔗 Toss Order 연동

- OKPOS에서 Toss Order로 POS 시스템 변경
- 매장 주문 → Toss Order → POS 시스템 자동 전송
- Circuit Breaker 패턴 및 Retry 로직 구현 (장애 대응)
    
    ### **1. 서킷브레이커 (`pos.resilience.ts`)**
    
    **라이브러리:** `opossum` (v9.0.0)
    
    **설정:**
    
    - **timeout:** 3초 — 개별 요청 제한 시간
    - **errorThresholdPercentage:** 50% — 실패율이 50% 이상이면 회로 개방
    - **resetTimeout:** 10초 — OPEN 후 HALF_OPEN으로 전환 대기 시간
    
    **상태 전이:**
    
    | **상태** | **설명** |
    | --- | --- |
    | **CLOSED** (🟢) | 정상 운영. 요청이 POS 프로바이더에 전달됨 |
    | **OPEN** (🔴) | 모든 요청 차단, fallback으로 `false` 반환 |
    | **HALF_OPEN** (🟡) | 10초 후 테스트 요청 1건 허용. 성공→CLOSED, 실패→OPEN |
    
    `breaker.fire(order)`로 호출되며, POS 주문 전송 함수를 래핑합니다.
    
    ---
    
    ### **2. 재시도 로직 — 주문 상태 업데이트 (`order.ts`)**
    
    `최대 3회 시도, 선형 백오프 (attempt × 1초)
    시도 1 실패 → 1초 대기 → 시도 2 실패 → 2초 대기 → 시도 3`
    
    - **트리거:** 네트워크 오류 또는 HTTP 비정상 응답
    - 3회 모두 실패하면 에러 로그만 남기고 종료
    
    ---
    
    ### **3. 요청 타임아웃 (`api/client.ts`)**
    
    - **기본값:** 10초
    - `AbortController` 기반으로 요청 시간 초과 시 `ApiClientError(408, 'TIMEOUT')` 발생
    - GET/POST/PATCH/PUT/DELETE 모든 메서드에 적용
    
    ---
    
    ### **4. Realtime + Polling 이중화 (`realtime.ts`)**
    
    - **Primary:** Supabase Realtime (postgres_changes 구독)
    - **Fallback:** 연결 실패(`CHANNEL_ERROR`, `TIMED_OUT`) 시 5초 후 재구독
    - **Backup:** 30초 간격 polling (`setInterval`)으로 Realtime 장애 시에도 주문 처리 보장
    
    ---
    
    ### **5. 중복 처리 방지**
    
    | **위치** | **방식** |
    | --- | --- |
    | `order.ts` — `processingOrders` Set | 동시 요청 시 같은 주문 중복 처리 방지 (in-memory Set) |
    | `pos.controller.ts` — 멱등성 검사 | 이미 동일 상태인 주문은 재처리하지 않고 그대로 반환 |
    
    ---
    
    ### **6. React Query 재시도 (`queryClient.ts`)**
    
    - **Query:** 1회 재시도
    - **Mutation:** 재시도 없음 (멱등성 보장 불가)
    - **staleTime:** 5분 / **gcTime:** 10분
    - **refetchOnReconnect:** 활성화 (네트워크 복구 시 자동 재요청)
    
    ---
    
    ### **전체 흐름 요약**
    
    `[클라이언트 요청]
      → fetchWithTimeout (10s 타임아웃)
      → React Query (query 1회 재시도)
      
    [POS 주문 전송]
      → Circuit Breaker (50% 에러율 → 차단)
        → 성공 시: 정상 처리
        → 실패 시: fallback(false) 반환
        
    [주문 상태 업데이트]
      → 최대 3회 재시도 (선형 백오프)
      
    [주문 수신]
      → Realtime (primary) + Polling 30s (backup)
      → 중복 방지: Set + 멱등성 검사`
    
    서킷브레이커는 POS 연동 계층에서 장애 전파를 차단하고, 재시도 로직은 일시적 네트워크 오류에 대응하며, polling은 Realtime 채널 장애 시 안전망 역할을 합니다.
    
- 주문 실패 시 자동 재시도 및 에러 로그 저장

### 🔒 보안 및 성능 최적화

- Rate Limiting 구현 (DDoS 방어: 1초당 10개, 1분당 100개, 15분당 1000개)
- Helmet.js로 보안 헤더 설정 (XSS, CSRF 방어)
- CORS 정책 적용 및 Input Validation
- Winston Logger를 통한 에러 추적 및 로그 관리
- Supabase Realtime으로 실시간 주문 알림 (WebSocket 대체)

### 🧪 테스트 및 품질 관리

- Vitest 기반 단위 테스트 작성 (24개 테스트 통과)
- MSW를 활용한 API 모킹으로 백엔드 독립 개발
- TypeScript 엄격 모드로 타입 안전성 확보

### 🚀 DevOps 및 배포

- Vercel Serverless 배포로 서버 관리 부담 제거
- GitHub Actions CI/CD 구축 (자동 테스트, 빌드, 배포)
- git push만으로 프로덕션 배포 완료 (배포 시간 90% 단축)

## 🎯 핵심 기능

### 1. 다중 주문 채널

- 매장 내 태블릿 주문
- QR 코드 스캔 후 고객 스마트폰 주문
- 배달 주문 (웹/앱)

### 2. 실시간 상태 관리

- Zustand로 클라이언트 상태 관리 (장바구니, 주문)
- TanStack Query로 서버 상태 관리 (캐싱, 동기화)
- Supabase Realtime으로 주방 화면 실시간 업데이트

### 3. 네이티브 앱 기능 (배달 주문)

- PWA + Capacitor로 iOS/Android 앱 빌드
- 12개 Native 플러그인: 카메라, GPS, 푸시 알림, 진동 등
- 실시간 배달 추적 및 라이더 전화

### 4. 관리자 기능

- 주방 화면: 실시간 주문 알림 및 조리 상태 관리
- 대시보드: 매출 통계, 메뉴 관리, 재고 관리

### 5. 개별 서비스 상세

- 1️⃣ Table Order (테이블 주문)
    
    ### 📱 개요
    
    고객이 매장 내 태블릿 또는 자신의 스마트폰(QR 스캔)으로 직접 메뉴를 주문하는 시스템입니다.
    
    ### 🎯 주요 기능
    
    **메뉴 조회 및 주문**
    
    - 카테고리별 메뉴 브라우징
    - 메뉴 상세 정보 및 옵션 선택
    - 실시간 장바구니 관리 (수량 조절, 삭제)
    - 주문 생성 및 주문번호 표시
    
    **QR 코드 주문**
    
    - 테이블 QR 코드 스캔 → 자동으로 테이블 번호 설정
    - 고객 스마트폰으로 주문 가능
    - 3초 카운트다운 후 자동 확인
    
    **직원 호출**
    
    - 테이블에서 직원 호출 버튼
    - 관리자 화면으로 실시간 알림
    
    ### 🏗️ 기술적 특징
    
    **Container/Presenter 패턴**
    
    - UI 컴포넌트(Presenter)와 비즈니스 로직(Container) 분리
    - 재사용 가능한 컴포넌트 설계
    - 테스트 용이성 향상
    
    **상태 관리**
    
    - Zustand: 장바구니, 테이블 번호 등 클라이언트 상태
    - TanStack Query: 메뉴, 주문 등 서버 상태 (캐싱, 자동 갱신)
    
    **MSW Mock API**
    
    - 백엔드 없이도 프론트엔드 개발 가능
    - 24개 단위 테스트 통과 (Vitest + Testing Library)
    
    ### 📦 배포
    
    - **URL**: https://order-front-frontend.vercel.app
    - **환경**: Vercel Edge Network (전세계 CDN)
    - **포트**: 3000 (개발 서버)
- 2️⃣ Delivery Customer (배달 주문)
    
    ### 🚚 개요
    
    고객이 웹 또는 네이티브 앱(iOS/Android)으로 배달 주문을 하는 시스템입니다. PWA와 Capacitor를 활용하여 하나의 코드베이스로 웹과 앱을 모두 지원합니다.
    
    ### 🎯 주요 기능
    
    **웹 & 앱 동시 지원**
    
    - PWA: 브라우저에서 접속, "홈 화면에 추가" 지원
    - Capacitor: iOS/Android 네이티브 앱 빌드 가능
    - 하나의 코드베이스로 웹과 앱 모두 관리
    
    **12개 Native 플러그인**
    
    1. **Camera**: 사진 촬영, 갤러리에서 선택 (리뷰 작성)
    2. **Geolocation**: GPS 현재 위치, 배달 주소 자동 입력
    3. **Push Notifications**: 주문 상태 변경 알림
    4. **Status Bar**: 다크 모드 지원
    5. **Haptics**: 진동 피드백
    6. **Share**: 앱 공유 기능
    7. **Browser**: 외부 링크 열기
    8. **Toast**: 네이티브 토스트 메시지
    9. **Network**: 인터넷 연결 상태 확인
    10. **Keyboard**: 키보드 제어
    11. **App**: 앱 정보 조회
    12. **Device**: 디바이스 정보 조회
    
    **실시간 배달 추적**
    
    - 주문 상태 실시간 업데이트
    - 배달 예상 시간 표시
    - 라이더 전화 연결
    
    **결제 시스템**
    
    - Toss Payments SDK 연동
    - 카드, 카카오페이, 네이버페이 등 다양한 결제 수단
    
    ### 🏗️ 기술적 특징
    
    **Capacitor 네이티브 앱**
    
    - Next.js Static Export를 네이티브 앱으로 변환
    - Android: Windows에서도 빌드 가능
    - iOS: macOS + Xcode 필요
    
    **플랫폼별 분기 처리**
    
    ```tsx
    if (platform.isNative) {
      // 네이티브 앱 전용 기능 (카메라, GPS 등)
      await takePicture();
    } else {
      // 웹 브라우저 전용 기능
      navigator.geolocation.getCurrentPosition();
    }
    ```
    
    **개발 워크플로우**
    
    - 웹 개발: `pnpm dev` (빠른 개발)
    - 네이티브 테스트: `pnpm cap:sync && pnpm android`
    
    ### 📦 배포
    
    - **웹**: Vercel 자동 배포
    - **Android**: Google Play Store (APK/AAB)
    - **iOS**: Apple App Store (향후 예정, Mac 필요)
    - **포트**: 3001 (개발 서버)
- 3️⃣ Admin Dashboard (관리자)
    
    ### 👨‍🍳 개요
    
    주방 화면과 관리자 대시보드를 제공하는 시스템입니다.
    
    ### 🎯 주요 기능
    
    **주방 화면**
    
    - 실시간 주문 알림 (Supabase Realtime)
    - 주문 상태 관리 (접수, 조리 중, 완료)
    - 소리 재생 및 Toast 알림
    - 직원 호출 알림
    
    **관리자 대시보드**
    
    - 매출 통계 및 차트
    - 메뉴 관리 (등록, 수정, 품절 처리)
    - 매장 설정 (영업 시간, 배달 지역 등)
    - 재고 관리
    
    **주문 내역 관리**
    
    - 전체 주문 내역 조회
    - 주문 상태별 필터링
    - 주문 상세 정보 확인
    
    ### 🏗️ 기술적 특징
    
    **Realtime Dashboard**
    
    - Supabase Realtime으로 주문 상태 자동 업데이트
    - WebSocket 없이 PostgreSQL LISTEN/NOTIFY 활용
    - 지연시간 ~100ms
    
    **통계 및 차트**
    
    - Recharts 라이브러리 활용
    - 일별/주별/월별 매출 통계
    - 인기 메뉴 분석
    
    ### 📦 배포
    
    - **환경**: Vercel
    - **포트**: 3003 (개발 서버)
- 4️⃣ Brand Website (브랜드 홈페이지)
    
    ### 🎨 개요
    
    브랜드 소개 및 마케팅을 위한 정적 홈페이지입니다.
    
    ### 🎯 주요 기능
    
    **브랜드 소개**
    
    - 브랜드 스토리 및 비전
    - 매장 분위기 사진
    
    **메뉴 소개**
    
    - 대표 메뉴 소개
    - 메뉴 카테고리별 분류
    - 고화질 메뉴 사진
    
    **매장 정보**
    
    - 전국 매장 위치 (지도 연동)
    - 영업 시간 및 연락처
    - 오시는 길 안내
    
    **이벤트 및 프로모션**
    
    - 진행 중인 이벤트 안내
    - 할인 쿠폰 제공
    
    **채용 정보**
    
    - 채용 공고
    - 지원 방법 안내
    
    ### 🏗️ 기술적 특징
    
    **Static Site Generation (SSG)**
    
    - Next.js SSG로 정적 사이트 생성
    - 빌드 타임에 모든 페이지 생성
    - CDN 캐싱으로 초고속 로딩
    
    **SEO 최적화**
    
    - Meta 태그 최적화
    - Open Graph 설정
    - Sitemap 자동 생성
    - 구조화된 데이터 (JSON-LD)
    
    **성능 최적화**
    
    - 이미지 최적화 (Next.js Image)
    - 코드 스플리팅
    - Lazy Loading
    
    ### 📦 배포
    
    - **환경**: Vercel (정적 호스팅)
    - **포트**: 3002 (개발 서버)
    - **참고**: https://www.yupdduk.com/
- 5️⃣ Backend API (백엔드)
    
    ### ⚙️ 개요
    
    NestJS 기반 통합 API 서버로, 모든 프론트엔드 앱에서 사용하는 RESTful API를 제공합니다.
    
    ### 🎯 주요 기능
    
    **메뉴 관리 (MenusModule)**
    
    - 메뉴 CRUD (등록, 조회, 수정, 삭제)
    - 카테고리별 메뉴 조회
    - 품절 처리
    - 메뉴 옵션 관리
    
    **주문 관리 (OrdersModule)**
    
    - 주문 생성 및 조회
    - 주문 상태 변경 (접수, 조리 중, 완료, 취소)
    - 주문 내역 조회 (고객별, 매장별)
    - 주문 통계
    
    **매장 관리 (StoresModule)**
    
    - 매장 정보 조회
    - 다중 매장 지원 (storeType, branchId)
    - 영업 시간 설정
    
    **Toss Order 연동 (IntegrationsModule)**
    
    - 주문 생성 시 Toss Order API 자동 호출
    - Circuit Breaker 패턴으로 장애 대응
    - axios-retry로 재시도 로직 (최대 3회)
    - 실패 시 `failed_pos_orders` 테이블에 저장
    
    **에러 로그 (ErrorLogsModule)**
    
    - Winston Logger로 에러 추적
    - Critical 에러는 DB에 저장
    - 슬랙 알림 연동 (향후 예정)
    
    ### 🔒 보안 기능
    
    **1. Rate Limiting (DDoS 방어)**
    
    - 1초당 10개 요청 제한
    - 1분당 100개 요청 제한
    - 15분당 1000개 요청 제한
    - IP 기반 제한
    
    **2. Helmet.js (보안 헤더)**
    
    - XSS 방어
    - Clickjacking 방어
    - Content Security Policy
    
    **3. CORS 정책**
    
    - 허용된 도메인만 접근 가능
    - Credentials 지원
    
    **4. Input Validation**
    
    - class-validator로 DTO 검증
    - Whitelist 모드 (허용된 필드만 허용)
    - Transform 자동 적용
    
    **5. 에러 처리**
    
    - HttpExceptionFilter로 일관된 에러 응답
    - 프로덕션에서 스택 트레이스 숨김
    - Winston Logger로 에러 로그 저장
    
    ### 🗄️ Database (Prisma)
    
    **주요 모델**
    
    - Menu: 메뉴 정보
    - Category: 카테고리
    - Order: 주문
    - OrderItem: 주문 항목
    - Table: 테이블
    - Store: 매장
    - User: 사용자
    - ErrorLog: 에러 로그
    - FailedPosOrder: POS 전송 실패 주문
    
    **특징**
    
    - PostgreSQL 14 (Supabase)
    - JSONB 지원 (주문 옵션 저장)
    - Full-text Search
    - Foreign Keys 및 Index 최적화
    
    ### 📡 주요 API 엔드포인트
    
    **메뉴**
    
    - `GET /api/v1/stores/:storeType/:branchId/menus` - 메뉴 목록
    - `GET /api/v1/stores/:storeType/:branchId/menus/:id` - 메뉴 상세
    
    **주문**
    
    - `POST /api/v1/stores/:storeType/:branchId/orders` - 주문 생성
    - `GET /api/v1/stores/:storeType/:branchId/orders/:orderId` - 주문 조회
    - `PATCH /api/v1/stores/:storeType/:branchId/orders/:orderId` - 주문 상태 변경
    
    **매장**
    
    - `GET /api/v1/stores/:storeType/:branchId` - 매장 정보
    
    ### 📦 배포
    
    - **환경**: Vercel Serverless Functions
    - **포트**: 4000 (개발 서버)
    - **Swagger**: http://localhost:4000/api/docs
    - **런타임**: Node.js 20.x
    - **메모리**: 1024MB
    - **실행 시간**: 10초 (Hobby), 60초 (Pro)

## 🏗️ 인프라 및 아키텍처

### Vercel + Supabase 스택

**Vercel (Frontend + Backend)**

- Edge Network로 전세계 CDN 제공
- Serverless Functions로 백엔드 배포
- git push만으로 자동 배포
- 무료 Tier: 100GB 대역폭/월

**Supabase (Database + Realtime + Storage)**

- PostgreSQL 14 (500MB)
- Realtime: PostgreSQL LISTEN/NOTIFY
- Storage: S3 호환 (1GB)
- 무료 Tier로 70개 테이블 완전 지원

### 비용 분석

**기존 (NCP 클라우드)**

- Server: 2vCPU, 4GB → 월 65,000원
- Cloud DB: PostgreSQL → 월 30,000원
- **총 월 95,000원**

**변경 (Vercel + Supabase)**

- Vercel Hobby: 무료 (100GB 대역폭)
- Supabase Free: 무료 (500MB DB, 2GB 대역폭)
- **총 월 0원 (100% 절감)**

**확장 시나리오**

- 70개 테이블 (7개 매장): 무료 Tier ✅
- 150개 테이블: 무료 Tier ✅
- 280개 테이블: Vercel Pro 필요 (월 $20)
- 300개 테이블: Vercel Pro + Supabase Pro (월 $45)

## 📊 성과 및 결과

### ✅ 기술적 성과

1. **모노레포 구조 확립**
    - 4개 프론트엔드 + 1개 백엔드를 하나의 코드베이스로 관리
    - 공유 패키지로 코드 재사용성 극대화
    - 유지보수 시간 50% 단축 예상
2. **Container/Presenter 패턴**
    - UI 컴포넌트 재사용성 향상
    - 테스트 용이성 증가 (24개 테스트 통과)
    - 비즈니스 로직과 UI 분리로 유지보수성 향상
3. **프로덕션 레벨 보안**
    - Rate Limiting으로 DDoS 방어
    - Helmet.js로 XSS, Clickjacking 방어
    - Input Validation으로 SQL Injection 방어
    - Winston Logger로 에러 추적 및 분석
4. **DevOps 자동화**
    - git push만으로 자동 배포 (배포 시간 90% 단축)
    - GitHub Actions CI/CD 구축
    - Vercel Analytics로 성능 모니터링
5. **비용 절감**
    - 월 95,000원 → 0원 (100% 절감)
    - 서버 관리 부담 제거 (Serverless)
    - 280개 테이블까지 무료 Tier로 확장 가능

### 🚀 비즈니스 성과

1. **실제 비즈니스 적용 가능**
    - 가족 식당 체인 (약 7개 매장) 도입 준비 중
    - T-Order 대체로 연간 수백만 원 비용 절감 예상
2. **다중 채널 지원**
    - 태블릿 주문 (매장 내)
    - QR 주문 (고객 스마트폰)
    - 배달 주문 (웹/앱)
3. **확장 가능한 구조**
    - 새로운 매장 추가 용이 (다중 매장 지원)
    - 새로운 기능 추가 시 모든 앱에 자동 반영 (공유 패키지)

### 📈 향후 계획

1. **Toss Order 연동 완료**
    - POS 시스템 실시간 연동
    - 주문 자동 전송 및 상태 동기화
2. **결제 시스템 통합**
    - Toss Payments 카드 결제
    - 카카오페이, 네이버페이 연동 예정
3. **Admin Dashboard 고도화**
    - 실시간 매출 대시보드
    - 메뉴 추천 AI
    - 재고 자동 관리
4. **iOS 앱 출시**
    - Android 앱 이후 iOS 앱 개발 (Mac 환경 필요)
    - App Store 제출
5. **추가 매장 확장**
    - 7개 매장 → 15개 매장 확장 계획
    - 프랜차이즈 시스템 구축

### 🎓 학습 내용

1. **모노레포 설계**
    - pnpm workspace와 Turborepo 활용
    - 공유 패키지 설계 및 관리
    - 빌드 캐싱 최적화
2. **Vercel Serverless**
    - Edge Functions vs Serverless Functions 차이
    - Cold Start 최적화
    - 환경변수 관리
3. **Supabase Realtime**
    - PostgreSQL LISTEN/NOTIFY 활용
    - WebSocket 없이 실시간 통신
    - 백엔드 코드 불필요
4. **Capacitor 네이티브 앱**
    - 웹 기술로 네이티브 앱 개발
    - 플랫폼별 분기 처리
    - Native 플러그인 활용
5. **보안 및 성능**
    - Rate Limiting 전략
    - Circuit Breaker 패턴
    - Retry 로직 구현