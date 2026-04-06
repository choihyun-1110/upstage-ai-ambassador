---
name: jd-fit
description: |
  JD 적합도 분석기 — 채용공고와 사용자 커리어 프로필을 비교해 적합도 점수(0-100), 강점/약점, 지원 추천 여부, 자소서 포인트를 분석합니다.

  다음 상황에서 반드시 이 스킬을 사용하세요:
  - 사용자가 채용공고 URL, PDF, 또는 JD 텍스트를 제공할 때
  - "/jd-fit", "JD 적합도", "이 공고 분석해줘", "지원해도 될까", "나한테 맞는 공고 추천해줘" 같은 표현이 나올 때
  - 취업/이직 관련 채용 사이트 URL을 공유할 때
  - 채용 공고 목록 페이지 URL을 주고 어떤 포지션이 맞는지 물어볼 때
---

# JD Fit 스킬

사용자가 채용공고(JD)를 주면 → 사용자 프로필과 비교 → Upstage Solar LLM으로 분석 → 적합도 리포트 출력.

---

## 1단계: 입력 파악

입력 유형을 먼저 판단한다:

| 입력 유형 | 처리 방법 |
|-----------|-----------|
| PDF 파일 경로 | Upstage Document Parse API로 텍스트 추출 |
| 단일 JD URL | Chrome MCP로 페이지 내용 읽기 |
| 채용 목록/랜딩 페이지 URL | Chrome MCP로 모든 JD 링크 추출 → 각각 방문 → 전체 분석 후 랭킹 |
| 텍스트로 직접 붙여넣기 | 그대로 분석 |

**절대 사용자에게 내용을 복사-붙여넣기 해달라고 요청하지 않는다.** Chrome MCP로 직접 읽는다.

---

## 2단계: 사용자 프로필 수집

우선순위 순서로 프로필을 가져온다:

1. **대화에서 명시적으로 제공한 경우** → 그대로 사용
2. **`~/.claude/jd-fit-profile.json` 파일이 존재하는 경우** → 읽어서 사용
3. **위 두 경우가 아닐 때만** → 아래 항목을 물어본다:
   - 전공 또는 현재 직무
   - 주요 기술/경험 (언어, 도구, 프로젝트)
   - 커리어 목표
   - 경력 연차 (신입 / n년차)
   - (선택) 이력서/자기소개서 내용 붙여넣기

프로필 수집 후 `~/.claude/jd-fit-profile.json`에 저장해 다음번에 재사용한다.

---

## 3단계: JD 내용 가져오기

### 3A. URL인 경우 (Chrome MCP 사용)

현대 채용 사이트는 JavaScript SPA로 구성되어 일반 HTTP 요청으로는 내용을 읽을 수 없다.
반드시 Chrome MCP를 사용한다:

1. `mcp__Claude_in_Chrome__navigate` → URL로 이동
2. `mcp__Claude_in_Chrome__get_page_text` → 페이지 전체 텍스트 읽기

**채용 목록 페이지인 경우** (하나의 URL에 여러 공고가 나열된 경우):

먼저 `mcp__Claude_in_Chrome__javascript_tool`로 모든 JD 링크를 추출한다:
```javascript
const links = [...document.querySelectorAll("a")]
  .map(a => ({ href: a.href, text: a.textContent.trim() }))
  .filter(a => a.href && !a.href.endsWith("#") && a.href !== window.location.href);
return JSON.stringify(links.slice(0, 50));
```

링크가 `href="#"` 또는 JS 모달 방식인 경우, DOM에서 data 속성을 파싱하거나
`mcp__Claude_in_Chrome__find`로 버튼을 찾아 직접 클릭하여 모달/상세 페이지의 내용을 읽는다.

각 JD 링크를 순차적으로 방문하여 내용을 읽은 후 jd_list에 { url, title, content }로 저장한다.

### 3B. PDF인 경우 (Upstage Document Parse)

