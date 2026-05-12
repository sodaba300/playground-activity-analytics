# Cursor 작업 요청서
## Activity 기준 분석 그래프 수정 — 탭 방식 레벨별 막대그래프 적용

현재 `index.html`의 **사용자 레벨 분석** 섹션에 있는 Activity 기준 분석 그래프를 수정해주세요.

기존처럼 모든 Activity와 모든 레벨을 한 화면에 100% stacked bar chart로 보여주면 레벨 종류가 너무 많아 가독성이 떨어집니다.

따라서 아래 방식으로 변경해주세요.

---

# 작업 목표

## 기존 방식 제거 또는 대체

현재 적용되어 있거나 적용 예정인:

```text
Activity별 레벨 구성 비율
100% Stacked Horizontal Bar Chart
```

방식은 사용하지 않습니다.

레벨 종류가 많기 때문에 한 화면에서 모든 Activity × 모든 레벨을 동시에 보여주면 너무 복잡합니다.

---

# 변경할 방식

## Activity 탭 버튼 + 선택 Activity 레벨별 막대그래프

상단에는 Activity 버튼을 탭 형태로 나열하고, 사용자가 Activity를 클릭하면 해당 Activity의 레벨별 사용자 수 막대그래프가 표시되도록 해주세요.

즉, 한 번에 하나의 Activity만 보여줍니다.

```text
[Magic Word World] [Emoji Story Chain] [Treasure Hunter] [Voice Actor] ...
↓
선택된 Activity의 레벨별 사용자 수 막대그래프
```

---

# 추가 위치

현재 HTML의 아래 섹션 내부 또는 바로 아래에 추가해주세요.

```html
<section class="section" id="level-overview-section">
```

현재 이 섹션에는:

```text
전체 레벨 사용자 분포 (사용자 수 기준)
```

그래프가 있습니다.

그 아래에 다음 순서로 추가해주세요.

```text
1. 전체 레벨 사용자 분포
2. Activity 기준 분석
```

---

# 섹션 제목

```html
<div class="section-head">
  <h2>Activity 기준 분석</h2>
  <div class="section-divider"></div>
</div>
```

---

# UI 구조 예시

아래 구조를 참고해서 구현해주세요.

```html
<section class="section" id="activity-level-analysis-section">
  <div class="section-head">
    <h2>Activity 기준 분석</h2>
    <div class="section-divider"></div>
  </div>

  <div class="chart-card">
    <div class="chart-title">
      <span class="dot" style="background:var(--accent)"></span>
      선택 Activity의 레벨별 사용자 수
    </div>

    <div class="level-activity-tabs" id="levelActivityTabs"></div>

    <div class="selected-activity-summary" id="selectedActivitySummary"></div>

    <div style="position:relative;height:420px;">
      <canvas id="activityLevelBarChart" role="img" aria-label="선택 Activity 레벨별 사용자 수 차트">
        선택 Activity 레벨별 사용자 수 차트
      </canvas>
    </div>
  </div>
</section>
```

---

# 디자인 요구사항

현재 리포트의 다크 테마와 기존 탭 스타일을 최대한 재사용해주세요.

현재 사용자 의견 섹션에 이미 아래와 같은 탭 스타일이 있습니다.

```css
.tabs-nav
.tab-btn
.tab-btn.active
```

Activity 기준 분석에서도 비슷한 느낌으로 구현해주세요.

단, 기존 사용자 의견 탭과 충돌하지 않도록 클래스명은 새로 만들어도 됩니다.

추천 클래스명:

```css
.level-activity-tabs
.level-activity-tab
.level-activity-tab.active
.selected-activity-summary
```

---

# 탭 디자인 요구사항

Activity가 많으므로 탭 영역은 줄바꿈 또는 가로/세로 스크롤이 가능해야 합니다.

추천 CSS:

```css
.level-activity-tabs {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 1rem;
  max-height: 160px;
  overflow-y: auto;
}

.level-activity-tab {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 5px 10px;
  border-radius: 20px;
  border: 1px solid var(--border2);
  background: transparent;
  color: var(--text2);
  font-size: 12px;
  font-family: 'Pretendard', sans-serif;
  cursor: pointer;
  transition: all 0.15s;
  white-space: nowrap;
}

.level-activity-tab:hover {
  background: var(--bg3);
  color: var(--text);
}

.level-activity-tab.active {
  background: var(--accent);
  border-color: var(--accent);
  color: #fff;
}
```

---

# 선택 Activity 요약 영역

그래프 위에 선택된 Activity 정보를 간단히 표시해주세요.

예시:

```text
Magic Word World
총 사용자 12,345명 · 대표 레벨 ECP7 · 상위 3개 레벨 비중 42.5%
```

HTML 예시:

```html
<div class="selected-activity-summary" id="selectedActivitySummary">
  <div class="summary-main">Magic Word World</div>
  <div class="summary-sub">총 사용자 12,345명 · 대표 레벨 ECP7 · 상위 3개 레벨 비중 42.5%</div>
</div>
```

추천 CSS:

