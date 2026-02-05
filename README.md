
# 커밋나우 커뮤니티 게시판

## ✏️ 과제의 개요

| 카테고리 | 난이도 | 제한시간 |
|----------|--------|----------|
| full_stack | normal | 14d |

### 💻 과제에서 요구하는 개발언어

- nodejs
- express
- nestjs
- typescript
- nextjs
- react
- javascript


## 📜 과제의 내용

> 과제 설명과 요구사항을 참고하여, 구현해 주세요.

### 👀 과제의 설명

# 과제 가이드

## 개요

커밋나우 **커뮤니티 게시판**의 풀스택 과제입니다.

핵심 여정은 **게시글 목록 → 상세/댓글 → 작성·수정 → 반응(좋아요) → 검색/태그 필터 → 신고/관리(선택)**&#xC774;며, 인증·권한·페이지네이션·상태 전이를 포함합니다.

### 사용자 시나리오

1. **목록 탐색**

   1. 사용자는 `/community`에서 최신순/인기순 정렬, 태그 필터, 검색어로 글을 찾는다.
   2. 무한스크롤 또는 페이지네이션으로 더 불러온다. 로딩/빈/에러 상태를 구분하여 표시한다.
2. **상세/댓글**

   1. `/community/:postId`에서 본문(마크다운), 작성자, 태그, 좋아요 수, 댓글 스레드를 본다.
   2. 로그인 사용자는 댓글/대댓글을 작성·수정·삭제할 수 있다(제한적 2뎁스).
3. **작성/수정**

   1. `/community/write`에서 제목·본문(마크다운), 태그(오토컴플리트)를 입력해 게시한다.
   2. 본인 글은 수정 가능, 삭제는 소프트 삭제(상태 전환)로 처리한다.
4. **반응/신고**

   1. 좋아요 토글(중복 방지, 낙관적 UI 허용).
   2. 부적절한 게시글/댓글은 신고(선택). 관리자는 숨김/차단할 수 있다(선택).
5. **인증/권한**

   1. 글/댓글 작성·수정·삭제는 로그인 필요.
   2. 관리자만 숨김/차단/신고 처리 가능(선택).

### 필요한 화면

* **게시글 목록** `/community` 검색 인풋, 태그 칩 필터, 정렬 드롭다운(최신/인기), 그리드 or 리스트 카드, 로딩/빈/에러 상태.&#x20;
* **게시글 상세** `/community/:postId` 제목/메타(작성자·시간·조회), 본문(마크다운 렌더), 태그, 좋아요, 댓글 트리(2뎁스), 본문/댓글 편집 버튼(권한 시).&#x20;
* **글 작성/수정** `/community/write`, `/community/:postId/edit` 제목, 마크다운 에디터(프리뷰 탭), 태그 입력(쉼표/칩), 임시저장(선택).&#x20;
* **마이페이지(선택)** `/me` 내가 쓴 글/댓글, 좋아요한 글.&#x20;
* **관리(선택)** `/admin/community` 신고 목록, 숨김 처리, 사용자 제재(간단 토글).&#x20;

### 데이터 인터페이스 및 예시

```typescript
// types.ts (권장: 백/프 공용 DTO)
export type Role = 'USER' | 'ADMIN';
export type PostStatus = 'DRAFT' | 'PUBLISHED' | 'HIDDEN' | 'DELETED';
export type CommentStatus = 'VISIBLE' | 'HIDDEN' | 'DELETED';

export interface UserPublic {
  id: string;           // "u_101"
  nickname: string;     // 노출명
  role: Role;           // USER/ADMIN
  avatarUrl?: string;
}

export interface Tag {
  id: string;           // "t_typescript"
  name: string;         // "TypeScript"
}

export interface Post {
  id: string;           // "p_501"
  title: string;
  body: string;         // markdown
  author: UserPublic;
  tags: Tag[];
  status: PostStatus;   // 목록엔 PUBLISHED만
  likeCount: number;
  commentCount: number;
  createdAt: string;    // ISO
  updatedAt: string;    // ISO
}

export interface Comment {
  id: string;           // "c_9001"
  postId: string;
  parentId?: string;    // 대댓글이면 부모 ID
  body: string;         // markdown(제한적)
  author: UserPublic;
  status: CommentStatus;
  likeCount: number;
  createdAt: string;
  updatedAt: string;
}

export interface PageMeta {
  page: number;
  limit: number;
  total: number;
  hasNext: boolean;
}

export interface ApiError {
  code: string;         // 예: "VALIDATION_ERROR" | "NOT_FOUND"
  message: string;
  details?: Record<string, any>;
}

```

