# XAG Quiz 업데이트 내역

## 버전 정보

| 구분 | 버전 |
|---|---|
| 퀴즈 데이터 (Excel) | **v28** |
| 퀴즈 게임 (index.html) | **v1.21** |
| 이전 버전 | 데이터 v27 / 게임 v1.2 |

---

## 1. XAG101 퀴즈 전면 재설계

### 문제 수 및 유형 변경

| 항목 | v27 (이전) | v28 (이후) |
|---|---|---|
| 문제 수 | 49개 | 35개 |
| 원칙·정의 확인 | 49개 (100%) | 11개 (31%) |
| 판단·적용 | 0개 | 17개 (49%) |
| 설계 오류 발견 (시각) | 0개 | 7개 (20%) |

### 시각 문제 7개 구성

| key | render_type | 학습 내용 |
|---|---|---|
| V001 | font_compare | 산세리프 vs 장식체 서체 비교 |
| V002 | font_compare | 속공간 넓음 vs 좁음 (Filling-in 현상) |
| V003 | caps_compare | 자간 기준 충족(0.08em) vs 미달(-0.04em) |
| V004 | text_align | 왼쪽 정렬 vs 가운데 정렬 (Return Sweep) |
| V005 | caps_compare | Sentence case vs ALL CAPS |
| V006 | caps_compare | 행간 1.5배 vs 1.0배 |
| V007 | caps_compare | 전체 텍스트 vs 약축(Truncation) |

### 개선 원칙
- 원칙/정의 암기형 과잉 → 판단·적용 중심으로 재편
- 텍스트 스케일링·Reflow·동적 컨테이너 등 새 XAG101 PDF 내용 반영
- 기존 중복 문제(양끝 정렬 4개 반복, Return Sweep 2개 연속 등) 통합·제거

---

## 2. 게임 앱 업데이트 (index.html v1.2 → v1.3)

### 신규 비교 렌더러 3종 추가

| render_type | 렌더링 방식 | visual_code 핵심 필드 |
|---|---|---|
| `font_compare` | 동일 텍스트를 두 서체로 나란히 | `text`, `font_a/b`, `label_a/b` |
| `text_align` | 동일 텍스트를 두 정렬로 나란히 | `text`, `align_a/b`, `label_a/b` |
| `caps_compare` | 두 텍스트·형식을 나란히 (범용) | `text_a/b`, `label_a/b`, `style_a/b` |

세 타입 모두 `visual_type: color_pair` + `visual_code` JSON으로 동작하며 기존 문제와 완전 하위 호환됩니다.

### panel_style 지원 추가 (caps_compare)

`caps_compare` 타입에서 각 패널의 배경 스타일을 개별 지정할 수 있습니다.

```json
{
  "render_type": "caps_compare",
  "panel_style_a": { "background": "linear-gradient(...)" },
  "panel_style_b": { "background": "linear-gradient(...)" }
}
```

향후 배경 박스(Backdrop) 유무 비교 등 패널 배경 차별화가 필요한 문제에 데이터만으로 구현 가능합니다.

---

## 3. 버그 수정 및 데이터 정제

### OX / SA 문제 answer_index 누락 버그 수정
- 영향 범위: XAG103~123 전체 OX 문제(6개×21섹션) + SA 문제 전체
- 증상: `answer_index` 컬럼이 `None`으로 비어 있어 게임에서 정답 처리 오류 가능
- 조치: 해당 컬럼을 `1` (또는 정답 위치에 맞게 `2`)로 일괄 설정

### 시각 문제 힌트 노출 수정
- **V001**: 보기 `"A — 산세리프(Sans-serif) 서체"` → `"A"` (라벨 포함)
- **V002**: 보기·라벨 `"A — 속공간 넓은 서체"` → `"A"` (이전 버전에서 처리)
- **V006**: 라벨에서 `"(권장)"` 제거

### 기타
- `XAG103_Q017`: 이모지(🔧) 렌더링 깨짐 수정

---

## 4. 파일 현황

| 파일 | 설명 |
|---|---|
| `XAG_Quiz_RawData_v26.xlsx` | 보존 기준 (873개, 11컬럼) |
| `XAG_Quiz_RawData_v27.xlsx` | 중간 버전 (810개, 14컬럼) |
| `XAG_Quiz_RawData_v28.xlsx` | **현재 운영** (810개, 14컬럼) |
| `index.html` | **현재 운영** (v1.3) |
| `XAG_Quiz_Converter_latest.html` | v27 이후 동일 (14컬럼 지원) |



## v28 추가 수정 (게임 v1.3 반영)

### dialog_compare 렌더러 추가 (index.html)

게임 다이얼로그 UI 스타일로 두 정렬 방식을 나란히 비교하는 신규 render_type.

**visual_code 필드:**

| 필드 | 설명 |
|---|---|
| `render_type` | `"dialog_compare"` |
| `title` | 다이얼로그 타이틀 텍스트 |
| `text` | 본문 텍스트 |
| `button` | 하단 버튼 텍스트 |
| `label_a/b` | 각 패널 라벨 |
| `align_a/b` | 각 패널 텍스트 정렬 (`left` / `center` / `justify`) |

### XAG101_V004 visual_code 변경

`text_align` → `dialog_compare`로 교체.
왼쪽 정렬(A) vs 가운데 정렬(B)을 실제 게임 튜토리얼 다이얼로그 UI로 표시.