```css
.selected-activity-summary {
  background: var(--bg3);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  padding: 0.9rem 1rem;
  margin-bottom: 1rem;
}

.selected-activity-summary .summary-main {
  font-size: 15px;
  font-weight: 600;
  color: var(--text);
}

.selected-activity-summary .summary-sub {
  font-size: 12px;
  color: var(--text2);
  margin-top: 3px;
  font-family: 'DM Mono', monospace;
}
```

---

# 그래프 요구사항

## 그래프 종류

Chart.js 세로 막대그래프를 사용해주세요.

```javascript
type: 'bar'
```

---

# 그래프 기준

선택된 Activity 하나에 대해:

```text
X축 = 레벨명
Y축 = 사용자 수
```

를 표시합니다.

예:

```text
ECP5, ECP6, ECP6m, ECP7, ECP7i, GT1, GT2, ...
```

---

# 색상

막대 색상은 현재 리포트의 `--teal` 계열을 사용해주세요.

기본 막대:

```javascript
backgroundColor: 'rgba(45, 212, 191, 0.85)'
borderColor: '#2dd4bf'
```

대표 레벨, 즉 사용자 수가 가장 높은 레벨은 강조해주세요.

대표 레벨 막대:

```javascript
backgroundColor: 'rgba(108, 127, 255, 0.95)'
borderColor: '#6c7fff'
```

---

# Tooltip 요구사항

마우스 hover 시 아래 정보를 보여주세요.

```text
레벨: ECP7
사용자 수: 16,024명
비율: 12.8%
```

---

# 데이터 구조 요구사항

아래와 같은 데이터 구조를 만들어 사용해주세요.

실제 값은 첨부된 `AI_PLAYGROUND_레벨별이용자수.xlsx` 데이터를 기준으로 넣어주세요.

```javascript
const activityLevelData = {
  "Magic Word World": {
    total: 0,
    levels: {
      "ECP5": 0,
      "ECP6": 0,
      "ECP6m": 0,
      "ECP7": 0,
      "ECP7i": 0,
      "ECP7m": 0,
      "GT1": 0,
      "GT2": 0,
      "GT3": 0,
      "GT4": 0,
      "Honors 1": 0,
      "Honors 2": 0,
      "Honors 3": 0,
      "Leaders 1": 0,
      "Leaders 2": 0,
      "Leaders 3": 0,
      "LX A": 0,
      "LX B": 0,
      "LX C": 0,
      "LX D": 0,
      "LX E": 0,
      "S1": 0,
      "MAG2": 0,
      "MAG1": 0,
      "S2": 0,
      "MAG3": 0,
      "MGT1": 0,
      "MGT2": 0,
      "S3": 0,
      "MGT3": 0,
      "MAG4": 0,
      "S4": 0,
      "MGT4": 0,
      "SSPC": 0,
      "SSPD": 0
    }
  }
};
```

---

# 레벨 순서 고정

레벨 순서는 Activity마다 달라지면 안 됩니다.

반드시 아래 순서로 고정해주세요.

```javascript
const levelOrder = [
  'ECP5',
  'ECP6',
  'ECP6m',
  'ECP7',
  'ECP7i',
  'ECP7m',
  'GT1',
  'GT2',
  'GT3',
  'GT4',
  'Honors 1',
  'Honors 2',
  'Honors 3',
  'Leaders 1',
  'Leaders 2',
  'Leaders 3',
  'LX A',
  'LX B',
  'LX C',
  'LX D',
  'LX E',
  'S1',
  'MAG2',
  'MAG1',
  'S2',
  'MAG3',
  'MGT1',
  'MGT2',
  'S3',
  'MGT3',
  'MAG4',
  'S4',
  'MGT4',
  'SSPC',
  'SSPD'
];
```

없는 레벨은 0으로 처리해주세요.

---

# Chart.js 구현 예시

아래 구조를 참고해주세요.

