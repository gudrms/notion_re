사내 문서 검색기 (Local Document Searcher)

![image.png](attachment:5c2608b6-33f5-4715-8965-d37605a01732:image.png)

1. 사내 문서 검색기 개발 (PoC)

📅 기간 : 2025.11 ~ (PoC 완료, 2026년 정식 개발 예정)

👥 팀 구성 : 1명

🎯 역할 : 풀스택 개발자

🛠 기술 스택

- Python 3.10+, Streamlit, LangChain, Ollama (EXAONE 3.5 2.4B)
- ChromaDB, HuggingFace (jhgan/ko-sroberta-multitask)
- Document Processing: python-docx, olefile, pyhwp
- GPU Acceleration: CUDA 12.x
- 향후: Next.js 14, FastAPI, PostgreSQL (pgvector)

📋 주요 업무

- RAG 파이프라인 구축 : 문서 로딩, 청킹, 임베딩, 벡터 저장
- 다양한 문서 포맷 지원 : HWP (OLE2 & HWPX), DOCX, TXT 파서 구현
- GPU 가속 LLM 통합 : Ollama 기반 한국어 특화 모델 연동
- Streamlit 기반 채팅 UI 구현

### 📖 내용

- 외부 API 없이 로컬 환경에서 LLM을 구동하여 데이터 유출 방지
- 방대한 규정 문서에서 필요한 정보를 즉시 검색 및 질의응답
- 유사도 검색 기반 관련 조문 추출 (청크 크기 2000자, 중복 300자)
- 답변에 인용된 문서명 및 원문 미리보기 제공

### 🙋‍♂️ 역할

- PoC 설계 및 전체 개발
- HWP/DOCX 커스텀 로더 개발
- RAG 엔진 및 UI 구현

### 🎯 결과 및 성과

- 보안 강화 : 로컬 환경에서만 동작하여 데이터 유출 방지
- GPU 가속 : CUDA 활용으로 빠른 응답 속도 제공
- 업무 효율화 : 규정 문서 검색 시간 단축
- Phase 2 계획 : Next.js + FastAPI 기반 정식 서비스로 확장 예정