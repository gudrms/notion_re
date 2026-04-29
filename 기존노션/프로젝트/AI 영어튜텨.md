![대화.png](attachment:11fb5e5d-6f98-42d2-93c2-cdb27d305ee8:대화.png)

📅 기간 : 2025.03 ~

👥 팀 구성 : 1명

🎯 역할 : 풀스택 개발자

🛠 기술 스택

- **Frontend** : React 19, TypeScript, Vite
- **Backend** : Ruby on Rails 7 (API mode)
- **Database** : PostgreSQL 16
- **AI** : OpenAI GPT-4o-mini (LLM), Whisper (STT), TTS
- **Infra** : Docker Compose
- **Testing** : RSpec + FactoryBot (BE 84건), Vitest + React Testing Library (FE 52건)

📋 주요 업무

- AI 대화 플로우 구현 : 음성 녹음 → STT(Whisper) → LLM 응답(GPT-4o-mini, SSE 스트리밍) → TTS 음성 재생
- 멤버십 시스템 설계 : features 배열 기반 유연한 권한 관리 (learning, conversation, analysis)
- 어드민 기능 구현 : 멤버십 플랜 CRUD, 유저 멤버십 부여/삭제
- Mock 결제 시스템 : MockPgService 기반 결제 + 멤버십 생성 트랜잭션 처리
- 프론트엔드/백엔드 테스트 코드 작성 : 총 136건 (BE 84 + FE 52), 전체 통과

![메인 화면 캡쳐](attachment:b0888bdb-1b85-467d-a83e-f740ac554cd8:홈화면.png)

---

### 📖 내용

- OpenAI API(GPT-4o-mini, Whisper, TTS)를 활용한 **음성 기반 영어 대화 학습** 앱
- 멤버십 권한을 **features 배열**(PostgreSQL string array)로 관리하여 다양한 플랜을 유연하게 정의
- LLM 응답을 **SSE(Server-Sent Events) 스트리밍**으로 실시간 전송하여 체감 지연 최소화
- 텍스트 먼저 표시 → TTS 비동기 재생으로 **사용자 체감 속도 향상**
- Docker Compose로 PostgreSQL + Rails 백엔드를 컨테이너화, `npm run dev` 한 줄로 전체 실행

---

### 🙋‍♂️ 역할

- 전체 시스템 아키텍처 설계 및 기술 스택 선정
- Rails 7 API 백엔드 개발 (멤버십 관리, AI 대화, 결제)
- React 19 + TypeScript 프론트엔드 개발 (홈, 대화, 어드민 3개 페이지)
- OpenAI API 연동 (GPT-4o-mini, Whisper STT, TTS)
- Docker Compose 인프라 구성
- 프론트엔드/백엔드 테스트 코드 작성 (136건)

---

### 🎯 결과 및 성과

- **실시간 AI 대화** : SSE 스트리밍으로 LLM 응답을 실시간 표시, 체감 지연 최소화
- **유연한 멤버십** : features 배열 기반 설계로 새로운 플랜 추가 시 코드 변경 불필요
- **높은 테스트 커버리지** : BE 84건 + FE 52건 = 총 136건 전체 통과
- **원커맨드 실행** : `npm run dev`로 Docker 백엔드 + 프론트엔드 동시 기동
- **오남용 방지** : 녹음 시간 60초 제한 + 타이머 UI로 과도한 API 호출 차단

---

