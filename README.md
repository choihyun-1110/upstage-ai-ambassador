# 🎯 AI 커리어 어드바이저 — n8n 노코드 자동화

> Upstage AI Ambassador 2기 미션1 프로젝트
> PDF 한 장으로 나만의 커리어 코치를 만드세요.

## 📌 어떤 서비스인가요?

이력서, 자기소개서, 포트폴리오 등 본인의 경험이 담긴 PDF를 업로드하면:

1. **AI가 당신을 분석합니다** — "당신은 지금까지 이런 걸 해왔네요..."
2. **맞춤 액션을 추천합니다** — 대외활동, 채용공고, 업계 뉴스, 대학 공지
3. **이메일로 바로 받아봅니다** — 등록 즉시 첫 리포트 발송

### 이메일 예시
```
📬 최현님의 맞춤 커리어 리포트

🔍 AI가 본 당신
당신은 전기전자공학 전공으로 AI와 반도체 분야를 함께 탐구하며...

🏆 오늘 신청할 대외활동/공모전
1. [삼성 AI 챌린지] 반도체 설계 공모전 — 전공 역량을 직접 발휘할 수 있는 기회

💼 놓치면 안될 채용/인턴 정보
1. [SK하이닉스] 반도체 공정 인턴 — 희망 진로와 직결

📰 오늘 꼭 읽어야 할 업계 뉴스
1. [HBM 3세대 양산 소식] — 반도체 관심자라면 꼭 알아야 할 동향

🎓 전기전자공학부 대학 공지/세미나
1. KAIST AI 반도체 특강 — 연구 방향 설정에 도움

🎯 오늘의 커리어 미션
링커리어에서 AI 관련 대외활동 1개 찾아서 오늘 지원하기
```

---

## 🛠️ 기술 스택

| 역할 | 도구 |
|------|------|
| 자동화 플랫폼 | n8n (노코드) |
| PDF 분석 | Upstage Document Parse API |
| AI 분석/요약 | Upstage Solar Pro 3 |
| 벡터 임베딩 | Upstage Embeddings (4096차원) |
| 벡터 DB | Supabase pgvector |
| 데이터 수집 | Google News RSS, HackerNews RSS |
| 이메일 발송 | Gmail SMTP |

---

## 🚀 설치 및 실행 방법

### 1. 사전 준비

- [n8n 설치](https://docs.n8n.io/getting-started/installation/) (Node.js v20 권장)
- [Upstage API Key 발급](https://console.upstage.ai)
- Gmail 앱 비밀번호 발급 (Google 계정 → 보안 → 앱 비밀번호)
- [Supabase 계정 및 프로젝트 생성](https://supabase.com)

### 2. Supabase 테이블 생성

Supabase SQL Editor에서 실행:

```sql
create extension if not exists vector;

create table user_profiles (
  id uuid primary key default gen_random_uuid(),
  name text,
  email text,
  chat_id text,
  summary text,
  major text,
  career_goal text,
  keywords text[],
  embedding vector(4096),
  created_at timestamp default now()
);
```

### 3. n8n 실행

```bash
# 파일 접근 권한 포함하여 실행
N8N_RESTRICT_FILE_ACCESS_TO=/path/to/this/repo n8n start
```

### 4. 워크플로우 Import

1. n8n 접속 → `http://localhost:5678`
2. **Workflows → Import from file**
3. `n8n_workflow/career_advisor.json` 선택

### 5. 크레덴셜 등록

n8n → **Credentials** → **Add Credential**

**Upstage API Key (Header Auth)**
- Header Name: `Authorization`
- Header Value: `Bearer YOUR_UPSTAGE_API_KEY`
- 이름: `Upstage API Key`

**Gmail SMTP**
- Host: `smtp.gmail.com`
- Port: `465`
- SSL/TLS: `SSL/TLS`
- User: Gmail 주소
- Password: 앱 비밀번호 (16자리)
- 이름: `Gmail SMTP`

### 6. Supabase URL/Key 업데이트

`career_advisor.json` 내 `Supabase 저장` 노드의 헤더값을 본인 프로젝트 값으로 교체:
- `apikey`: Supabase anon key
- `Authorization`: `Bearer YOUR_SUPABASE_ANON_KEY`
- URL: `https://YOUR_PROJECT_ID.supabase.co/rest/v1/user_profiles`

### 7. 테스트

1. 워크플로우 활성화 (우측 상단 토글)
2. 폼 URL 접속 (워크플로우 내 Form Trigger 노드에서 URL 확인)
3. 이름, 이메일, PDF 업로드
4. 이메일 수신 확인

---

## 📁 파일 구조

```
├── n8n_workflow/
│   ├── career_advisor.json      # 메인 워크플로우 (단일 파일)
│   └── user_profile.json        # 등록 후 자동 생성 (gitignore됨)
├── 프로젝트_계획.md
├── Todo.md
├── README.md
└── .gitignore
```

---

## 🔑 환경 설정 요약

| 항목 | 값 |
|------|-----|
| n8n 포트 | 5678 |
| 워크플로우 파일 | `n8n_workflow/career_advisor.json` |
| 프로필 저장 경로 | `n8n_workflow/user_profile.json` |
| Solar 모델 | `solar-pro3` |
| Embeddings 모델 | `embedding-passage` |

---

## 📝 관련 블로그

- Upstage AI Ambassador 2기 미션1 후기 (작성 예정)
- PDF 파싱 비교: OpenDataLoader vs Upstage Document Parse (작성 예정)

---

*Powered by [Upstage Solar AI](https://console.upstage.ai)*
