# Cursor 작업 요청서
## 레벨 기준 분석 그래프 추가 — 레벨 탭 + Activity별 막대그래프

현재 `index.html`에는 아래 구조가 이미 구현되어 있습니다.

```text
1. 전체 레벨 사용자 분포
2. Activity 기준 분석
```

이번에는 아래 섹션을 추가해주세요.

```text
3. 레벨 기준 분석
```

---

# 작업 목표

사용자가 특정 레벨(ECP5, GT1, MAG2 등)을 선택하면:

```text
해당 레벨 학생들이 어떤 Activity를 많이 사용하는가?
```

를 확인할 수 있는 막대그래프를 표시합니다.

---

# 핵심 UX 방향

현재 구현된:

```text
Activity 기준 분석
```

과 동일한 UX 패턴을 사용합니다.

즉:

```text
Activity 탭 클릭
→ 레벨별 사용자 수 그래프 표시
```

구조를 반대로 바꿉니다.

이번에는:

```text
레벨 탭 클릭
→ Activity별 사용자 수 그래프 표시
```

형태입니다.

---

# 추가 위치

아래 section 바로 아래에 추가해주세요.

```html
<section class="section" id="activity-level-analysis-section">
```

즉 최종 순서는:

```text
1. 전체 레벨 사용자 분포
2. Activity 기준 분석
3. 레벨 기준 분석 ← 이번 작업
```

---

# 섹션 구조

아래 HTML 구조를 참고해주세요.

```html
<section class="section" id="level-activity-analysis-section">
  <div class="section-head">
    <h2>레벨 기준 분석</h2>
    <div class="section-divider"></div>
  </div>

  <div class="chart-card">
    <div class="chart-title">
      <span class="dot" style="background:var(--purple)"></span>
      선택 레벨의 Activity별 사용자 수
    </div>

    <div class="level-tabs" id="levelTabs"></div>

    <div class="selected-level-summary" id="selectedLevelSummary"></div>

    <div style="position:relative;height:420px;">
      <canvas id="levelActivityBarChart" role="img" aria-label="선택 레벨 Activity별 사용자 수 차트">
        선택 레벨 Activity별 사용자 수 차트
      </canvas>
    </div>
  </div>
</section>
```

---

# 디자인 요구사항

현재 Activity 기준 분석 스타일과 최대한 동일하게 구현해주세요.

추천 클래스명:

```css
.level-tabs
.level-tab
.level-tab.active
.selected-level-summary
```

---

# 탭 디자인

레벨 수가 많기 때문에:

- flex-wrap 허용
- max-height 지정
- overflow-y auto

구조 유지해주세요.

---

# 선택 레벨 요약 영역

그래프 위 summary 카드 표시해주세요.

예시:

```text
ECP7
총 사용자 16,024명 · 최다 사용 Activity Magic Word World · 상위 3개 Activity 비중 48.3%
```

---

# 그래프 요구사항

## 그래프 종류

Chart.js 세로 막대그래프 사용.

```javascript
type: 'bar'
```

---

# 그래프 기준

선택된 레벨 하나에 대해:

```text
X축 = Activity명
Y축 = 사용자 수
```

를 표시합니다.

---

# Activity 정렬 기준

사용자 수 높은 순 DESC 정렬해주세요.

---

# 막대 색상

기본 막대:

```javascript
backgroundColor: 'rgba(45, 212, 191, 0.85)'
borderColor: '#2dd4bf'
```

최다 사용 Activity는 강조해주세요.

```javascript
backgroundColor: 'rgba(167, 139, 250, 0.95)'
borderColor: '#a78bfa'
```

---

# Tooltip 요구사항

hover 시:

```text
Activity: Magic Word World
사용자 수: 4,320명
비율: 12.8%
```

표시해주세요.

---

# 데이터 구조

현재 존재하는:

```javascript
const activityLevelData
```

를 기반으로 역방향 데이터를 생성해주세요.

즉:

```javascript
const levelActivityData = {
  "ECP7": {
    total: 0,
    activities: {
      "Magic Word World": 0,
      "Voice Actor": 0,
      "Story Builder": 0
    }
  }
}
```

형태로 생성해주세요.

---

# 데이터 생성 방식

```javascript
function buildLevelActivityData() {
  const result = {};

  Object.entries(activityLevelData).forEach(([activityName, activity]) => {
    Object.entries(activity.levels).forEach(([levelName, count]) => {
      if (!result[levelName]) {
        result[levelName] = {
          total: 0,
          activities: {}
        };
      }

      result[levelName].activities[activityName] = Number(count || 0);
      result[levelName].total += Number(count || 0);
    });
  });

  return result;
}

const levelActivityData = buildLevelActivityData();
```

---

# 필수 함수

```javascript
renderLevelTabs()
selectLevel(levelName)
updateSelectedLevelSummary(levelName)
renderLevelActivityBarChart(levelName)
```

---

# 초기 실행

```javascript
renderLevelTabs();
```

---

# 중요 요구사항

- 기존 Activity 기준 분석은 유지합니다.
- 레벨 기준 분석을 새 section으로 추가합니다.
- Activity 기준 분석과 UX 패턴을 통일해주세요.
- 현재 다크 테마 유지해주세요.
- 새 라이브러리 추가 금지.
- Chart.js만 사용해주세요.

---

# 최종 기대 결과

사용자는:

```text
[ECP7] 클릭
→ ECP7 학생들이 많이 사용하는 Activity 막대그래프 확인

[GT2] 클릭
→ GT2 학생들의 Activity 사용 패턴 확인
```

할 수 있어야 합니다.

이 구조를 통해:

```text
특정 레벨 학생들이 어떤 콘텐츠에 편중되어 있는가?
```

를 직관적으로 분석할 수 있어야 합니다.
