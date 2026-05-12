# Cursor 작업 요청: 전체 레벨 사용자 분포 그래프 추가

## 목표
기존 `index.html` 리포트에 **“1. 전체 레벨 사용자 분포(사용자 수 기준)”** 그래프를 추가해주세요.

이번 작업은 레벨 분석 섹션의 첫 번째 단계입니다.  
Activity별 세부 분석이나 레벨별 TOP 콘텐츠 분석은 아직 추가하지 말고, **전체 레벨 사용자 분포 막대그래프 1개만** 먼저 적용해주세요.

---

## 참고 파일

### 1. 기존 리포트 파일
- 파일명: `index.html`
- 현재 Chart.js 4.4.1을 사용하고 있음
- 기존 리포트의 다크 테마, `.section`, `.chart-card`, `.chart-title`, `.chart-legend` 스타일을 그대로 유지해주세요.

### 2. 레벨별 사용자 수 데이터 파일
- 파일명: `AI_PLAYGROUND_레벨별이용자수.xlsx`
- 시트명: `Sheet1`
- 컬럼 구조:

| 컬럼명 | 설명 |
|---|---|
| `ACTIVITY_SEQ` | Activity 번호 |
| `ACTIVITY_NAME` | Activity 이름 |
| `LEVEL_NAME` | 사용자 레벨명 |
| `TOTAL_USER_COUNT` | 해당 Activity + Level 조합의 사용자 수 |

---

## 추가할 섹션 위치

기존 리포트에서 아래 섹션 뒤에 추가해주세요.

```html
<!-- CHART: 일평균 비교 -->
<section class="section">
  ...
</section>
```

그리고 아래 섹션보다 위에 위치시키면 됩니다.

```html
<!-- INSIGHTS -->
<section class="section">
```

즉, 흐름은 아래처럼 되면 좋습니다.

```text
전체 요약
누적 사용 현황
오픈일 기준 정규화 비교
사용자 레벨 분석 ← 신규 추가
핵심 인사이트
Activity 상세 데이터
Activity 사용자 의견
```

---

## 신규 섹션 HTML 구조

기존 리포트 스타일과 맞추기 위해 아래 구조를 기준으로 추가해주세요.

```html
<!-- LEVEL ANALYSIS: 전체 레벨 사용자 분포 -->
<section class="section" id="level-overview-section">
  <div class="section-head">
    <h2>사용자 레벨 분석</h2>
    <div class="section-divider"></div>
  </div>

  <div class="chart-card">
    <div class="chart-title">
      <span class="dot" style="background:var(--teal)"></span>
      전체 레벨 사용자 분포 (사용자 수 기준)
    </div>

    <div class="chart-legend">
      <div class="legend-item">
        <span class="legend-sq" style="background:#2dd4bf"></span>
        레벨별 전체 사용자 수
      </div>
      <div class="legend-item">단위: 명</div>
    </div>

    <div style="position:relative;height:420px;">
      <canvas id="levelUserOverviewChart" role="img" aria-label="전체 레벨 사용자 분포 차트">전체 레벨 사용자 분포 차트</canvas>
    </div>
  </div>
</section>
```

---

## 데이터 처리 기준

엑셀 데이터에서 `LEVEL_NAME` 기준으로 `TOTAL_USER_COUNT`를 합산해주세요.

### 집계 로직

```text
LEVEL_NAME별 TOTAL_USER_COUNT 합계
```

예:

```js
const levelUserOverviewData = [
  { level: 'ECP5', userCount: 1234 },
  { level: 'ECP6', userCount: 2345 },
  { level: 'ECP7', userCount: 3456 },
  ...
];
```

### 주의사항

- 같은 레벨이 여러 Activity에 반복되어 있으므로 반드시 합산해야 합니다.
- `TOTAL_USER_COUNT`는 숫자로 처리해주세요.
- `LEVEL_NAME` 정렬은 가능하면 기존 교육 레벨 순서를 유지해주세요.
- 알파벳 정렬만 하면 `GT1`, `GT2`, `Honors`, `Leaders`, `LX` 순서가 어색해질 수 있습니다.

---

## 권장 레벨 정렬 순서

엑셀에 있는 전체 레벨명을 확인한 뒤, 가능하면 아래 계열 순서로 정렬해주세요.

```js
const levelOrder = [
  'ECP5', 'ECP6', 'ECP6m', 'ECP7', 'ECP7i', 'ECP7m',
  'GTi', 'GT1', 'GT2', 'GT3', 'GT4',
  'Honors 1', 'Honors 2', 'Honors 3',
  'Leaders 1', 'Leaders 2', 'Leaders 3',
  'LX A', 'LX B', 'LX C', 'LX D', 'LX E', 'LX F',
  'MX A', 'MX B', 'MX C', 'MX D', 'MX E', 'MX F',
  'NCR A', 'NCR B', 'NCR C', 'NCR D',
  'READY'
];
```

엑셀에 있는 레벨 중 위 목록에 없는 레벨이 있으면, 기존 목록 뒤에 추가해주세요.

---

## Chart.js 구현 요청