```bash
curl -X POST "https://api.upstage.ai/v1/document-digitization" \
  -H "Authorization: Bearer $UPSTAGE_API_KEY" \
  -F "document=@/path/to/jd.pdf" \
  -F 'output_formats=["text"]'
```

응답의 `content.text` 또는 `pages[].text`를 JD 텍스트로 사용한다.

---

## 4단계: Upstage Solar LLM으로 분석

각 JD에 대해 Solar Pro 모델로 분석한다.
아래 Python 코드를 실행하거나 동일한 curl 요청을 보낸다:

```python
import os, requests

api_key = os.environ.get("UPSTAGE_API_KEY", "")

def analyze_jd(jd_text, user_profile):
    prompt = f"""## 채용공고
{jd_text}

## 지원자 프로필
{user_profile}

다음 형식으로 분석해주세요:
1. 적합도 점수 (0-100, 숫자만)
2. 매칭되는 강점 (각 항목에 근거 포함)
3. 부족하거나 보완이 필요한 부분 (각 항목에 보완 방법 포함)
4. 지원 추천 여부: 추천 / 비추천 / 조건부 추천 (이유 포함)
5. 자기소개서/면접에서 강조할 포인트 3가지
6. JD에서 주목할 키워드 3-5개 (각 키워드의 의미 포함)"""

    resp = requests.post(
        "https://api.upstage.ai/v1/chat/completions",
        headers={"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"},
        json={
            "model": "solar-pro",
            "messages": [
                {"role": "system", "content": "당신은 커리어 어드바이저입니다. 채용공고와 지원자 프로필을 분석하여 정확하고 솔직한 적합도 분석을 제공합니다."},
                {"role": "user", "content": prompt}
            ],
            "stream": False
        }
    )
    return resp.json()["choices"][0]["message"]["content"]
```

`$UPSTAGE_API_KEY` 환경 변수로 읽는다. 없을 경우 한 번만 사용자에게 물어본다.

---

## 5단계: 결과 출력

### 단일 JD 분석 형식:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 JD 적합도 분석 결과
[회사명] 직무명
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎯 적합도 점수: XX/100

✅ 매칭되는 강점
• [강점] → 근거

⚠️ 부족하거나 보완이 필요한 부분
• [부족한 점] → 보완 방법

📌 지원 추천: 추천 / 비추천 / 조건부 추천
[추천 이유 1-2문장]

💡 자기소개서 & 면접에서 강조할 포인트
1. [포인트1]
2. [포인트2]
3. [포인트3]

🔍 JD에서 주목할 키워드
• "키워드" — 의미
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 여러 JD 비교 형식 (랜딩 페이지 입력 시):

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏆 채용공고 적합도 랭킹 (총 N개 분석)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| 순위 | 직무명 | 점수 | 추천 |
|------|--------|------|------|
| 1위  | [직무] | 85점 | ✅ 추천 |
| 2위  | [직무] | 72점 | ✅ 추천 |
| 3위  | [직무] | 45점 | ⚠️ 조건부 |

🥇 최우선 추천: [직무명]
[추천 이유 2-3문장]

[이하 각 공고 상세 분석...]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 주의사항

- **Chrome MCP 우선**: JS 동적 렌더링 사이트는 일반 HTTP 요청 불가. 항상 `navigate` + `get_page_text` 사용.
- **모달/JS 링크 처리**: `href="#"` 버튼들은 `javascript_tool`로 DOM 파싱하거나 직접 클릭해서 내용 추출.
- **API 키**: `$UPSTAGE_API_KEY` 환경변수 사용. 없을 때만 한 번 물어본다.
- **프로필 저장**: 첫 사용 후 `~/.claude/jd-fit-profile.json`에 저장 여부를 물어본다.
- **Upstage 제품 사용**: 분석에는 반드시 Solar Pro LLM, PDF 처리에는 Document Parse를 사용한다.
