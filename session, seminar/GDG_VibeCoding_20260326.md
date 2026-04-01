# GDG_VibeCoding_20260326

## Prototype -> 개발 -> 배포 약간

### 1. AI의 역사

### 2. Context is Important

- AI는 확률에 기반해서 움직임

---

context -> 정보 제공 범위(AI 프롬프트에 들어가는 거)
prompt -> AI 방향성 지휘

5가지(일반적인 AI)
목표, 정보, 제약조건, 참고자료, 결과 산출물

5가지(바이브코딩)
목표, 정보, 제약조건, 참고자료, **방향성(도메인)**

## 목표

### context

### constraints

- context 50% -> 길어지면 퍼포먼스 다운

- multi-turns
  -> avg 39% reduce

- 중요한 정보는 처음이랑 끝에 얘기하는 것이 AI가 잘 알아들음

- 1 Task 1 Session
- 세션 초기화!!!!
- 설명할 때 사족을 붙이지 말고, 핵심만 간추려서 할 것

- 멀티 턴 계속 길어지면?
  -> 룰이랑 상태 잘 저장할 것.

---

- 은혁이 레포 참고! / PPT에 영어 설명 있음
  -> 이거 할 건데, 이 자료 먼저 참고해봐

장점

- 할루시네이션 감소(굿 베이스 프롬프트)
- 태스크 라우팅
- 프로젝트 탐색, 세션을 열어도 큰 문제 없어짐

-> 우리가 쓰는대로 AI가 하니까 정신 차리기

### docs 템플릿 예제(Gemini 추천)

```
# 🤖 AGENTS.md: antigravity Project Blueprint

## 1. Environment Setup (환경 설정)
- 본 프로젝트는 보안을 위해 `.env` 파일을 기반으로 환경 변수를 관리합니다.
- **Action**: `cp .env.example .env` 명령어를 실행한 후 필수 값을 채우세요.

### 필수 환경 변수 목록
- `BACKEND_URL`: FastAPI 서버 주소 (Default: http://localhost:8000)
- `REACT_APP_API_URL`: 프론트엔드 통신용 주소
- `GEMMA_API_KEY`: 트렌드 분석을 위한 API 키 (비공개 필수)

## 2. Technical Stack & Architecture
- **Backend**: FastAPI (Python 3.10+) - Pydantic을 통한 엄격한 타입 검증
- **Frontend**: React (Functional Components) - Hooks 기반 상태 관리
- **Database**: PostgreSQL (SQLAlchemy ORM 사용)

## 3. Development Agent Guide (AI 협업 규칙)
- 새로운 API 추가 시: `app/api/v1` 폴더에 위치시키고 `schemas/`에 데이터 모델 정의 필수.
- 프론트엔드 컴포넌트 추가 시: `src/components` 하위에 기능별 폴더 생성.
```

지금 제공한 API key는 프로젝트 환경변수에 넣어서 서비스에 적용할 것.
절대 google AI studio에 적용해서 추가로 쓰는 건 아님.
