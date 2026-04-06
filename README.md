# Upstage AI Ambassador 2기 — 최현 프로젝트 모음

> Upstage AI Ambassador 2기 활동에서 진행한 프로젝트들을 모아둔 레포입니다.
> Upstage Solar LLM, Document Parse 등 Upstage 제품을 활용한 실습 프로젝트들입니다.

---

## 📂 프로젝트 목록

### 1. AI 커리어 어드바이저 — n8n 노코드 자동화
> `n8n_workflow/` | 미션 1

PDF 한 장으로 나만의 커리어 코치를 만드는 자동화 워크플로우.
이력서/자기소개서 PDF를 업로드하면 AI가 프로필을 분석하고, 맞춤 대외활동·채용공고·업계 뉴스를 이메일로 발송합니다.

**기술 스택:** n8n · Upstage Document Parse · Upstage Solar Pro · Supabase pgvector · Gmail SMTP

→ [자세히 보기](#-ai-커리어-어드바이저--n8n-노코드-자동화)

---

### 2. JD Fit — Claude Code 스킬
> `skills/jd-fit/` | 미션 2

채용공고 URL 하나만 던지면 전체 공고를 자동 크롤링해서 적합도 점수(0-100)와 함께 랭킹을 알려주는 Claude Code 스킬.
기아, 삼성 등 JS 동적 렌더링 채용 사이트도 Chrome MCP로 자동 읽기. Upstage Solar Pro로 분석.

**기술 스택:** Claude Code Skills · Upstage Solar Pro · Upstage Document Parse · Chrome MCP

→ [자세히 보기](#-jd-fit--claude-code-스킬)

---

## 🔑 공통 환경 설정

```bash
# 레포 클론
git clone https://github.com/choihyun-1110/upstage-ai-ambassador.git
cd upstage-ai-ambassador

# API 키 설정
cp .env.example .env
# .env 파일에 UPSTAGE_API_KEY 입력
```

`.env` 파일:
```
UPSTAGE_API_KEY=your_upstage_api_key_here
```

---

## 🤖 AI 커리어 어드바이저 — n8n 노코드 자동화

이력서, 자기소개서, 포트폴리오 등 본인의 경험이 담긴 PDF를 업로드하면:

1. **AI가 당신을 분석합니다** — "당신은 지금까지 이런 걸 해왔네요..."
2. **맞춤 액션을 추천합니다** — 대외활동, 채용공고, 업계 뉴스, 대학 공지
3. **이메일로 바로 받아봅니다** — 등록 즉시 첫 리포트 발송

### 기술 스택

| 역할 | 도구 |
|------|------|
| 자동화 플랫폼 | n8n (노코드) |
| PDF 분석 | Upstage Document Parse API |
| AI 분석/요약 | Upstage Solar Pro |
| 벡터 임베딩 | Upstage Embeddings (4096차원) |
| 벡터 DB | Supabase pgvector |
| 데이터 수집 | Google News RSS, HackerNews RSS |
| 이메일 발송 | Gmail SMTP |

### 설치 및 실행

**1. 사전 준비**
- [n8n 설치](https://docs.n8n.io/getting-started/installation/) (Node.js v20 권장)
- [Upstage API Key 발급](https://console.upstage.ai)
- Gmail 앱 비밀번호 발급
- [Supabase 계정 생성](https://supabase.com)

**2. Supabase 테이블 생성**

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

**3. 실행**

```bash
N8N_RESTRICT_FILE_ACCESS_TO=/path/to/this/repo n8n start
```

n8n 접속 → `Workflows → Import from file` → `n8n_workflow/career_advisor.json`

**4. 크레덴셜 등록**

- Upstage API Key: Header Auth (`Authorization: Bearer YOUR_KEY`)
- Gmail SMTP: `smtp.gmail.com:465` + 앱 비밀번호

### 파일 구조

```
n8n_workflow/
├── career_advisor.json      # 메인 워크플로우
└── user_profile.json        # 등록 후 자동 생성 (gitignore)
```

---

## 🔍 JD Fit — Claude Code 스킬

채용 사이트 URL 하나만 입력하면:

1. **전체 공고를 자동으로 수집합니다** — JS 렌더링 사이트도 Chrome MCP로 자동 크롤링
2. **Upstage Solar Pro로 적합도를 분석합니다** — 0-100점 + 강점/약점/추천 이유
3. **랭킹 테이블로 정리해줍니다** — 어떤 공고에 먼저 지원할지 한눈에

### 사용 방법

**설치:**
```bash
# SKILL.md를 Claude Code 스킬 디렉토리에 복사
mkdir -p ~/.claude/skills/jd-fit
cp skills/jd-fit/SKILL.md ~/.claude/skills/jd-fit/SKILL.md
```

**사용:**
```
# Claude Code에서
/jd-fit https://career.kia.com/apply/applyList.kc

# 또는 그냥 URL을 던져도 자동 트리거
https://kia-autoworld.com/entry/  ← 이거 분석해줘
```

**프로필 저장** (선택):
```
~/.claude/jd-fit-profile.json  ← 한 번 입력하면 다음부터 재사용
```

### 기술 스택

| 역할 | 도구 |
|------|------|
| 스킬 플랫폼 | Claude Code Skills |
| JD 분석 | Upstage Solar Pro |
| PDF JD 처리 | Upstage Document Parse |
| JS 사이트 크롤링 | Chrome MCP (navigate + get_page_text) |

### 파일 구조

```
skills/
└── jd-fit/
    └── SKILL.md    # 스킬 정의 파일
```

### 분석 결과 예시

```
🏆 채용공고 적합도 랭킹 (총 6개 분석)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| 순위 | 직무 | 점수 | 추천 |
|------|------|------|------|
| 1위  | Data Scientist | 65점 | ⚠️ 조건부 추천 |
| 2위  | Machine Learning Engineer | 65점 | ⚠️ 조건부 추천 |
| 3위  | 글로벌 CRM | 50점 | ⚠️ 조건부 |
| 4위  | IT Project Manager | 45점 | ⚠️ 조건부 |
| 5위  | Mobile Developer | 40점 | ❌ 비추천 |
| 6위  | Solution Architect (ERP FI) | 30점 | ❌ 비추천 |
```

---

*Powered by [Upstage Solar AI](https://console.upstage.ai)*