## 🏗 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────────┐
│                      Frontend (React 19 + TypeScript + Vite)    │
│                           localhost:5173                        │
│  ┌───────────┐  ┌──────────────────┐  ┌──────────────────────┐ │
│  │ HomePage   │  │ ConversationPage │  │    AdminPage         │ │
│  │ 멤버십 조회│  │ AI 음성 대화     │  │ 플랜 CRUD            │ │
│  │ 구매       │  │ Waveform 시각화  │  │ 멤버십 부여/삭제     │ │
│  └───────────┘  └──────────────────┘  └──────────────────────┘ │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTP/REST + SSE
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Backend (Ruby on Rails 7 API mode)             │
│                        localhost:3000                           │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────────┐ │
│  │ Membership   │  │ Conversation │  │ Admin Controllers     │ │
│  │ Controller   │  │ Controller   │  │ (Plans, Memberships)  │ │
│  └──────┬───────┘  └──────┬───────┘  └───────────────────────┘ │
│         │                  │                                    │
│  ┌──────▼───────┐  ┌──────▼───────┐  ┌───────────────────────┐ │
│  │ MockPgService│  │OpenAI Service│  │   Models / Services   │ │
│  │ (결제 Mock)  │  │(GPT/STT/TTS) │  │                       │ │
│  └──────────────┘  └──────────────┘  └───────────────────────┘ │
└────────────────────────────┬────────────────────────────────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌──────────────┐ ┌───────────┐ ┌────────────────┐
     │ PostgreSQL 16│ │ OpenAI API│ │  Docker Compose │
     │ (멤버십, 유저│ │ GPT-4o-mini│ │  (인프라 관리)  │
     │  결제 데이터)│ │ Whisper   │ │                 │
     │              │ │ TTS       │ │                 │
     └──────────────┘ └───────────┘ └────────────────┘
```

---

## 🎙 AI 대화 플로우

```
[유저] 마이크 버튼 클릭 → MediaRecorder 녹음 시작
     │                    (Waveform 시각화 + 60초 타이머)
     ▼
[답변완료 클릭] → 녹음 중지 → audio/webm Blob 생성
     │
     ▼
POST /api/conversations/transcribe
     → OpenAI Whisper STT → 유저 텍스트 변환 → 채팅 UI에 표시
     │
     ▼
POST /api/conversations/reply (SSE 스트리밍)
     → GPT-4o-mini → AI 응답 텍스트 실시간 표시
     │
     ▼
POST /api/conversations/synthesize
     → OpenAI TTS → 음성 자동 재생
```

---

## 💾 데이터 모델

```
User (name, email, role: user/admin)
  │
  ├── UserMembership (starts_at, expires_at, status: active/expired/cancelled)
  │     └── MembershipPlan (name, duration_days, features: string[], price)
  │
  └── Payment (amount, status: pending/completed/failed, pg_transaction_id)
        └── MembershipPlan
```

- **멤버십 만료** : `expires_at < Time.current` 시점에 실시간 체크 (배치 불필요)
- **권한 체크** : `User#has_feature?("conversation")` → 활성 멤버십의 features 배열 확인
- **features 배열** : `["learning", "conversation", "analysis"]` 조합으로 다양한 플랜 정의

---

## 🎬 시연 영상

> 📹 아
>

[지형근_과제_시연영상.mp4](attachment:3b64d967-3912-4d4b-8415-158ca3ceb95a:지형근_과제_시연영상.mp4)

---

## 📸 주요 화면 캡쳐

### 1. 홈 화면 — 멤버십 조회 및 구매

![홈 화면](attachment:f69eba1b-74eb-4e6e-9fb3-c9ee160ca454:홈_구매.png)

### 2. AI 대화 화면 — 음성 녹음 및 실시간 응답

![대화 화면](attachment:dbc504e8-d331-4cb8-962d-d475bba71420:대화.png)

### 3. 녹음 중 Waveform — 오디오 인식 UX 피드백

![Waveform](attachment:1dcecd79-5321-48bf-83f1-93c0ab8517d1:대화_구매.png)

### 4. 어드민 화면 — 멤버십 플랜 관리 (CRUD)

![어드민 화면](attachment:86b88ee6-d698-4c19-815a-c582f0a81a62:관리자화면.png)

---

## ✅ 테스트

### Backend (RSpec) — 84 tests, 0 failures ✅

| 테스트 영역 | 테스트 내용 |
| --- | --- |
| **Model** | User, MembershipPlan, UserMembership, Payment (associations, validations, 비즈니스 로직) |
| **Request** | 전체 API 엔드포인트 통합 테스트 (인증, 권한, 에러 처리) |
| **Service** | MockPgService (결제 + 멤버십 생성 트랜잭션) |
| **Mock** | OpenAI API는 `instance_double`로 mock 처리 |

### Frontend (Vitest) — 52 tests, 0 failures ✅