기존 파일의 다른 차트 생성 코드가 있는 `<script>` 영역에 아래와 같은 형태로 추가해주세요.

### 구현 방향

- Chart type: `bar`
- X축: 레벨명
- Y축: 사용자 수
- Dataset: 전체 사용자 수
- 막대 색상: 기존 테마와 어울리게 `#2dd4bf` 또는 레벨별 컬러 적용
- y축 숫자는 `toLocaleString()` 적용
- tooltip도 `명` 단위 표시
- legend는 숨겨도 됩니다. 이미 chart-card 상단에 legend가 있으므로 `plugins.legend.display = false` 권장

### 예시 코드

```js
function renderLevelUserOverviewChart() {
  const canvas = document.getElementById('levelUserOverviewChart');
  if (!canvas) return;

  const labels = levelUserOverviewData.map(item => item.level);
  const values = levelUserOverviewData.map(item => item.userCount);

  new Chart(canvas, {
    type: 'bar',
    data: {
      labels,
      datasets: [{
        label: '사용자 수',
        data: values,
        backgroundColor: '#2dd4bf',
        borderColor: '#2dd4bf',
        borderWidth: 1,
        borderRadius: 6,
        maxBarThickness: 34
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: { display: false },
        tooltip: {
          callbacks: {
            label: function(context) {
              return `사용자 수: ${Number(context.raw).toLocaleString()}명`;
            }
          }
        }
      },
      scales: {
        x: {
          ticks: {
            color: '#8b90a0',
            maxRotation: 45,
            minRotation: 0
          },
          grid: { color: 'rgba(255,255,255,0.04)' }
        },
        y: {
          beginAtZero: true,
          ticks: {
            color: '#8b90a0',
            callback: function(value) {
              return Number(value).toLocaleString();
            }
          },
          grid: { color: 'rgba(255,255,255,0.07)' }
        }
      }
    }
  });
}
```

---

## 데이터 삽입 방식

이번 리포트는 1회성 보고용 파일이므로, 서버 연동이나 외부 파일 fetch는 하지 않아도 됩니다.

가장 안전한 방식은 아래입니다.

1. 엑셀 데이터를 Cursor 또는 별도 스크립트로 집계
2. 집계 결과를 `index.html` 내부에 JS 배열로 하드코딩
3. 페이지 로드 시 `renderLevelUserOverviewChart()` 실행

예:

```js
const levelUserOverviewData = [
  { level: 'ECP5', userCount: 0 },
  { level: 'ECP6', userCount: 0 },
  { level: 'ECP6m', userCount: 0 },
  { level: 'ECP7', userCount: 0 }
];
```

위 예시는 형식 예시입니다. 실제 값은 엑셀 집계 결과로 교체해주세요.

---

## 기존 차트 초기화 흐름에 연결

기존 파일 하단에 `init`, `DOMContentLoaded`, 또는 직접 차트를 렌더링하는 코드가 있을 가능성이 큽니다.

기존 방식에 맞춰 아래 함수를 호출해주세요.

```js
renderLevelUserOverviewChart();
```

예:

```js
document.addEventListener('DOMContentLoaded', () => {
  renderTotalUsageChart();
  renderDailyUsageChart();
  renderDailyUsersChart();
  renderDailyReviewChart();
  renderBubbleChart();

  // 신규 추가
  renderLevelUserOverviewChart();
});
```

만약 기존 코드가 `DOMContentLoaded`를 사용하지 않고 바로 실행하는 구조라면, 기존 방식에 맞춰 자연스럽게 연결해주세요.

---

## 디자인 가이드

기존 리포트와 톤을 맞춰주세요.

- 배경: 기존 `.chart-card` 사용
- 글자색: 기존 `--text2`, `--text3` 사용
- 강조색: `--teal` 또는 `--accent2`
- Chart.js grid color는 기존 차트와 유사하게 `rgba(255,255,255,0.07)` 사용
- 레벨명이 많아 X축이 복잡하면 `maxRotation: 45` 적용
- 차트 높이는 420px 권장

---

## 완료 기준

작업 완료 후 아래를 확인해주세요.

- [ ] `index.html`에 “사용자 레벨 분석” 섹션이 추가되었는가?
- [ ] “전체 레벨 사용자 분포 (사용자 수 기준)” 막대그래프가 표시되는가?
- [ ] 엑셀의 `LEVEL_NAME` 기준으로 `TOTAL_USER_COUNT`가 정상 합산되었는가?
- [ ] y축/tooltip 숫자에 천 단위 콤마가 적용되었는가?
- [ ] 기존 차트와 디자인 톤이 자연스럽게 맞는가?
- [ ] 기존 차트, 댓글 탭, 검색/필터 기능이 깨지지 않는가?

---

## 이번 작업에서 하지 말 것

아래 작업은 다음 단계에서 진행할 예정이므로 이번 요청에서는 제외해주세요.

- Activity별 레벨 구성 비율 그래프 추가
- 레벨별 TOP 콘텐츠 그래프 추가
- 레벨별 인사이트 문구 자동 생성
- 엑셀 업로드 기능 구현
- 서버 API 연동
- 기존 리포트 전체 구조 리팩토링
