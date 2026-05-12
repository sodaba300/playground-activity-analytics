# Cursor 작업 요청서
## Activity 기준 분석 그래프 개선 — 레벨별 전체 사용자 수 기준 상대 참여율 적용

현재 `index.html`에는 사용자 레벨 분석 섹션이 아래 순서로 구성되어 있습니다.

```text
1. 전체 레벨 사용자 분포
2. Activity 기준 분석
3. 레벨 기준 분석
```

현재 `Activity 기준 분석`은 선택한 Activity에 대해 레벨별 사용자 수를 절대값으로 보여주고 있습니다.

하지만 레벨마다 전체 사용자 수가 다르기 때문에, 전체 사용자 수가 많은 레벨이 거의 항상 상위로 보이는 문제가 있습니다.

따라서 `Activity 기준 분석`에 **상대 참여율 그래프**를 추가하거나, 기존 그래프를 상대 참여율 기준으로 전환할 수 있도록 수정해주세요.

---

# 문제 정의

현재 Activity 기준 분석은 아래 기준입니다.

```text
선택 Activity의 레벨별 사용자 수
```

예:

```text
ECP7: 1,000명
MAG4: 200명
```

이 경우 ECP7이 더 많이 사용한 것처럼 보입니다.

하지만 각 레벨 전체 사용자 수가 아래와 같다면:

```text
ECP7 전체 사용자: 10,000명
MAG4 전체 사용자: 500명
```

실제 참여율은:

```text
ECP7: 1,000 / 10,000 = 10%
MAG4: 200 / 500 = 40%
```

즉 MAG4가 해당 Activity를 상대적으로 더 많이 사용한 것입니다.

---

# 작업 목표

Activity 기준 분석에서 아래 값을 계산하여 그래프로 보여주세요.

```text
레벨별 상대 참여율(%)
=
선택 Activity의 해당 레벨 사용자 수
/
해당 레벨의 전체 사용자 수
× 100
```

이 그래프의 목적은:

```text
이 Activity를 어떤 레벨이 상대적으로 더 선호하는가?
```

를 확인하는 것입니다.

---

# 현재 유지할 것

아래 기존 섹션은 유지합니다.

```html
<section class="section" id="activity-level-analysis-section">
```

현재 이 섹션 안에는:

```html
<div class="level-activity-tabs" id="levelActivityTabs"></div>
<div class="selected-activity-summary" id="selectedActivitySummary"></div>
<canvas id="activityLevelBarChart"></canvas>
```

구조가 있습니다.

이 구조를 기반으로 수정해주세요.

---

# 추천 UX

## 방식 A 추천: 토글 버튼 추가

Activity 기준 분석 안에서 사용자가 그래프 기준을 선택할 수 있게 해주세요.

```text
[사용자 수] [상대 참여율]
```

- 사용자 수: 기존 절대 사용자 수 그래프
- 상대 참여율: 레벨 전체 사용자 수 대비 참여율 그래프

이 방식이 가장 좋습니다.

절대값과 상대값은 의미가 다르므로 둘 다 볼 수 있어야 합니다.

---

# UI 추가 예시

`selectedActivitySummary` 아래 또는 위에 아래 토글 영역을 추가해주세요.

```html
<div class="activity-chart-mode-toggle" id="activityChartModeToggle">
  <button class="activity-chart-mode-btn active" type="button" onclick="setActivityChartMode('count')">
    사용자 수
  </button>
  <button class="activity-chart-mode-btn" type="button" onclick="setActivityChartMode('rate')">
    상대 참여율
  </button>
</div>
```

---

# CSS 예시

현재 다크 테마와 탭 스타일을 유지해주세요.

```css
.activity-chart-mode-toggle {
  display: flex;
  gap: 6px;
  margin-bottom: 1rem;
}

.activity-chart-mode-btn {
  padding: 5px 12px;
  border-radius: 20px;
  border: 1px solid var(--border2);
  background: transparent;
  color: var(--text2);
  font-size: 12px;
  font-family: 'Pretendard', sans-serif;
  cursor: pointer;
  transition: all 0.15s;
}

.activity-chart-mode-btn:hover {
  background: var(--bg3);
  color: var(--text);
}

.activity-chart-mode-btn.active {
  background: var(--accent);
  border-color: var(--accent);
  color: #fff;
}
```

