# ANG — AI Network Group

> 대학 행정 조직을 위한 통합 그룹웨어 플랫폼

Spring Boot + React 기반의 풀스택 그룹웨어입니다. 계층형 조직 관리, 전자결재, AI 문서 작성, 실시간 채팅, 파일 스토리지 등 행정 업무에 필요한 기능을 하나의 플랫폼에서 제공합니다.

---

## 주요 기능

| 기능 | 설명 |
|------|------|
| **조직 관리** | Company → Department → Team 3단계 계층 구조, 다중 소속·직급 지원 |
| **문서 관리** | 부서 단위 접근 제어, AWS S3 기반 파일 저장, DOCX·XLSX·HWP 미리보기 |
| **전자결재** | 결재 라인 지정, 서명 첨부, PDF 생성 및 다운로드 |
| **AI 문서 작성** | Ollama(Qwen3:14b) 기반 LLM으로 공문·보고서·회의록 초안 자동 생성 |
| **실시간 채팅** | WebSocket(STOMP) 기반 개인·그룹 채팅 |
| **일정 관리** | 개인·팀 캘린더, 일정 공유 |
| **게시판** | 공지사항, 자유 게시판 |
| **사내 메일** | 내부 메일 발신·수신 |
| **관리자 패널** | 가입 승인/거절, 권한 변경, 조직 편집, 강제 퇴사 처리 |

---

## 기술 스택

### Frontend
- **React 19** + Vite 8
- React Router v7, TanStack Query v5
- Axios (JWT 자동 갱신 인터셉터 적용)
- WebSocket — `@stomp/stompjs` + `sockjs-client`
- `docx-preview`, `xlsx`, `@rhwp/editor` (문서 미리보기)

### Backend
- **Spring Boot 3.4** / Java 21
- Spring Security + **JJWT 0.12** (AccessToken 30분 / RefreshToken 자동 갱신)
- Spring Data JPA + **MariaDB**
- Spring WebSocket (STOMP 브로커)
- **AWS S3** (파일 저장)
- Apache POI (DOCX·XLSX 파싱), OpenHTMLtoPDF + PDFBox (PDF 생성)

### AI / HWP Bridge
- **FastAPI** (Python) — REST API 서버
- **Ollama** + Qwen3:14b 커스텀 모델 (`ang-ai:latest`)
- pywin32 — HWP 자동화 (생성·수정·PDF 변환)

### Infra
- **Docker Compose** 멀티 컨테이너 구성 (frontend / backend / ai)
- Nginx (프론트엔드 SPA 서빙)
- GitHub Actions CI/CD

---

## 아키텍처

```
Browser
  │
  ├─ Nginx (port 5500)           ← Frontend (React SPA)
  │
  └─ Spring Boot (port 9090)     ← Backend REST API + WebSocket
       ├─ MariaDB
       ├─ AWS S3
       └─ FastAPI (port 8888)    ← AI Chat + HWP Bridge
            └─ Ollama (port 11434)
```

---

## 로컬 실행

### 사전 준비
- Docker & Docker Compose
- (HWP 기능 사용 시) Windows 환경 + 한컴오피스 + pywin32

### 1. 환경 변수 설정

```bash
cp .env.example .env
# .env 파일에서 다음 항목 입력:
# DB_URL, DB_USERNAME, DB_PASSWORD
# JWT_SECRET, ADMIN_INIT_PASSWORD
# AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION, S3_BUCKET
```

### 2. Docker Compose 실행

```bash
docker compose up -d
```

| 서비스 | 접속 주소 |
|--------|-----------|
| Frontend | http://localhost:5500 |
| Backend API | http://localhost:9090 |
| AI Bridge | http://localhost:8888 |

### 3. 개별 개발 서버 실행

**Frontend**
```bash
cd Frontend
npm install
npm run dev   # http://localhost:5500
```

**Backend**
```bash
cd Backend
./gradlew bootRun
```

**AI**
```bash
cd AI
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 8888
```

---

## 조직 체계 및 권한

시스템은 3단계 계층 구조를 가집니다.

```
Level 1 (COMPANY)    영진전문대학교
Level 2 (DEPARTMENT) ├─ 평생교육원          ← 핵심 권한 단위
Level 3 (TEAM)       │   ├─ 장기요양교육센터
                     │   └─ 행정지원팀
                     └─ 산학협력단
```

**문서 접근 규칙:** 동일한 Level 2 부서 산하의 구성원은 서로의 문서를 열람할 수 있습니다.

**권한 레벨:**
| 레벨 | 역할 |
|------|------|
| 1 | 일반 직원 |
| 50 | 부서 관리자 |
| 100 | 최고 관리자 |

---

## 사용자 가입 흐름

```
회원가입 (부서 코드 입력)
  → PENDING (승인 대기)
  → 관리자 승인 (권한·직급 지정)
  → ACTIVE (로그인 가능)
```

---

## AI 문서 작성

Ollama 위에서 실행되는 `ang-ai:latest` 모델(Qwen3:14b 파인튜닝)이 한국어 업무 문서를 작성합니다.

- 공문·보고서·제안서·기획서·회의록·결재서 등 문서 유형별 구조 자동 적용
- 참고 파일 업로드 시 해당 내용을 근거로 초안 생성
- 생성된 문서를 HWP·DOCX·PDF로 내보내기

---

## 프로젝트 구조

```
capstone/
├── Frontend/          # React + Vite SPA
├── Backend/           # Spring Boot REST API
├── AI/                # FastAPI (AI Chat + HWP Bridge)
├── HwpBridge/         # HWP 자동화 관련 유틸
├── docs/              # 설계 문서
├── scripts/           # 배포·운영 스크립트
├── docker-compose.yml
└── docker-compose.local.yml
```
