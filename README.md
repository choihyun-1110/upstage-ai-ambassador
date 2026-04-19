# Upstage AI Ambassador 2기 — 최현 프로젝트 모음

> Upstage AI Ambassador 2기 활동에서 진행한 프로젝트들을 모아둔 레포입니다.  
> Upstage Solar LLM, Document Parse 등 Upstage 제품을 활용한 실전 프로젝트들입니다.

🌐 **랜딩 페이지**: [choihyun-1110.github.io/upstage-ai-ambassador](https://choihyun-1110.github.io/upstage-ai-ambassador)

---

## 📂 프로젝트 목록

### 1. AI 커리어 어드바이저 — n8n 노코드 자동화
> `n8n_workflow/` | 미션 1

PDF 한 장으로 나만의 커리어 코치를 만드는 자동화 워크플로우.  
이력서/자기소개서 PDF를 업로드하면 AI가 프로필을 분석하고, 맞춤 대외활동·채용공고·업계 뉴스를 이메일로 발송합니다.

**기술 스택:** n8n · Upstage Document Parse · Upstage Solar Pro · Supabase pgvector · Gmail SMTP

---

### 2. JD Fit — Claude Code Skill
> `skills/jd-fit/` | 미션 2

채용공고 URL 하나만 던지면 전체 공고를 자동 크롤링해서 적합도 점수(0–100)와 함께 랭킹을 알려주는 Claude Code Skill.  
JS 동적 렌더링 채용 사이트도 Chrome MCP로 자동 크롤링. Upstage Solar Pro로 분석.

**기술 스택:** Claude Code Skills · Upstage Solar Pro · Upstage Document Parse · Chrome MCP

---

### 3. CatchTable Sniper — Claude Code Skill
> `skills/catchtable-sniper/` | 사이드 프로젝트

캐치테이블 빈자리를 30초 간격으로 감시하다가 취소 슬롯이 뜨는 순간 자동으로 예약하는 Claude Code Skill.  
멀티 타겟 동시 감시, 예약 오픈런 모드, 인원 유연 매칭, Dry-run 알림 모드 지원.

**기술 스택:** Claude Code Skills · Chrome MCP

🔗 [k-skill PR 리뷰 중](https://github.com/choihyun-1110/k-skill/pull/new/feat/catchtable-sniper)

---

## 🔑 공통 환경 설정

```bash
git clone https://github.com/choihyun-1110/upstage-ai-ambassador.git
cd upstage-ai-ambassador

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
├── career_advisor.json
└── user_profile.json
```

---

## 🔍 JD Fit — Claude Code Skill

채용 사이트 URL 하나만 입력하면:

1. **전체 공고를 자동으로 수집합니다** — JS 렌더링 사이트도 Chrome MCP로 자동 크롤링
2. **Upstage Solar Pro로 적합도를 분석합니다** — 0–100점 + 강점/약점/추천 이유
3. **랭킹 테이블로 정리해줍니다** — 어떤 공고에 먼저 지원할지 한눈에

### 설치

```bash
mkdir -p ~/.claude/skills/jd-fit
cp skills/jd-fit/SKILL.md ~/.claude/skills/jd-fit/SKILL.md
```

또는 curl로 바로 설치:

```bash
mkdir -p ~/.claude/skills/jd-fit
curl -o ~/.claude/skills/jd-fit/SKILL.md \
  https://raw.githubusercontent.com/choihyun-1110/upstage-ai-ambassador/main/skills/jd-fit/SKILL.md
```

### 사용

```
# Claude Code에서 URL 그냥 던지기
https://kia-autoworld.com/entry/  분석해줘

# PDF JD 파일
/Downloads/삼성전자_공고.pdf 분석해줘
```

### 기술 스택

| 역할 | 도구 |
|------|------|
| 스킬 플랫폼 | Claude Code Skills |
| JD 분석 | Upstage Solar Pro |
| PDF JD 처리 | Upstage Document Parse |
| JS 사이트 크롤링 | Chrome MCP |

### 분석 결과 예시

```
🏆 채용공고 적합도 랭킹 (총 6개 분석)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| 순위 | 직무                        | 점수 | 추천        |
|------|-----------------------------|------|-------------|
| 1위  | Data Scientist              | 65점 | ⚠️ 조건부   |
| 2위  | Machine Learning Engineer   | 65점 | ⚠️ 조건부   |
| 3위  | 글로벌 CRM                  | 50점 | ⚠️ 조건부   |
| 4위  | IT Project Manager          | 45점 | ⚠️ 조건부   |
| 5위  | Mobile Developer            | 40점 | ❌ 비추천   |
| 6위  | Solution Architect (ERP FI) | 30점 | ❌ 비추천   |
```

---

## 🍣 CatchTable Sniper — Claude Code Skill

원하는 식당을 지정하면 30초마다 빈자리를 체크하다가, 취소가 생기는 순간 자동으로 예약합니다.

### 주요 기능

| 기능 | 설명 |
|------|------|
| **취소 스나이핑** | 30초 간격 폴링, 빈자리 발견 즉시 자동 예약 |
| **멀티 타겟** | 식당 여러 개 동시 감시, 먼저 뜨는 곳 예약 |
| **오픈런 모드** | 예약 오픈 시간 정각에 맞춰 즉시 예약 시도 |
| **인원 유연 매칭** | 2인 없으면 4인 슬롯 발견 시 알림 |
| **Dry-run 모드** | 발견 알림만, 예약은 사람이 직접 |

### 설치

```bash
mkdir -p ~/.claude/skills/catchtable-sniper
curl -o ~/.claude/skills/catchtable-sniper/SKILL.md \
  https://raw.githubusercontent.com/choihyun-1110/upstage-ai-ambassador/main/skills/catchtable-sniper/SKILL.md
```

### 사전 준비

Chrome에서 캐치테이블(`app.catchtable.co.kr`) 로그인 후 Chrome MCP 연결.  
로그인 자동화 없음 — 세션만 사용.

### 사용 예시

```
"온지음 5월 토요일 저녁 2인 빈자리 나오면 예약해줘"
"온지음, 밍글스, 라연 중 5월 주말 2인 아무데나 먼저 뜨는 거 잡아줘"
"라연 5월 예약이 4월 30일 오전 10시 오픈이야, 그때 맞춰서 잡아줘"
"스시야마 이번달 2인 — 없으면 4인도 괜찮아, dry-run으로"
```

### 동작 흐름

```
[14:21:03] 온지음 5/3 확인 중... 없음
[14:21:33] 밍글스 5/3 확인 중... 없음
[14:22:03] 스시미루 5/3 확인 중... 없음
[14:22:33] ✅ 스시미루 19:30 빈자리 발견!
[14:22:34] 🎉 예약 완료
```

> 🔗 [k-skill PR 리뷰 중](https://github.com/choihyun-1110/k-skill/pull/new/feat/catchtable-sniper) — 머지 후 업데이트 예정

---

## 📁 전체 파일 구조

```
upstage-ai-ambassador/
├── index.html                        # GitHub Pages 랜딩 페이지
├── skills/
│   ├── jd-fit/
│   │   └── SKILL.md
│   └── catchtable-sniper/
│       └── SKILL.md
└── n8n_workflow/
    ├── career_advisor.json
    └── SETUP_GUIDE.md
```

---

*Powered by [Upstage Solar AI](https://console.upstage.ai)*