---

# 차트 제목 변경

현재 제목:

```text
선택 Activity의 레벨별 사용자 수
```

토글 상태에 따라 의미가 바뀌도록 해주세요.

추천:

```text
사용자 수 모드: 선택 Activity의 레벨별 사용자 수
상대 참여율 모드: 선택 Activity의 레벨별 상대 참여율
```

가능하면 `chart-title` 안의 텍스트를 JS로 업데이트해주세요.

예:

```html
<span id="activityLevelChartTitleText">선택 Activity의 레벨별 사용자 수</span>
```

---

# 데이터 계산 기준

## 전체 레벨 사용자 수

현재 `전체 레벨 사용자 분포` 그래프에 사용되는 데이터가 있을 것입니다.

예상 구조는 다음 중 하나일 수 있습니다.

```javascript
const levelUserOverviewData = ...
```

또는

```javascript
const levelTotals = ...
```

정확한 변수명이 다르다면 현재 파일에 있는 전체 레벨 사용자 분포 차트 데이터를 재사용해주세요.

없다면 `activityLevelData` 전체를 순회해서 레벨별 총합을 계산해주세요.

---

# 레벨별 전체 사용자 수 계산 함수

아래 함수를 추가해주세요.

```javascript
function buildLevelTotalMap() {
  const totals = {};

  levelOrder.forEach(level => {
    totals[level] = 0;
  });

  Object.values(activityLevelData).forEach(activity => {
    levelOrder.forEach(level => {
      totals[level] += Number(activity.levels[level] || 0);
    });
  });

  return totals;
}

const levelTotalMap = buildLevelTotalMap();
```

단, 이미 전체 레벨 사용자 분포에서 사용하는 정확한 총 사용자 수 데이터가 있다면 그 데이터를 우선 사용해주세요.

---

# 상대 참여율 계산 공식

```javascript
const rate = levelTotal > 0
  ? (activityLevelCount / levelTotal) * 100
  : 0;
```

---

# 전역 상태 추가

아래 변수를 추가해주세요.

```javascript
let activityChartMode = 'count'; // 'count' | 'rate'
```

---

# 토글 함수 추가

```javascript
function setActivityChartMode(mode) {
  activityChartMode = mode;

  document.querySelectorAll('.activity-chart-mode-btn').forEach(btn => {
    const isActive =
      (mode === 'count' && btn.textContent.includes('사용자 수')) ||
      (mode === 'rate' && btn.textContent.includes('상대 참여율'));

    btn.classList.toggle('active', isActive);
  });

  if (selectedActivityName) {
    updateSelectedActivitySummary(selectedActivityName);
    renderActivityLevelBarChart(selectedActivityName);
  }
}
```

---

# renderActivityLevelBarChart 수정 요구사항

기존 `renderActivityLevelBarChart(activityName)` 함수를 수정해주세요.

## 사용자 수 모드

기존과 동일:

```text
Y축 = 사용자 수
```

## 상대 참여율 모드

```text
Y축 = 상대 참여율(%)
```

데이터:

```javascript
const rates = labels.map(level => {
  const count = Number(activity.levels[level] || 0);
  const levelTotal = Number(levelTotalMap[level] || 0);
  return levelTotal > 0 ? (count / levelTotal) * 100 : 0;
});
```

---

# 막대 강조 기준 변경

## 사용자 수 모드

가장 사용자 수가 높은 레벨 강조.

## 상대 참여율 모드

가장 상대 참여율이 높은 레벨 강조.

---

# Tooltip 요구사항

## 사용자 수 모드 tooltip

```text
레벨: ECP7
사용자 수: 1,000명
해당 Activity 내 비중: 12.8%
```

## 상대 참여율 모드 tooltip

