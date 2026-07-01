# XAG Quiz 앱 업데이트 내역

> 기준 버전: v26 → v27  
> 업데이트 범위: 엑셀 로우데이터 구조 변경 / 컨버터 / index.html

---

## 1. 엑셀 로우데이터 구조 변경 (v26 → v27)

### 컬럼 3개 추가

| 컬럼명 | 위치 | 값 형식 | 설명 |
|---|---|---|---|
| `learning_type` | 12번째 | `principle` / `apply` / `visual` | 문제 학습 유형 |
| `visual_type` | 13번째 | `none` / `color_pair` | 시각 요소 유형 |
| `visual_code` | 14번째 | JSON 문자열 또는 빈값 | 시각 요소 렌더링 명세 |

**기존 문제(XAG103~123) 기본값**: `learning_type=principle`, `visual_type=none`, `visual_code=''`

### visual_code JSON 구조 (color_pair 타입)

```json
{
  "bg": "#4A90D9",
  "text_color": "#FFFFFF",
  "contrast_ratio": "약 3.3:1",
  "element": "18pt 미만 본문 텍스트",
  "render_type": "text",
  "font_size": "13px",
  "sample_text": "게임 설정을 저장하려면 A 버튼을 누르세요."
}
```

| 필드 | 설명 |
|---|---|
| `bg` | 배경색 (hex) |
| `text_color` | 텍스트 또는 버튼 테두리 색 (hex) |
| `contrast_ratio` | 표시용 대비율 문자열 |
| `element` | 적용 요소 설명 (배지에 표시) |
| `render_type` | `text` 또는 `button_border` |
| `font_size` | `render_type=text`일 때 폰트 크기 (예: `13px`) |
| `sample_text` | 렌더링할 샘플 문자열 |

### render_type 종류

| render_type | 렌더링 방식 | 적용 예시 |
|---|---|---|
| `text` | 지정 폰트 크기로 샘플 텍스트 렌더링 | 본문 텍스트 대비 문제 |
| `button_border` | 투명 배경 + 지정 색 테두리 버튼 렌더링 | UI 요소 테두리 대비 문제 |

---

## 2. XAG102 퀴즈 재설계 (84개 → 35개)

### 학습 유형 분포

| 학습 유형 | 수량 | 비율 | 문제 유형 |
|---|---|---|---|
| `principle` (원칙·정의 확인) | 11개 | 31% | MC 6 + OX 3 + SA 2 |
| `apply` (판단·적용) | 16개 | 46% | MC 10 + OX 4 + SA 3 |
| `visual` (설계 오류 발견) | 8개 | 23% | color_pair 8개 |

### visual 문제 8개 목록

| key | bg | text_color | 대비율 | element | render_type | font_size |
|---|---|---|---|---|---|---|
| XAG102_A005 | #FFFFFF | #999999 | 약 2.8:1 | 기능적 아이콘 | text | 36px (❤) |
| XAG102_V001 | #4A90D9 | #FFFFFF | 약 3.3:1 | 18pt 미만 본문 | text | 13px |
| XAG102_V002 | #4A90D9 | #FFFFFF | 약 3.3:1 | 버튼 테두리 | button_border | — |
| XAG102_V003 | #000000 | #FF0000 | 약 5.25:1 | 18pt 미만 본문 | text | 15px |
| XAG102_V004 | #FFFFFF | #FFFF00 | 약 1.07:1 | 18pt 미만 본문 | text | 14px |
| XAG102_V005 | #FFFFFF | #767676 | 약 4.54:1 | 18pt 미만 본문 | text | 12px |
| XAG102_V006 | #AAAAAA | #777777 | 약 1.77:1 | 버튼 테두리 | button_border | — |
| XAG102_V007 | #FFFFFF | #CC5500 | 약 4.32:1 | 18pt 미만 본문 | text | 13px |

---

## 3. 컨버터 업데이트 (XAG_Quiz_Converter_latest.html)

### 변경 위치 1 — Excel → JS 변환 (q 객체 생성부)

```javascript
// 기존 (11필드)
const q = {
  key, pool_id, question_type, question,
  options, answer_index, answer_text, explanation
};

// 변경 후 (14필드)
const q = {
  key, pool_id, question_type, question,
  options, answer_index, answer_text,
  explanation: String(r[10] || ''),
  learning_type: String(r[11] || 'principle'),   // 신규
  visual_type:   String(r[12] || 'none'),         // 신규
  visual_code:   String(r[13] || '')              // 신규
};
```

### 변경 위치 2 — JS → Excel 역방향 변환 (headers, rows)

```javascript
// 기존 (11컬럼)
const headers = ['key', ..., 'explanation'];
const rows = data.questions.map(q => [
  q.key, ..., q.explanation
]);

// 변경 후 (14컬럼)
const headers = ['key', ..., 'explanation',
                 'learning_type', 'visual_type', 'visual_code'];
const rows = data.questions.map(q => [
  q.key, ..., q.explanation,
  q.learning_type || 'principle',
  q.visual_type   || 'none',
  q.visual_code   || ''
]);
```

---

## 4. index.html 업데이트

### 4-1. HTML 구조 변경

