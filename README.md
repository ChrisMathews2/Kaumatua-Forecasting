# Kaumatua-Forecasting
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Kaumatua Energy – Yield Valuation Dashboard</title>
  <style>
    :root{--bg:#0b1220;--panel:#0f172a;--muted:#94a3b8;--accent:#38bdf8;--good:#22c55e;--warn:#f59e0b;--bad:#ef4444;--border:#1e293b}
    html,body{height:100%}
    body{margin:0;background:var(--bg);color:#e5e7eb;font-family:system-ui,-apple-system,Segoe UI,Roboto,Inter,Ubuntu,Cantarell,'Helvetica Neue','Noto Sans',sans-serif}
    .wrap{max-width:1200px;margin:0 auto;padding:28px}
    h1{font-size:28px;margin:0 0 12px}
    p.lead{color:var(--muted);margin:0 0 20px}
    .grid{display:grid;gap:16px}
    .grid.cols-3{grid-template-columns:repeat(3,minmax(0,1fr))}
    .grid.cols-2{grid-template-columns:repeat(2,minmax(0,1fr))}
    .card{background:var(--panel);border:1px solid var(--border);border-radius:16px;padding:16px}
    .card h2{font-size:16px;margin:0 0 10px;color:#fff}
    .row{display:grid;grid-template-columns:220px 1fr;align-items:center;gap:10px;margin:8px 0}
    .row label{color:#cbd5e1}
    .row input,.row select,.row textarea{width:100%;padding:10px 12px;border-radius:10px;border:1px solid #334155;background:#0b1326;color:#e5e7eb}
    .hint{font-size:12px;color:var(--muted)}
    .pill{display:inline-block;padding:6px 10px;border-radius:999px;background:#0b1326;border:1px solid #334155;color:#e5e7eb;font-size:12px}
    .kpi{display:flex;flex-direction:column;gap:6px}
    .kpi .label{font-size:12px;color:var(--muted)}
    .kpi .value{font:600 22px/1.1 ui-sans-serif,system-ui,sans-serif}
    .kpi .value.good{color:var(--good)}
    .kpi .value.warn{color:var(--warn)}
    .kpi .value.bad{color:var(--bad)}
    table{width:100%;border-collapse:collapse;border:1px solid var(--border)}
    th,td{border:1px solid var(--border);padding:8px 10px;text-align:right}
    th{background:#0b1326;color:#cbd5e1}
    td:first-child,th:first-child{text-align:left}
    .actions{display:flex;flex-wrap:wrap;gap:10px}
    .btn{padding:10px 12px;border-radius:10px;border:1px solid #334155;background:#0b1326;color:#e5e7eb;cursor:pointer}
    .btn.primary{background:linear-gradient(180deg,#0284c7,#0369a1);border-color:#075985}
    .note{font-size:12px;color:var(--muted);margin-top:8px}
    .footer{margin-top:18px;color:#9ca3af;font-size:12px}
    @media (max-width:920px){.grid.cols-3{grid-template-columns:1fr}.row{grid-template-columns:1fr}}
  </style>
</head>
<body>
  <div class="wrap">
    <h1>Kaumatua Energy – Yield Valuation Dashboard</h1>
    <p class="lead">Enter a three‑year forecast for revenue and net profit, choose yield and growth assumptions, and generate capital value estimates with sensitivity tables. All inputs persist locally in your browser.</p>

    <div class="grid cols-3">
      <section class="card" id="forecasts">
        <h2>1) Three‑Year Forecast</h2>
        <div class="row"><label>Revenue Year 1</label><input id="rev1" type="number" step="0.01" placeholder="e.g. 12,000,000" /></div>
        <div class="row"><label>Revenue Year 2</label><input id="rev2" type="number" step="0.01" /></div>
        <div class="row"><label>Revenue Year 3</label><input id="rev3" type="number" step="0.01" /></div>
        <div class="row"><label>Net Profit (after tax) Year 1</label><input id="np1" type="number" step="0.01" /></div>
        <div class="row"><label>Net Profit (after tax) Year 2</label><input id="np2" type="number" step="0.01" /></div>
        <div class="row"><label>Net Profit (after tax) Year 3</label><input id="np3" type="number" step="0.01" /></div>
        <div class="row"><label>Payout Ratio (dividends as % of NP)</label><input id="payout" type="number" step="0.01" value="50" /><span class="hint">%</span></div>
        <div class="row"><label>Net Debt (–) / Net Cash (+)</label><input id="netDebt" type="number" step="0.01" value="0" /></div>
        <div class="row"><label>Shares Outstanding</label><input id="shares" type="number" step="1" value="1000000" /></div>
        <div class="note">Tip: If you pay dividends, the Dividend Discount Model uses Year 3 as the base year.</div>
      </section>

      <section class="card" id="assumptions">
        <h2>2) Assumptions</h2>
        <div class="row"><label>Required Equity Yield (r)</label><input id="yieldReq" type="number" step="0.01" value="10" /><span class="hint">%</span></div>
        <div class="row"><label>Long‑term Growth (g)</label><input id="growthLT" type="number" step="0.01" value="2" /><span class="hint">%</span></div>
        <div class="row"><label>Valuation Base</label>
          <select id="baseMode">
            <option value="y3">Use Year 3 values</option>
            <option value="avg">Use 3‑year average</option>
          </select>
        </div>
        <div class="row"><label>Revenue Multiple (optional)</label><input id="revMultiple" type="number" step="0.1" value="" placeholder="e.g. 2.5×" /></div>
        <div class="note">r must be greater than g for the dividend model. Revenue multiple estimates Enterprise Value; equity value adjusts for net debt/cash.</div>
      </section>

      <section class="card" id="kpis">
        <h2>3) Snapshot</h2>
        <div class="grid cols-2">
          <div class="kpi"><span class="label">Year 3 Revenue</span><span id="kpiRev3" class="value">–</span></div>
          <div class="kpi"><span class="label">Year 3 Net Profit</span><span id="kpiNP3" class="value">–</span></div>
          <div class="kpi"><span class="label">NP Margin (Y3)</span><span id="kpiMargin" class="value">–</span></div>
          <div class="kpi"><span class="label">Avg NP (3y)</span><span id="kpiNPA" class="value">–</span></div>
        </div>
        <div class="actions" style="margin-top:12px">
          <button class="btn primary" id="recalc">Recalculate</button>
          <button class="btn" id="reset">Reset</button>
          <label class="btn" for="csvIn">Import CSV</label>
          <input id="csvIn" type="file" accept=".csv" style="display:none" />
          <button class="btn" id="exportCSV">Export Sensitivity CSV</button>
        </div>
      </section>
    </div>

    <div class="grid cols-2" style="margin-top:16px">
      <section class="card" id="valuations">
        <h2>Core Valuation Outputs</h2>
        <table>
          <thead>
            <tr><th>Method</th><th>Enterprise Value (EV)</th><th>Equity Value</th><th>Per Share</th></tr>
          </thead>
          <tbody id="valBody"></tbody>
        </table>
        <div class="note">Earnings Yield uses Net Profit as a proxy for equity cash flow (simplified). Dividend Discount uses payout × Net Profit as base dividend (forward). Revenue Multiple computes EV = Multiple × Revenue (Year 3 or 3‑year average as selected).</div>
      </section>

      <section class="card" id="sensitivity">
        <h2>Yield Sensitivity (Equity Value)</h2>
        <div class="row"><label>Yield Range (%)</label>
          <input id="yieldMin" type="number" value="6" step="0.5" />
          <input id="yieldMax" type="number" value="16" step="0.5" />
          <input id="yieldStep" type="number" value="1" step="0.5" />
        </div>
        <div class="row"><label>Growth Scenarios g (%)</label>
          <input id="growthList" type="text" value="0,1,2,3" />
        </div>
        <table style="margin-top:10px">
          <thead id="sensHead"></thead>
          <tbody id="sensBody"></tbody>
        </table>
      </section>
    </div>

    <div class="footer">This is a simplified financial model for exploratory valuation only. Consider taxes, maintenance capex, working capital, and financing effects for formal valuations.</div>
  </div>

<script>
(function(){
  const $ = id => document.getElementById(id);
  const ids = [
    'rev1','rev2','rev3','np1','np2','np3','payout','netDebt','shares',
    'yieldReq','growthLT','baseMode','revMultiple','yieldMin','yieldMax','yieldStep','growthList'
  ];

  // Load / Save state
  function load(){
    const raw = localStorage.getItem('ke_yield_dash');
    if(!raw) return;
    try{
      const data = JSON.parse(raw);
      ids.forEach(k=>{ if(data[k] !== undefined && $(k)) $(k).value = data[k]; });
    }catch(e){ console.warn('State load failed', e); }
  }
  function save(){
    const data = {};
    ids.forEach(k=> data[k] = $(k).value);
    localStorage.setItem('ke_yield_dash', JSON.stringify(data));
  }

  ids.forEach(k => $(k)?.addEventListener('input', ()=>{ save(); }));

  // Helpers
  const toNum = v => {
    const n = parseFloat(String(v).replace(/[,\s]/g,''));
    return isFinite(n) ? n : 0;
  };
  const fmt = n => isFinite(n) ? n.toLocaleString(undefined,{maximumFractionDigits:0}) : '–';
  const fmt2 = n => isFinite(n) ? n.toLocaleString(undefined,{maximumFractionDigits:2}) : '–';

  function baseValues(mode){
    const R = [toNum($('#rev1').value), toNum($('#rev2').value), toNum($('#rev3').value)];
    const N = [toNum($('#np1').value),  toNum($('#np2').value),  toNum($('#np3').value)];
    if(mode==='avg'){
      return { rev: (R[0]+R[1]+R[2])/3, np: (N[0]+N[1]+N[2])/3 };
    }
    return { rev: R[2], np: N[2] };
  }

  function calc(){
    const mode = $('#baseMode').value;
    const base = baseValues(mode);
    const payout = toNum($('#payout').value)/100; // fraction
    const netDebt = toNum($('#netDebt').value);
    const shares = Math.max(1, toNum($('#shares').value));
    const r = toNum($('#yieldReq').value)/100;
    const g = toNum($('#growthLT').value)/100;
    const revMultiple = toNum($('#revMultiple').value || '0');

    // KPIs
    $('#kpiRev3').textContent = '$ '+fmt(toNum($('#rev3').value));
    $('#kpiNP3').textContent  = '$ '+fmt(toNum($('#np3').value));
    const margin = (toNum($('#np3').value) && toNum($('#rev3').value)) ? (toNum($('#np3').value)/toNum($('#rev3').value))*100 : 0;
    $('#kpiMargin').textContent = fmt2(margin)+'%';
    const avgNP = (toNum($('#np1').value)+toNum($('#np2').value)+toNum($('#np3').value))/3;
    $('#kpiNPA').textContent = '$ '+fmt(avgNP);

    // Valuation methods
    const rows = [];

    // 1) Earnings Yield (Equity Value ≈ NP / r)
    const eqEY = r>0 ? base.np / r : NaN;
    rows.push(['Earnings Yield (NP / r)', NaN, eqEY, eqEY/shares]);

    // 2) Dividend Discount (Gordon) using payout×NP as next dividend base
    let eqDDM = NaN;
    if(r>g && payout>0){
      const d1 = base.np * payout * (1+g);
      eqDDM = d1 / (r - g);
    }
    rows.push(['Dividend Discount (Gordon)', NaN, eqDDM, eqDDM/shares]);

    // 3) Revenue multiple → EV = multiple × Revenue; Equity = EV − NetDebt
    if(revMultiple>0){
      const ev = revMultiple * base.rev;
      const eq = ev - netDebt;
      rows.push([`Revenue Multiple (${revMultiple.toFixed(2)}×)`, ev, eq, eq/shares]);
    }

    // Render table
    const tbody = document.querySelector('#valBody');
    tbody.innerHTML = rows.map(rw=>{
      const [name,ev,eq,ps] = rw;
      return `<tr><td>${name}</td><td>${isFinite(ev)?'$ '+fmt(ev):'–'}</td><td>${isFinite(eq)?'$ '+fmt(eq):'–'}</td><td>${isFinite(ps)?'$ '+fmt2(ps):'–'}</td></tr>`;
    }).join('');

    // Sensitivity table (Equity Value) across yield range and growth list (DDM only where valid). Also show Earnings Yield.
    const yMin = toNum($('#yieldMin').value)/100;
    const yMax = toNum($('#yieldMax').value)/100;
    const yStep= Math.max(0.001, toNum($('#yieldStep').value)/100);
    const gList = String($('#growthList').value).split(',').map(s=>toNum(s)/100).filter(x=>isFinite(x));

    // Header: Yield → columns; rows → methods × growth
    let yields = [];
    for(let y=yMin; y<=yMax+1e-9; y+=yStep){ yields.push(+y.toFixed(6)); }

    const headHtml = `<tr><th>Scenario</th>${yields.map(y=>`<th>${(y*100).toFixed(2)}%</th>`).join('')}</tr>`;
    $('#sensHead').innerHTML = headHtml;

    const bodyRows = [];
    // Earnings Yield row (no growth parameter)
    const rowEY = [`Earnings Yield (NP/r) – ${mode==='y3'?'Y3':'Avg'}`];
    yields.forEach(y=>{
      const val = y>0 ? base.np / y : NaN;
      rowEY.push(isFinite(val)?'$ '+fmt(val):'–');
    });
    bodyRows.push('<tr>'+rowEY.map((c,i)=>`<${i? 'td':'td'}>${c}</td>`).join('')+'</tr>');

    // DDM rows for each g
    gList.forEach(gx=>{
      const label = `DDM (payout ${Math.round(payout*100)}%, g ${ (gx*100).toFixed(1)}%)`;
      const row = [label];
      yields.forEach(y=>{
        if(y>gx && payout>0){
          const d1 = base.np * payout * (1+gx);
          const val = d1 / (y - gx);
          row.push('$ '+fmt(val));
        } else {
          row.push('n/a');
        }
      });
      bodyRows.push('<tr>'+row.map((c,i)=>`<td>${c}</td>`).join('')+'</tr>');
    });

    // Revenue multiple row if set
    if(revMultiple>0){
      const label = `Revenue Multiple (${revMultiple.toFixed(2)}×) – Equity`;
      const eq = (revMultiple * base.rev) - netDebt;
      const row = [label, `<td colspan="${yields.length}">$ ${fmt(eq)}</td>`];
      bodyRows.push('<tr><td>'+label+'</td>'+`<td colspan="${yields.length}">$ ${fmt(eq)}</td>`+'</tr>');
    }

    $('#sensBody').innerHTML = bodyRows.join('');
    save();
  }

  // CSV import (headers: rev1,rev2,rev3,np1,np2,np3,payout,netDebt,shares,yieldReq,growthLT,revMultiple)
  $('#csvIn').addEventListener('change', async (e)=>{
    const file = e.target.files?.[0];
    if(!file) return;
    const text = await file.text();
    const [head,...rows] = text.trim().split(/\r?\n/);
    const headers = head.split(',').map(h=>h.trim());
    const first = rows[0]?.split(',') || [];
    headers.forEach((h,i)=>{ if($(h)) $(h).value = first[i] ?? $(h).value; });
    save();
    calc();
  });

  // Export sensitivity as CSV
  function exportSensitivity(){
    // Recompute to ensure latest DOM
    const mode = $('#baseMode').value;
    const base = baseValues(mode);
    const payout = toNum($('#payout').value)/100; 
    const yMin = toNum($('#yieldMin').value)/100;
    const yMax = toNum($('#yieldMax').value)/100;
    const yStep= Math.max(0.001, toNum($('#yieldStep').value)/100);
    const gList = String($('#growthList').value).split(',').map(s=>toNum(s)/100).filter(x=>isFinite(x));

    let yields = [];
    for(let y=yMin; y<=yMax+1e-9; y+=yStep){ yields.push(+y.toFixed(6)); }

    let lines = [];
    lines.push(['Scenario', ...yields.map(y=>(y*100).toFixed(2)+'%')].join(','));

    // Earnings Yield
    lines.push(['Earnings Yield'].concat(yields.map(y=> (y>0 ? Math.round(base.np / y) : '') )).join(','));

    // DDM per growth
    gList.forEach(gx=>{
      const row = [`DDM g ${(gx*100).toFixed(1)}%`];
      yields.forEach(y=>{
        if(y>gx && payout>0){
          const d1 = base.np * payout * (1+gx);
          row.push(Math.round(d1 / (y - gx)));
        } else row.push('');
      });
      lines.push(row.join(','));
    });

    const blob = new Blob([lines.join('\n')],{type:'text/csv'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url; a.download = 'kaumatua_yield_sensitivity.csv';
    document.body.appendChild(a); a.click(); a.remove();
    URL.revokeObjectURL(url);
  }

  // Reset (keeps shares default)
  function reset(){
    ['rev1','rev2','rev3','np1','np2','np3','payout','netDebt','yieldReq','growthLT','revMultiple','yieldMin','yieldMax','yieldStep','growthList'].forEach(id=>{
      const el = $('#'+id); if(!el) return;
      switch(id){
        case 'payout': el.value = 50; break;
        case 'netDebt': el.value = 0; break;
        case 'yieldReq': el.value = 10; break;
        case 'growthLT': el.value = 2; break;
        case 'yieldMin': el.value = 6; break;
        case 'yieldMax': el.value = 16; break;
        case 'yieldStep': el.value = 1; break;
        case 'growthList': el.value = '0,1,2,3'; break;
        default: el.value = '';
      }
    });
    $('#baseMode').value = 'y3';
    save();
    calc();
  }

  $('#recalc').addEventListener('click', calc);
  $('#reset').addEventListener('click', reset);
  $('#exportCSV').addEventListener('click', exportSensitivity);

  // Boot
  load();
  calc();
})();
</script>
</body>
</html>