```javascript
let activityLevelBarChart = null;
let selectedActivityName = null;

function renderActivityLevelTabs() {
  const tabWrap = document.getElementById('levelActivityTabs');
  if (!tabWrap) return;

  const activities = Object.keys(activityLevelData);

  tabWrap.innerHTML = activities.map((activityName, index) => `
    <button
      class="level-activity-tab ${index === 0 ? 'active' : ''}"
      type="button"
      onclick="selectActivityLevel('${activityName.replace(/'/g, "\\'")}')"
    >
      ${activityName}
    </button>
  `).join('');

  if (activities.length > 0) {
    selectActivityLevel(activities[0]);
  }
}

function selectActivityLevel(activityName) {
  selectedActivityName = activityName;

  document.querySelectorAll('.level-activity-tab').forEach(btn => {
    btn.classList.toggle('active', btn.textContent.trim() === activityName);
  });

  updateSelectedActivitySummary(activityName);
  renderActivityLevelBarChart(activityName);
}

function updateSelectedActivitySummary(activityName) {
  const summaryEl = document.getElementById('selectedActivitySummary');
  const activity = activityLevelData[activityName];
  if (!summaryEl || !activity) return;

  const values = levelOrder.map(level => ({
    level,
    count: Number(activity.levels[level] || 0)
  }));

  const total = values.reduce((sum, item) => sum + item.count, 0);
  const sorted = [...values].sort((a, b) => b.count - a.count);
  const topLevel = sorted[0] || { level: '-', count: 0 };
  const top3Total = sorted.slice(0, 3).reduce((sum, item) => sum + item.count, 0);
  const top3Pct = total > 0 ? ((top3Total / total) * 100).toFixed(1) : '0.0';

  summaryEl.innerHTML = `
    <div class="summary-main">${activityName}</div>
    <div class="summary-sub">
      총 사용자 ${total.toLocaleString()}명 · 대표 레벨 ${topLevel.level} · 상위 3개 레벨 비중 ${top3Pct}%
    </div>
  `;
}

function renderActivityLevelBarChart(activityName) {
  const canvas = document.getElementById('activityLevelBarChart');
  const activity = activityLevelData[activityName];
  if (!canvas || !activity) return;

  const ctx = canvas.getContext('2d');

  const labels = levelOrder;
  const counts = labels.map(level => Number(activity.levels[level] || 0));
  const total = counts.reduce((sum, value) => sum + value, 0);
  const maxValue = Math.max(...counts);

  const backgroundColors = counts.map(value =>
    value === maxValue && value > 0
      ? 'rgba(108, 127, 255, 0.95)'
      : 'rgba(45, 212, 191, 0.85)'
  );

  const borderColors = counts.map(value =>
    value === maxValue && value > 0
      ? '#6c7fff'
      : '#2dd4bf'
  );

  if (activityLevelBarChart) {
    activityLevelBarChart.destroy();
  }

  activityLevelBarChart = new Chart(ctx, {
    type: 'bar',
    data: {
      labels,
      datasets: [{
        label: '사용자 수',
        data: counts,
        backgroundColor: backgroundColors,
        borderColor: borderColors,
        borderWidth: 1,
        borderRadius: 4,
        maxBarThickness: 28
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: {
          display: false
        },
        tooltip: {
          backgroundColor: '#13161d',
          titleColor: '#e8eaf0',
          bodyColor: '#8b90a0',
          borderColor: 'rgba(255,255,255,0.12)',
          borderWidth: 1,
          callbacks: {
            title: function(context) {
              return `레벨: ${context[0].label}`;
            },
            label: function(context) {
              const value = Number(context.raw || 0);
              const pct = total > 0 ? ((value / total) * 100).toFixed(1) : '0.0';
              return [
                `사용자 수: ${value.toLocaleString()}명`,
                `비율: ${pct}%`
              ];
            }
          }
        }
      },
      scales: {
        x: {
          grid: {
            color: 'rgba(255,255,255,0.05)'
          },
          ticks: {
            color: '#8b90a0',
            maxRotation: 55,
            minRotation: 45,
            font: {
              size: 10
            }
          }
        },
        y: {
          beginAtZero: true,
          grid: {
            color: 'rgba(255,255,255,0.05)'
          },
          ticks: {
            color: '#8b90a0',
            callback: function(value) {
              return Number(value).toLocaleString();
            }
          }
        }
      }
    }
  });
}
```

---

# 초기 실행

기존 차트 초기화 코드가 있는 곳에서 아래 함수를 호출해주세요.

```javascript
renderActivityLevelTabs();
```

예:

```javascript
document.addEventListener('DOMContentLoaded', function() {
  renderActivityLevelTabs();
});
```

이미 DOMContentLoaded 또는 init 함수가 있으면 그 안에 추가해주세요.

---

# 기존 그래프와 충돌 방지

아래 ID를 사용하는 기존 그래프가 있다면 제거하거나 사용하지 않도록 해주세요.

```text
activityLevelDistributionChart
```

이번 방식에서는 아래 ID를 사용합니다.

```text
activityLevelBarChart
levelActivityTabs
selectedActivitySummary
```

---

# 주의사항

- 기존 전체 레벨 사용자 분포 그래프 `levelUserOverviewChart`는 유지합니다.
- Activity 기준 분석만 탭 방식으로 변경합니다.
- 레벨을 LV1~LV5로 그룹핑하지 않습니다.
- 실제 레벨명 그대로 사용합니다.
- 사용자가 한 번에 하나의 Activity 레벨 분포만 확인할 수 있게 합니다.
- 현재 리포트의 디자인 톤을 유지합니다.
- 새 라이브러리는 추가하지 말고 기존 Chart.js만 사용합니다.

---

# 최종 기대 결과

사용자는 Activity 버튼을 클릭하면서 각 콘텐츠별 레벨 사용자 분포를 확인할 수 있습니다.

예:

```text
Magic Word World 클릭
→ Magic Word World의 ECP5~SSPD 레벨별 사용자 수 막대그래프 표시

Voice Actor 클릭
→ Voice Actor의 ECP5~SSPD 레벨별 사용자 수 막대그래프 표시
```

이 방식은 기존 100% stacked chart보다 가독성이 좋고, 레벨 데이터를 그룹화하지 않아도 되어 실제 운영 레벨 구조를 그대로 보여줄 수 있습니다.