```html
<!-- 기존 -->
<div class="question-card" ...>
  <div class="question-number" id="questionNumber"></div>
  <div class="question-text" id="questionText"></div>
</div>
<div class="options" id="optionsContainer"></div>

<!-- 변경 후 — visualContainer 추가 -->
<div class="question-card" ...>
  <div class="question-number" id="questionNumber"></div>
  <div class="question-text" id="questionText"></div>
</div>
<div id="visualContainer"></div>   <!-- 신규 -->
<div class="options" id="optionsContainer"></div>
```

### 4-2. CSS 추가

```css
/* 공통 래퍼 */
.color-pair-preview {
  border-radius: 10px;
  padding: 20px 24px;
  margin-bottom: 16px;
  min-height: 72px;
  border: 1px solid rgba(255,255,255,0.08);
  box-shadow: 0 4px 20px rgba(0,0,0,0.35);
}

/* 텍스트 타입: 좌우 배치 */
.color-pair-preview.cp-text-layout {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

/* 버튼 테두리 타입: 세로 중앙 배치 */
.color-pair-preview.cp-btn-layout {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 24px;
}

/* 샘플 텍스트 */
.cp-sample { font-weight: 500; flex: 1; line-height: 1.5; }

/* 항상 가독 가능한 정보 배지 */
.cp-info-badge {
  background: rgba(0,0,0,0.65);
  color: rgba(255,255,255,0.92);
  padding: 5px 10px;
  border-radius: 6px;
  border: 1px solid rgba(255,255,255,0.18);
  display: flex;
  flex-direction: column;
  gap: 3px;
  flex-shrink: 0;
}
.cp-info-badge.centered { align-items: center; }
.cp-ratio  { font-size: 12px; font-family: monospace; font-weight: 600; }
.cp-element { font-size: 10px; opacity: 0.78; }

/* 버튼 테두리 시뮬레이션 */
.cp-btn-demo {
  padding: 10px 28px;
  border-radius: 6px;
  background: transparent;
  font-size: 14px;
  font-weight: 500;
  cursor: default;
  pointer-events: none;
  border-width: 2px;
  border-style: solid;
}
```

### 4-3. JS 변경 — loadQuestion() 수정

```javascript
function loadQuestion() {
  // ... 기존 코드 ...

  // ── 신규: 시각 요소 처리 ──
  const visualContainer = document.getElementById('visualContainer');
  visualContainer.innerHTML = '';
  if (q.visual_type === 'color_pair' && q.visual_code) {
    const cpEl = renderColorPair(q.visual_code);
    if (cpEl) visualContainer.appendChild(cpEl);
  }

  const container = document.getElementById('optionsContainer');
  // ... 기존 코드 ...
}
```

### 4-4. JS 신규 — renderColorPair() 함수

```javascript
function renderColorPair(visualCode) {
  try {
    const vc = JSON.parse(visualCode);
    const wrapper = document.createElement('div');
    wrapper.className = 'color-pair-preview';
    wrapper.style.backgroundColor = vc.bg || '#ffffff';

    const makeBadge = (centered) => {
      const badge = document.createElement('div');
      badge.className = 'cp-info-badge' + (centered ? ' centered' : '');
      const ratio   = document.createElement('span');
      ratio.className = 'cp-ratio';
      ratio.textContent = vc.contrast_ratio || '';
      const element = document.createElement('span');
      element.className = 'cp-element';
      element.textContent = vc.element || '';
      badge.appendChild(ratio);
      badge.appendChild(element);
      return badge;
    };

    if (vc.render_type === 'button_border') {
      wrapper.classList.add('cp-btn-layout');
      const btn = document.createElement('button');
      btn.className = 'cp-btn-demo';
      btn.style.borderColor = vc.text_color;
      btn.style.color = vc.text_color;
      btn.textContent = vc.sample_text || '버튼';
      wrapper.appendChild(btn);
      wrapper.appendChild(makeBadge(true));
    } else {
      wrapper.classList.add('cp-text-layout');
      wrapper.style.color = vc.text_color || '#000000';
      const sample = document.createElement('span');
      sample.className = 'cp-sample';
      sample.style.fontSize = vc.font_size || '14px';
      sample.textContent = vc.sample_text || '게임 UI 텍스트 예시';
      wrapper.appendChild(sample);
      wrapper.appendChild(makeBadge(false));
    }
    return wrapper;
  } catch (e) { return null; }
}
```

---

## 5. 하위 호환성

- `visual_type=none`인 기존 문제(XAG103~123)는 `visualContainer`가 빈 상태로 렌더링되어 기존 동작과 동일
- 컨버터에서 기존 JS 파일(11필드)을 읽을 때 `learning_type`·`visual_type`·`visual_code`가 없으면 기본값(`principle`·`none`·`''`) 자동 적용

---

## 6. 파일 버전 현황

| 파일 | 버전 | 설명 |
|---|---|---|
| `XAG_Quiz_RawData_v26.xlsx` | 보존 기준 | XAG101~123, 873개, 11컬럼 |
| `XAG_Quiz_RawData_v27.xlsx` | 현재 운영 | XAG101~123, 824개, 14컬럼, XAG102 재설계 |
| `XAG_Quiz_Converter_latest.html` | 현재 운영 | 14컬럼 지원, Excel↔JS 양방향 |
| `index.html` | 현재 운영 | visualContainer + renderColorPair() 포함 |
