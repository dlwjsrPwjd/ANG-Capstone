# ANG Frontend

React 19 + Vite 기반 SPA입니다. 루트 [README.md](../README.md)에서 전체 아키텍처와 실행 방법을 확인하세요.

---

## 기술 스택

| 분류 | 라이브러리 |
|------|-----------|
| UI 프레임워크 | React 19, Vite 8 |
| 라우팅 | React Router v7 |
| 서버 상태 관리 | TanStack Query v5 |
| HTTP 클라이언트 | Axios (JWT 자동 갱신 인터셉터) |
| 실시간 통신 | STOMP over SockJS |
| 문서 미리보기 | docx-preview, xlsx, @rhwp/editor |
| 캘린더 | react-calendar |

---

## 로컬 실행

```bash
npm install
npm run dev
```

`http://localhost:5500` 에서 접속

백엔드 API 서버(`http://localhost:9090`)가 실행 중이어야 합니다.

---

## 프로젝트 구조

```
src/
├── api/               # Axios 인스턴스 및 API 호출 함수
├── components/
│   ├── auth/          # 로그인·회원가입
│   ├── calendar/      # 일정 관리
│   ├── chat/          # 실시간 채팅 (WebSocket)
│   ├── common/        # 공통 UI 컴포넌트
│   ├── esignature/    # 전자결재·서명
│   ├── file/          # 파일 스토리지·미리보기
│   ├── mail/          # 사내 메일
│   └── pages/         # 페이지 레벨 컴포넌트
│       ├── Admin.jsx          # 관리자 패널
│       ├── Board.jsx          # 게시판
│       ├── Calendar.jsx       # 캘린더
│       ├── Chat.jsx           # 채팅
│       ├── DocumentWriter.jsx # AI 문서 작성
│       ├── ESignature.jsx     # 전자결재
│       ├── FileStorage.jsx    # 파일 스토리지
│       ├── Home.jsx           # 대시보드
│       ├── Mail.jsx           # 사내 메일
│       ├── MyPage.jsx         # 마이페이지
│       └── Organization.jsx   # 조직도
├── contexts/          # React Context (인증 상태 등)
├── hooks/             # 커스텀 훅
├── router/            # 라우트 정의
├── constants/         # 상수
└── utils/             # 유틸리티 함수
```

---

## JWT 자동 갱신

`src/api/` 의 Axios 인터셉터가 토큰 만료를 자동으로 처리합니다.

1. 요청 헤더에 `AccessToken` 첨부
2. 백엔드에서 `401` 응답 수신 시 요청 보류
3. `LocalStorage`의 `RefreshToken`으로 `/auth/refresh` 호출
4. 새 토큰 발급 후 원래 요청 자동 재시도
5. RefreshToken마저 만료된 경우 로그아웃 처리

---

## 빌드

```bash
npm run build   # dist/ 디렉토리에 정적 파일 생성
```

프로덕션 배포는 Docker 이미지(`Dockerfile`)를 통해 Nginx로 서빙됩니다.
