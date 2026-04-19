# 테크 뉴스 요약 봇 - n8n 셋업 가이드

## 워크플로우 개요

```
[Schedule] 평일 오전 9시
    ↓
[RSS 수집] HackerNews + TechCrunch + The Verge
    ↓
[키워드 필터] AI, LLM, Apple, Nvidia 등
    ↓
[Upstage Solar API] 한국어 요약 생성
    ↓
[Telegram] 브리핑 발송  +  [Google Sheets] 로그 저장
```

---

## 1단계: n8n 설치

### 로컬 (Docker)
```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```
→ http://localhost:5678 접속

### 클라우드 (권장: n8n.cloud)
- https://n8n.cloud 에서 무료 계정 생성 (14일 트라이얼)

---

## 2단계: 필요한 API 키 준비

| 서비스 | 발급 위치 | 용도 |
|--------|-----------|------|
| **Upstage API** | https://console.upstage.ai | Solar 요약 |
| **Telegram Bot** | @BotFather → `/newbot` | 알림 발송 |
| **Google Sheets** | Google Cloud Console OAuth | 기록 저장 |

### Telegram 봇 만들기
1. Telegram에서 `@BotFather` 검색
2. `/newbot` 입력 → 봇 이름/username 설정
3. **Bot Token** 복사
4. `@userinfobot` 에서 내 Chat ID 확인

---

## 3단계: 워크플로우 가져오기

1. n8n 좌측 메뉴 → **Workflows** → **Import from File**
2. `tech_news_bot.json` 파일 선택
3. 워크플로우 열림 확인

---

## 4단계: Credentials 설정

### Upstage API Key
- n8n → Settings → Credentials → New
- Type: **Header Auth**
- Name: `Upstage API Key`
- Header Name: `Authorization`
- Header Value: `Bearer YOUR_UPSTAGE_API_KEY`

### Telegram Bot
- Credentials → New → **Telegram**
- Access Token: BotFather에서 받은 토큰

### Google Sheets (선택)
- Credentials → New → **Google Sheets OAuth2**
- Google Cloud Console에서 OAuth 앱 생성 후 연결

---

## 5단계: Variables 설정

n8n → Settings → Variables:
| 변수명 | 값 |
|--------|-----|
| `TELEGRAM_CHAT_ID` | 본인 Telegram Chat ID (예: `123456789`) |
| `GOOGLE_SHEET_ID` | Google Sheets URL의 spreadsheet ID |

---

## 6단계: 테스트 실행

1. 워크플로우 열기 → **Test workflow** 클릭
2. 각 노드 실행 결과 확인
3. Telegram에 메시지 도착 확인

---

## 커스터마이징

### 키워드 필터 수정
`키워드 필터` 노드 → regex 패턴 수정:
```
AI|LLM|GPT|Claude|Gemini|Apple|Google|OpenAI|Nvidia|chip|semiconductor|startup
```
관심 키워드 추가/제거

### 다른 RSS 피드 추가
- MIT Technology Review: `https://www.technologyreview.com/feed/`
- Wired: `https://www.wired.com/feed/rss`
- GeekWire: `https://www.geekwire.com/feed/`

### 발송 시간 변경
`매일 오전 9시` 노드 → cron 표현식:
- 매일 8시 30분: `30 8 * * *`
- 평일만 9시: `0 9 * * 1-5`
- 매주 월요일 9시: `0 9 * * 1`

---

## SNS 공유 포인트

스크린샷으로 찍으면 좋은 것들:
1. **n8n 워크플로우 전체 화면** - 노드 연결 그림
2. **Telegram 수신 결과** - 요약된 뉴스 브리핑
3. **Upstage Solar 노드 설정** - AI 연동 포인트 강조

---

## 아키텍처 다이어그램

```
┌─────────────────────────────────────────────────────────┐
│                   n8n Workflow                          │
│                                                         │
│  ⏰ Schedule         📰 RSS Feeds                       │
│  (평일 9AM)    →    HN + TechCrunch + Verge             │
│                           ↓                             │
│                    🔍 Keyword Filter                     │
│                    (AI, LLM, Apple...)                   │
│                           ↓                             │
│                    📦 Aggregate Articles                 │
│                           ↓                             │
│                    🤖 Upstage Solar API                  │
│                    (한국어 요약 생성)                     │
│                           ↓                             │
│              ┌────────────┴────────────┐                │
│              ↓                         ↓                │
│         📱 Telegram              📊 Google Sheets       │
│         (브리핑 발송)             (로그 저장)            │
└─────────────────────────────────────────────────────────┘
```