```text
레벨: MAG4
Activity 사용자 수: 200명
레벨 전체 사용자 수: 500명
상대 참여율: 40.0%
```

---

# Y축 설정

## 사용자 수 모드

기존 유지.

```javascript
ticks: {
  callback: function(value) {
    return Number(value).toLocaleString();
  }
}
```

## 상대 참여율 모드

```javascript
ticks: {
  callback: function(value) {
    return value + '%';
  }
}
```

상대 참여율은 소수점 1자리까지 표현해주세요.

---

# Summary 영역 수정

현재 summary는 아래와 비슷할 것입니다.

```text
총 사용자 12,345명 · 대표 레벨 ECP7 · 상위 3개 레벨 비중 42.5%
```

상대 참여율 모드에서는 아래처럼 변경해주세요.

```text
총 사용자 12,345명 · 상대 참여율 1위 MAG4 40.0% · 레벨 전체 대비 사용률 기준
```

사용자 수 모드에서는 기존 문구 유지.

---

# 차트 색상

현재와 동일하게 유지하되, 상대 참여율 모드에서는 강조색을 `var(--amber)` 또는 `#fbbf24`로 사용해주세요.

추천:

## 사용자 수 모드
```javascript
기본: rgba(45, 212, 191, 0.85)
강조: rgba(108, 127, 255, 0.95)
```

## 상대 참여율 모드
```javascript
기본: rgba(96, 165, 250, 0.75)
강조: rgba(251, 191, 36, 0.95)
```

---

# 구현 예시

아래 코드를 참고해서 기존 함수를 수정해주세요.

```javascript
let activityChartMode = 'count';

function setActivityChartMode(mode) {
  activityChartMode = mode;

  document.querySelectorAll('.activity-chart-mode-btn').forEach(btn => {
    btn.classList.remove('active');
  });

  const targetBtn = document.querySelector(`[data-activity-chart-mode="${mode}"]`);
  if (targetBtn) targetBtn.classList.add('active');

  if (selectedActivityName) {
    updateSelectedActivitySummary(selectedActivityName);
    renderActivityLevelBarChart(selectedActivityName);
  }
}
```

HTML은 아래처럼 data attribute를 사용하는 방식이 더 안정적입니다.

```html
<div class="activity-chart-mode-toggle">
  <button
    class="activity-chart-mode-btn active"
    data-activity-chart-mode="count"
    type="button"
    onclick="setActivityChartMode('count')"
  >
    사용자 수
  </button>
  <button
    class="activity-chart-mode-btn"
    data-activity-chart-mode="rate"
    type="button"
    onclick="setActivityChartMode('rate')"
  >
    상대 참여율
  </button>
</div>
```

---

# renderActivityLevelBarChart 예시