| 테스트 영역 | 테스트 내용 |
| --- | --- |
| **API Client** | fetch mock으로 전체 엔드포인트 호출/에러 처리/SSE 스트리밍 검증 |
| **HomePage** | 로딩, 멤버십 상태별 UI 분기, 구매 플로우, conversation 권한 체크 |
| **ConversationPage** | 멤버십 가드 (리다이렉트), AI 인사말, 채팅 UI |
| **AdminPage** | 플랜 CRUD (목록/생성/수정/삭제), 에러 처리 |

---

## 📋 API 엔드포인트

### Public

| Method | Path | 설명 |
| --- | --- | --- |
| GET | `/api/membership_plans` | 구매 가능 플랜 목록 |

### User (X-User-Id 필요)

| Method | Path | 설명 |
| --- | --- | --- |
| GET | `/api/memberships` | 내 멤버십 조회 |
| POST | `/api/purchases` | 멤버십 구매 (Mock PG) |
| POST | `/api/conversations/start` | AI 대화 시작 |
| POST | `/api/conversations/reply` | AI 응답 요청 (SSE 스트리밍) |
| POST | `/api/conversations/transcribe` | 음성→텍스트 (STT) |
| POST | `/api/conversations/synthesize` | 텍스트→음성 (TTS) |

### Admin (role: admin 필요)

| Method | Path | 설명 |
| --- | --- | --- |
| GET | `/api/admin/membership_plans` | 플랜 목록 |
| POST | `/api/admin/membership_plans` | 플랜 생성 |
| PATCH | `/api/admin/membership_plans/:id` | 플랜 수정 |
| DELETE | `/api/admin/membership_plans/:id` | 플랜 삭제 |
| POST | `/api/admin/user_memberships` | 멤버십 부여 |
| DELETE | `/api/admin/user_memberships/:id` | 멤버십 삭제 |

---

## 🗂 프로젝트 구조

```
ring/
├── backend/                    # Rails 7 API
│   ├── app/
│   │   ├── controllers/api/   # API 컨트롤러 (멤버십, 대화, 어드민)
│   │   ├── models/            # ActiveRecord 모델 (User, MembershipPlan, etc.)
│   │   └── services/          # OpenaiService, MockPgService
│   ├── spec/                  # RSpec 테스트 (84건)
│   │   ├── models/
│   │   ├── requests/
│   │   ├── services/
│   │   └── factories/
│   └── Dockerfile
├── frontend/                   # React 19 + TypeScript + Vite
│   ├── src/
│   │   ├── api/client.ts      # API 클라이언트
│   │   ├── pages/             # HomePage, ConversationPage, AdminPage
│   │   ├── hooks/             # useAudioRecorder (녹음 훅)
│   │   ├── types/             # TypeScript 인터페이스
│   │   └── test/              # Vitest 테스트 (52건)
│   └── package.json
├── docs/                       # 설계 문서
│   ├── requirements.md         # 요구사항 정리
│   ├── tech_spec.md            # 기술 설계 명세
│   └── checklist.md            # 구현 체크리스트
├── docker-compose.yml          # PostgreSQL + Rails 컨테이너
├── package.json                # 루트 스크립트 (빌드/테스트 통합)
└── README.md
```

---

## 📝 회고 및 배운 점

### 기술적 도전

- **SSE 스트리밍** : Rails의 `ActionController::Live`로 SSE 구현 시 커넥션 관리와 에러 핸들링에 주의 필요
- **MediaRecorder API** : 브라우저별 코덱 지원 차이 (`audio/webm`) 대응
- **TTS 비동기 처리** : 텍스트 먼저 표시 → TTS 별도 호출로 체감 속도를 개선하는 UI/UX 설계

### 핵심 교훈

- 멤버십 권한을 **features 배열**로 설계하면 새로운 플랜 추가 시 코드 변경 없이 대응 가능
- SSE 스트리밍은 WebSocket보다 구현이 단순하면서 **LLM 실시간 응답 표시에 최적**
- Docker Compose + 루트 `package.json` 스크립트로 **개발 환경 셋업을 원커맨드로 단순화**하는 것이 팀 생산성에 기여