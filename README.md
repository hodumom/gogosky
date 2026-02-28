# gogosky
<!-- index.html  (이 파일 하나만으로 동작합니다) -->
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Admission Dashboard (MVP)</title>

  <!-- Chart.js (CDN) -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>

  <style>
    :root{
      --bg:#0b1220; --card:#121a2a; --text:#e8eefc; --muted:#9fb0d0; --line:#23304a;
      --accent:#7aa2ff; --danger:#ff6b6b; --ok:#2dd4bf;
    }
    *{box-sizing:border-box}
    body{margin:0; font-family:system-ui,-apple-system,Segoe UI,Roboto,Apple SD Gothic Neo,Noto Sans KR,sans-serif; background:var(--bg); color:var(--text)}
    .wrap{max-width:1100px; margin:28px auto; padding:0 16px}
    header{display:flex; gap:12px; align-items:flex-end; justify-content:space-between; margin-bottom:14px}
    h1{margin:0; font-size:20px; letter-spacing:.2px}
    .sub{color:var(--muted); font-size:12px}
    .grid{display:grid; grid-template-columns: 1.2fr .8fr; gap:14px}
    .card{background:var(--card); border:1px solid var(--line); border-radius:14px; padding:14px}
    .card h2{margin:0 0 10px; font-size:14px; color:var(--muted); font-weight:600}
    .controls{display:grid; grid-template-columns: 1fr 1fr; gap:10px}
    .field label{display:block; font-size:12px; color:var(--muted); margin:0 0 6px}
    select,input{width:100%; padding:10px 10px; border-radius:10px; border:1px solid var(--line); background:#0e1626; color:var(--text)}
    input::placeholder{color:#60759d}
    .kpi{display:grid; grid-template-columns:1fr 1fr; gap:10px; margin-top:10px}
    .kpi .box{padding:12px; border:1px solid var(--line); border-radius:12px; background:#0e1626}
    .kpi .t{font-size:12px; color:var(--muted)}
    .kpi .v{font-size:18px; margin-top:6px}
    .pill{display:inline-flex; align-items:center; gap:8px; padding:8px 10px; border-radius:999px; border:1px solid var(--line); background:#0e1626; color:var(--muted); font-size:12px}
    .pill b{color:var(--text); font-weight:700}
    .row{display:flex; gap:10px; flex-wrap:wrap; margin-top:10px}
    .table{width:100%; border-collapse:collapse; overflow:hidden; border-radius:12px; border:1px solid var(--line)}
    .table th,.table td{padding:10px; border-bottom:1px solid var(--line); font-size:12px}
    .table th{color:var(--muted); font-weight:600; text-align:left; background:#0e1626}
    .table tr:last-child td{border-bottom:none}
    .badge{display:inline-block; padding:3px 8px; border-radius:999px; font-size:11px; border:1px solid var(--line); background:#0e1626; color:var(--muted)}
    .badge.ok{color:var(--ok); border-color:#1f4a44}
    .badge.no{color:var(--danger); border-color:#4a1f1f}
    .hint{color:var(--muted); font-size:12px; line-height:1.5}
    .footer{margin-top:14px; color:var(--muted); font-size:12px}
    @media (max-width: 900px){ .grid{grid-template-columns:1fr} }
  </style>
</head>

<body>
  <div class="wrap">
    <header>
      <div>
        <h1>입시 대시보드 (MVP)</h1>
        <div class="sub">사용자 입력 → 전형/연도 필터 → 컷 추이 그래프 + 합격 가능성 신호</div>
      </div>
      <div class="pill">배포: <b>GitHub Pages</b> 가능</div>
    </header>

    <div class="grid">
      <!-- LEFT -->
      <section class="card">
        <h2>그래프: 연도별 컷 추이</h2>
        <canvas id="cutChart" height="120"></canvas>
        <div class="row" id="legendRow"></div>
      </section>

      <!-- RIGHT -->
      <aside class="card">
        <h2>입력 및 필터</h2>

        <div class="controls">
          <div class="field">
            <label>대학</label>
            <select id="uniSelect"></select>
          </div>
          <div class="field">
            <label>모집단위</label>
            <select id="majorSelect"></select>
          </div>

          <div class="field">
            <label>전형</label>
            <select id="typeSelect"></select>
          </div>
          <div class="field">
            <label>내 등급(예: 2.7)</label>
            <input id="userGrade" type="number" step="0.1" min="1" max="9" placeholder="2.7" />
          </div>
        </div>

        <div class="kpi">
          <div class="box">
            <div class="t">선택 전형의 기준 컷(최근년도)</div>
            <div class="v" id="latestCut">-</div>
          </div>
          <div class="box">
            <div class="t">합격 가능성 신호(단순 룰)</div>
            <div class="v" id="signal">-</div>
          </div>
        </div>

        <div class="row">
          <span class="badge ok">안정</span>
          <span class="badge">적정</span>
          <span class="badge no">도전</span>
        </div>

        <div class="footer hint">
          이 MVP는 “컷과 사용자 등급 차이”로만 신호를 냅니다. 실제 컨설팅용으로 확장할 때는
          전형별 반영비율, 권장과목 충족, 실질 경쟁률, 충원율, 표준점수/백분위 등 변수를 추가하면 됩니다.
        </div>
      </aside>
    </div>

    <section class="card" style="margin-top:14px">
      <h2>표: 연도별 데이터</h2>
      <table class="table" id="dataTable">
        <thead>
          <tr>
            <th>연도</th>
            <th>대학</th>
            <th>모집단위</th>
            <th>전형</th>
            <th>컷(예: 50%)</th>
            <th>경쟁률</th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
      <div class="footer hint">
        데이터는 아래 JS의 admissionData 배열을 수정해서 채워 넣으면 됩니다. CSV를 JSON으로 변환해 붙여 넣어도 됩니다.
      </div>
    </section>
  </div>

  <script>
    /**
     * ---------------------------
     * 1) 샘플 데이터 (여기만 바꾸면 됨)
     * ---------------------------
     * cut50: “50% 컷” 같은 대표값(숫자 작을수록 유리한 등급/점수 체계 가정)
     * rate : 경쟁률(표시용)
     */
    const admissionData = [
      { year: 2023, uni: "A대", major: "컴퓨터공학", type: "학생부종합", cut50: 2.9, rate: 8.2 },
      { year: 2024, uni: "A대", major: "컴퓨터공학", type: "학생부종합", cut50: 2.7, rate: 9.1 },
      { year: 2025, uni: "A대", major: "컴퓨터공학", type: "학생부종합", cut50: 2.6, rate: 10.4 },

      { year: 2023, uni: "A대", major: "경영학", type: "학생부교과", cut50: 2.3, rate: 12.8 },
      { year: 2024, uni: "A대", major: "경영학", type: "학생부교과", cut50: 2.2, rate: 13.4 },
      { year: 2025, uni: "A대", major: "경영학", type: "학생부교과", cut50: 2.1, rate: 14.0 },

      { year: 2023, uni: "B대", major: "생명과학", type: "학생부종합", cut50: 3.1, rate: 6.9 },
      { year: 2024, uni: "B대", major: "생명과학", type: "학생부종합", cut50: 3.0, rate: 7.5 },
      { year: 2025, uni: "B대", major: "생명과학", type: "학생부종합", cut50: 2.9, rate: 8.0 },
    ];

    /**
     * ---------------------------
     * 2) 유틸
     * ---------------------------
     */
    const uniq = (arr) => [...new Set(arr)];
    const by = (key) => (a, b) => (a[key] > b[key] ? 1 : -1);

    function formatNum(n, digits = 1){
      if (n === null || n === undefined || Number.isNaN(n)) return "-";
      return Number(n).toFixed(digits);
    }

    /**
     * 합격 가능성 신호(초간단 룰)
     * - 내 등급이 컷보다 충분히 좋으면(작으면) 안정
     * - 근접하면 적정
     * - 나쁘면 도전
     */
    function calcSignal(userGrade, cut){
      if (!Number.isFinite(userGrade) || !Number.isFinite(cut)) return { label: "-", cls: "" };
      const diff = userGrade - cut; // 양수면 내 등급이 더 나쁨(도전)
      if (diff <= -0.3) return { label: "안정", cls: "ok" };
      if (diff <=  0.2) return { label: "적정", cls: "" };
      return { label: "도전", cls: "no" };
    }

    /**
     * ---------------------------
     * 3) DOM 연결
     * ---------------------------
     */
    const uniSelect   = document.getElementById("uniSelect");
    const majorSelect = document.getElementById("majorSelect");
    const typeSelect  = document.getElementById("typeSelect");
    const userGradeEl = document.getElementById("userGrade");

    const latestCutEl = document.getElementById("latestCut");
    const signalEl    = document.getElementById("signal");
    const tableBody   = document.querySelector("#dataTable tbody");
    const legendRow   = document.getElementById("legendRow");

    /**
     * ---------------------------
     * 4) 옵션 채우기
     * ---------------------------
     */
    function fillSelect(select, options){
      select.innerHTML = "";
      for (const opt of options){
        const o = document.createElement("option");
        o.value = opt;
        o.textContent = opt;
        select.appendChild(o);
      }
    }

    function refreshFilters(){
      const unis = uniq(admissionData.map(d => d.uni)).sort();
      fillSelect(uniSelect, unis);

      refreshMajorAndType();
    }

    function refreshMajorAndType(){
      const uni = uniSelect.value;
      const majors = uniq(admissionData.filter(d => d.uni === uni).map(d => d.major)).sort();
      fillSelect(majorSelect, majors);

      const major = majorSelect.value;
      const types = uniq(admissionData.filter(d => d.uni === uni && d.major === major).map(d => d.type)).sort();
      fillSelect(typeSelect, types);

      renderAll();
    }

    /**
     * ---------------------------
     * 5) 표 렌더
     * ---------------------------
     */
    function renderTable(filtered){
      tableBody.innerHTML = "";
      const rows = [...filtered].sort(by("year"));

      for (const r of rows){
        const tr = document.createElement("tr");
        tr.innerHTML = `
          <td>${r.year}</td>
          <td>${r.uni}</td>
          <td>${r.major}</td>
          <td>${r.type}</td>
          <td>${formatNum(r.cut50, 1)}</td>
          <td>${formatNum(r.rate, 1)}</td>
        `;
        tableBody.appendChild(tr);
      }
    }

    /**
     * ---------------------------
     * 6) 차트
     * ---------------------------
     */
    let chart;

    function renderChart(filtered){
      const rows = [...filtered].sort(by("year"));
      const labels = rows.map(r => r.year);
      const data = rows.map(r => r.cut50);

      const ctx = document.getElementById("cutChart");
      if (chart) chart.destroy();

      chart = new Chart(ctx, {
        type: "line",
        data: {
          labels,
          datasets: [{
            label: "컷(예: 50%)",
            data,
            tension: 0.25,
            pointRadius: 4
          }]
        },
        options: {
          responsive: true,
          plugins: {
            legend: { display: false },
            tooltip: { callbacks: { label: (c) => `컷: ${formatNum(c.parsed.y, 1)}` } }
          },
          scales: {
            y: {
              reverse: true, // 등급은 낮을수록 좋다는 가정
              ticks: { callback: (v) => v },
              grid: { color: "rgba(255,255,255,0.06)" }
            },
            x: { grid: { color: "rgba(255,255,255,0.06)" } }
          }
        }
      });

      // 범례 느낌의 정보
      legendRow.innerHTML = "";
      const pill = document.createElement("span");
      pill.className = "pill";
      const latest = rows[rows.length - 1];
      pill.innerHTML = `선택: <b>${latest.uni}</b> / <b>${latest.major}</b> / <b>${latest.type}</b>`;
      legendRow.appendChild(pill);
    }

    /**
     * ---------------------------
     * 7) KPI 렌더
     * ---------------------------
     */
    function renderKpi(filtered){
      const rows = [...filtered].sort(by("year"));
      const latest = rows[rows.length - 1];

      if (!latest){
        latestCutEl.textContent = "-";
        signalEl.textContent = "-";
        signalEl.className = "v";
        return;
      }

      const cut = latest.cut50;
      latestCutEl.textContent = formatNum(cut, 1);

      const userGrade = Number(userGradeEl.value);
      const s = calcSignal(userGrade, cut);

      signalEl.textContent = s.label;
      // 색감은 badge와 맞추기 위해 class만 부여
      signalEl.style.color = (s.cls === "ok") ? "var(--ok)" : (s.cls === "no") ? "var(--danger)" : "var(--text)";
    }

    /**
     * ---------------------------
     * 8) 전체 렌더
     * ---------------------------
     */
    function getFiltered(){
      const uni = uniSelect.value;
      const major = majorSelect.value;
      const type = typeSelect.value;

      return admissionData.filter(d =>
        d.uni === uni && d.major === major && d.type === type
      );
    }

    function renderAll(){
      const filtered = getFiltered();
      renderTable(filtered);
      renderChart(filtered);
      renderKpi(filtered);
    }

    /**
     * ---------------------------
     * 9) 이벤트 바인딩
     * ---------------------------
     */
    uniSelect.addEventListener("change", refreshMajorAndType);
    majorSelect.addEventListener("change", refreshMajorAndType);
    typeSelect.addEventListener("change", renderAll);
    userGradeEl.addEventListener("input", renderAll);

    // 시작
    refreshFilters();
  </script>
</body>
</html>