```json
// posts.json (최소 예시)
[
  {
    "id": "p_501",
    "title": "React 상태 관리, 무엇을 써야 할까?",
    "body": "팀 규모/요구사항에 따라 다릅니다... 경험담 공유합니다.",
    "author": { "id": "u_101", "nickname": "han", "role": "USER", "avatarUrl": "/avatars/han.png" },
    "tags": [{ "id": "t_react", "name": "React" }, { "id": "t_state", "name": "State" }],
    "status": "PUBLISHED",
    "likeCount": 12,
    "commentCount": 3,
    "createdAt": "2025-10-18T03:10:00Z",
    "updatedAt": "2025-10-18T03:10:00Z"
  },
  {
    "id": "p_502",
    "title": "NestJS에서 모듈 나누는 기준",
    "body": "도메인 중심으로 나누되, 순환 의존성 주의...",
    "author": { "id": "u_102", "nickname": "alice", "role": "USER" },
    "tags": [{ "id": "t_nest", "name": "NestJS" }],
    "status": "PUBLISHED",
    "likeCount": 5,
    "commentCount": 1,
    "createdAt": "2025-10-19T11:00:00Z",
    "updatedAt": "2025-10-19T11:00:00Z"
  }
]

```

```json
// comments.json (최소 예시)
[
  {
    "id": "c_9001",
    "postId": "p_501",
    "body": "전 전역 상태는 최소화하고, 서버 캐싱을 적극 씁니다.",
    "author": { "id": "u_103", "nickname": "bob", "role": "USER" },
    "status": "VISIBLE",
    "likeCount": 2,
    "createdAt": "2025-10-18T04:00:00Z",
    "updatedAt": "2025-10-18T04:00:00Z"
  },
  {
    "id": "c_9002",
    "postId": "p_501",
    "parentId": "c_9001",
    "body": "맞아요. 그리고 폴더 구조는 기능 단위로!",
    "author": { "id": "u_101", "nickname": "han", "role": "USER" },
    "status": "VISIBLE",
    "likeCount": 1,
    "createdAt": "2025-10-18T04:10:00Z",
    "updatedAt": "2025-10-18T04:10:00Z"
  }
]

```

```json
// tags.json (최소 예시)
[
  { "id": "t_react", "name": "React" },
  { "id": "t_state", "name": "State" },
  { "id": "t_nest", "name": "NestJS" }
]

```

```json
// users.public.json (최소 예시 – 공개 정보)
[
  { "id": "u_101", "nickname": "han", "role": "USER", "avatarUrl": "/avatars/han.png" },
  { "id": "u_102", "nickname": "alice", "role": "USER" },
  { "id": "u_201", "nickname": "admin", "role": "ADMIN" }
]

```

> 주의: 본문/댓글은 마크다운을 허용하되 XSS 방지를 위해 허용 태그·속성만 화이트리스트로 렌더하세요.

​


### 🎯 과제의 요구사항

- 목록/검색/필터/정렬: /community에서 검색어·태그·정렬(최신/인기) 동작, 페이지네이션 or 무한스크롤, 로딩/빈/에러 상태 구분
- 상세/댓글: 본문 마크다운 렌더, 댓글(2뎁스), 본인 소유 편집/삭제, 비로그인 읽기 전용
- 작성/수정: 제목/본문/태그 입력, 임시저장(선택), 소프트 삭제(상태 전환: DELETED or HIDDEN)
- 반응(좋아요): 게시글/댓글 좋아요 토글, 중복 방지, 카운트 정확
- 인증/권한: 로그인/권한 체크(JWT/세션/OAuth 자유). 관리자만 숨김/신고 처리(선택)
- 페이지네이션/정렬 일관성: 서버 응답의 PageMeta 사용, 정렬 안정성 확보
- 에러 모델 통일: ApiError 스키마(코드/메시지/필드)로 응답
- 보안: XSS/스크립트 삽입 방지(마크다운 sanitize), 첨부 파일 시 크기·확장자 검증(선택)
- 테스트: 글/댓글 CRUD, 권한, 좋아요 토글, 페이지네이션/정렬 최소 단위·통합 테스트
- (선택) 실시간: 신규 댓글/좋아요 카운트 SSE/WebSocket 브로드캐스트
- (선택) 신고/관리: 신고 생성·목록·처리(숨김), 관리 화면