```javascript
function renderActivityLevelBarChart(activityName) {
  const canvas = document.getElementById('activityLevelBarChart');
  const activity = activityLevelData[activityName];
  if (!canvas || !activity) return;

  const ctx = canvas.getContext('2d');

  const labels = levelOrder;

  const counts = labels.map(level => Number(activity.levels[level] || 0));
  const totalActivityUsers = counts.reduce((sum, value) => sum + value, 0);

  const rates = labels.map((level, index) => {
    const levelTotal = Number(levelTotalMap[level] || 0);
    return levelTotal > 0 ? (counts[index] / levelTotal) * 100 : 0;
  });

  const values = activityChartMode === 'rate' ? rates : counts;
  const maxValue = Math.max(...values);

  const backgroundColors = values.map(value => {
    if (value === maxValue && value > 0) {
      return activityChartMode === 'rate'
        ? 'rgba(251, 191, 36, 0.95)'
        : 'rgba(108, 127, 255, 0.95)';
    }

    return activityChartMode === 'rate'
      ? 'rgba(96, 165, 250, 0.75)'
      : 'rgba(45, 212, 191, 0.85)';
  });

  const borderColors = values.map(value => {
    if (value === maxValue && value > 0) {
      return activityChartMode === 'rate'
        ? '#fbbf24'
        : '#6c7fff';
    }

    return activityChartMode === 'rate'
      ? '#60a5fa'
      : '#2dd4bf';
  });

  if (activityLevelBarChart) {
    activityLevelBarChart.destroy();
  }

  activityLevelBarChart = new Chart(ctx, {
    type: 'bar',
    data: {
      labels,
      datasets: [{
        label: activityChartMode === 'rate' ? '상대 참여율' : '사용자 수',
        data: values,
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
              const level = context.label;
              const index = context.dataIndex;
              const count = counts[index];
              const levelTotal = Number(levelTotalMap[level] || 0);
              const activityShare = totalActivityUsers > 0
                ? ((count / totalActivityUsers) * 100).toFixed(1)
                : '0.0';
              const rate = levelTotal > 0
                ? ((count / levelTotal) * 100).toFixed(1)
                : '0.0';

              if (activityChartMode === 'rate') {
                return [
                  `Activity 사용자 수: ${count.toLocaleString()}명`,
                  `레벨 전체 사용자 수: ${levelTotal.toLocaleString()}명`,
                  `상대 참여율: ${rate}%`
                ];
              }

              return [
                `사용자 수: ${count.toLocaleString()}명`,
                `해당 Activity 내 비중: ${activityShare}%`
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
              if (activityChartMode === 'rate') {
                return Number(value).toFixed(1) + '%';
              }
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

# updateSelectedActivitySummary 수정 예시

```javascript
function updateSelectedActivitySummary(activityName) {
  const summaryEl = document.getElementById('selectedActivitySummary');
  const activity = activityLevelData[activityName];
  if (!summaryEl || !activity) return;

  const values = levelOrder.map(level => ({
    level,
    count: Number(activity.levels[level] || 0),
    levelTotal: Number(levelTotalMap[level] || 0)
  }));

  const total = values.reduce((sum, item) => sum + item.count, 0);

  if (activityChartMode === 'rate') {
    const rateValues = values.map(item => ({
      ...item,
      rate: item.levelTotal > 0 ? (item.count / item.levelTotal) * 100 : 0
    })).sort((a, b) => b.rate - a.rate);

    const topRate = rateValues[0] || { level: '-', rate: 0 };

    summaryEl.innerHTML = `
      <div class="summary-main">${activityName}</div>
      <div class="summary-sub">
        총 사용자 ${total.toLocaleString()}명 · 상대 참여율 1위 ${topRate.level} ${topRate.rate.toFixed(1)}% · 레벨 전체 대비 사용률 기준
      </div>
    `;

    return;
  }

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
```

---

# 초기 상태

초기 모드는 사용자 수로 두세요.

```javascript
let activityChartMode = 'count';
```

사용자가 `상대 참여율` 버튼을 누르면 상대 참여율 그래프로 전환됩니다.

---

# 중요 요구사항

- 기존 Activity 탭 기능은 유지합니다.
- 기존 레벨 기준 분석 섹션은 건드리지 않아도 됩니다.
- 이번 수정 대상은 `Activity 기준 분석`입니다.
- `사용자 수`와 `상대 참여율`을 토글할 수 있어야 합니다.
- 상대 참여율은 반드시 레벨별 전체 사용자 수를 분모로 계산해야 합니다.
- 레벨 전체 사용자 수가 0이면 참여율은 0으로 처리합니다.
- 새 라이브러리 추가 금지.
- 기존 Chart.js만 사용합니다.
- 현재 다크 테마와 카드 디자인을 유지합니다.

---

# 최종 기대 결과

사용자는 Activity를 선택한 뒤:

```text
사용자 수 모드
→ 실제로 어떤 레벨 학생이 많이 사용했는지 확인

상대 참여율 모드
→ 각 레벨 전체 사용자 수 대비 어떤 레벨이 상대적으로 더 많이 사용했는지 확인
```

할 수 있어야 합니다.

이렇게 하면 단순히 전체 사용자 수가 많은 레벨이 항상 1등으로 보이는 문제를 보완할 수 있습니다.
