<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Auto Budgeter</title>
<style>
 :root{
 --bg:#0e0f13; --panel:#151821; --sub:#1f2330; --text:#eef2ff; --muted:#a7b0c0;
 --accent:#7c9cff; --good:#47d16a; --warn:#ffb84d; --bad:#ff6473; --border:#2b3142;
 --chip:#2a3042;
 }
 *{box-sizing:border-box}
 body{
 margin:0; font-family:ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto,Inter,Arial;
 color:var(--text); background:linear-gradient(180deg,#0b0c10 0%, #0e0f13 100%);
 min-height:100svh;
 }
 header{
 padding:20px clamp(16px,4vw,32px);
 display:flex; align-items:center; justify-content:space-between; gap:16px;
 border-bottom:1px solid var(--border); background:rgba(21,24,33,0.7); backdrop-filter: blur(10px);
 position:sticky; top:0; z-index:10;
 }
 .brand{display:flex; align-items:center; gap:12px; font-weight:700; letter-spacing:.3px}
 .logo{
 width:34px; height:34px; border-radius:8px; background:
 conic-gradient(from 220deg,var(--accent),#84ffd6,#ffe087,var(--accent));
 filter:saturate(1.1) hue-rotate(2deg); border:1px solid #ffffff20;
 box-shadow: 0 0 0 3px #00000025 inset;
 }
 .wrap{
 padding:24px clamp(16px,4vw,32px); max-width:1100px; margin-inline:auto; display:grid;
 grid-template-columns: 1.1fr 1fr; gap:24px;
 }
 @media (max-width: 900px){ .wrap{grid-template-columns:1fr; } }

 .card{
 background:var(--panel); border:1px solid var(--border); border-radius:14px; padding:18px;
 box-shadow: 0 10px 30px #00000040;
 }
 h2{margin:8px 0 16px; font-size:1.1rem}
 h3{margin:10px 0 10px; font-size:1rem; color:var(--muted)}
 .row{display:flex; gap:12px; flex-wrap:wrap}
 .row > *{flex:1}
 label{font-size:.85rem; color:var(--muted); display:block; margin:4px 0 6px}
 input, select, button, textarea{
 width:100%; padding:10px 12px; border-radius:10px; border:1px solid var(--border);
 background:var(--sub); color:var(--text); outline:none; font:inherit;
 }
 input[type="number"]{appearance:textfield}
 input:focus, select:focus, textarea:focus{border-color:var(--accent); box-shadow:0 0 0 3px #7c9cff22}
 button{
 cursor:pointer; border:1px solid #ffffff22; background:linear-gradient(180deg,#2f3650,#262c42);
 }
 button.primary{background:linear-gradient(180deg,#5c74ff,#556bff); border-color:#8ea0ff}
 button.ghost{background:transparent}
 button.danger{background:linear-gradient(180deg,#ff6b7a,#d94a58); border-color:#ff9aa5}
 .kpi{
 display:grid; grid-template-columns:repeat(4,1fr); gap:12px;
 }
 @media (max-width:700px){ .kpi{grid-template-columns:repeat(2,1fr)} }
 .tile{
 background:var(--sub); border:1px solid var(--border); border-radius:12px; padding:14px;
 }
 .tile .label{color:var(--muted); font-size:.8rem}
 .tile .value{font-weight:700; font-size:1.15rem; margin-top:6px}
 .alloc-grid{
 display:grid; grid-template-columns:1fr 100px 120px; gap:10px; align-items:end;
 }
 .alloc-grid .head{color:var(--muted); font-size:.8rem}
 .bar{
 height:10px; background:#ffffff14; border-radius:999px; overflow:hidden;
 }
 .bar > span{display:block; height:100%}
 .chip{
 display:inline-flex; align-items:center; gap:8px; padding:6px 10px; border-radius:999px;
 background:var(--chip); color:#d7def0; font-size:.82rem; border:1px solid var(--border);
 }
 .list{display:grid; gap:10px; margin-top:10px}
 .item{
 display:grid; grid-template-columns:1fr 110px 110px 40px; gap:10px; align-items:center;
 padding:10px; border:1px solid var(--border); background:var(--sub); border-radius:10px;
 }
 @media (max-width:650px){ .item{grid-template-columns:1fr 90px 90px 40px} }
 .right{display:flex; gap:10px; align-items:center; flex-wrap:wrap}
 .footer{
 margin-top:20px; display:flex; gap:10px; flex-wrap:wrap; align-items:center; justify-content:space-between;
 }
 .hint{color:var(--muted); font-size:.85rem}
 .spacer{height:8px}
 .progress-row{display:grid; grid-template-columns:1fr 90px; gap:10px; align-items:center;}
 .empty{color:var(--muted); font-style:italic; text-align:center; padding:16px 8px}
 .link{color:#b9c6ff; text-decoration:underline; cursor:pointer}
 .danger-inline{color:#ff98a4; border-color:#ff98a4}
</style>
</head>
<body>
 <header>
 <div class="brand">
 <div class="logo" aria-hidden="true"></div>
 <div>Auto Budgeter</div>
 </div>
 <div class="right">
 <span class="chip">Data saved locally</span>
 <button id="exportBtn" class="ghost">Export</button>
 <label class="chip" style="cursor:pointer">
 Import <input id="importInput" type="file" accept="application/json" hidden />
 </label>
 <button id="resetBtn" class="danger">Reset</button>
 </div>
 </header>

 <main class="wrap">
 <!-- LEFT: Setup & Planning -->
 <section class="card" aria-labelledby="setupTitle">
 <h2 id="setupTitle">Setup & Monthly Plan</h2>

 <div class="kpi" role="region" aria-label="Key numbers">
 <div class="tile">
 <div class="label">Monthly Net Income</div>
 <div class="value" id="kIncome">$0</div>
 </div>
 <div class="tile">
 <div class="label">Planned Spending</div>
 <div class="value" id="kPlanned">$0</div>
 </div>
 <div class="tile">
 <div class="label">Remaining (Unallocated)</div>
 <div class="value" id="kLeft">$0</div>
 </div>
 <div class="tile">
 <div class="label">Month</div>
 <div class="value" id="kMonth">—</div>
 </div>
 </div>

 <div class="spacer"></div>

 <div class="row">
 <div>
 <label for="income">Monthly Net Income</label>
 <input id="income" type="number" step="0.01" min="0" placeholder="e.g., 3500" />
 </div>
 <div>
 <label for="month">Budget Month</label>
 <input id="month" type="month" />
 </div>
 </div>

 <h3>Auto Allocation (editable)</h3>
 <div class="alloc-grid" role="table" aria-label="Allocation targets">
 <div class="head">Category</div>
 <div class="head">% Target</div>
 <div class="head">Planned $</div>

 <div>Needs (rent, utilities, groceries)</div>
 <input id="pNeeds" type="number" step="1" min="0" max="100" value="50" />
 <input id="aNeeds" type="number" step="0.01" min="0" />

 <div>Wants (fun, dining, subs)</div>
 <input id="pWants" type="number" step="1" min="0" max="100" value="30" />
 <input id="aWants" type="number" step="0.01" min="0" />

 <div>Savings/Debt</div>
 <input id="pSave" type="number" step="1" min="0" max="100" value="20" />
 <input id="aSave" type="number" step="0.01" min="0" />
 </div>

 <div class="footer">
 <div class="right">
 <button id="autoBtn" class="primary">Auto allocate</button>
 <button id="recalcBtn">Recalculate totals</button>
 </div>
 <div class="hint">Tip: Adjust % or dollar amounts—this app will keep totals in sync.</div>
 </div>

 <div class="spacer"></div>

 <h3>Fixed Bills (Needs)</h3>
 <div class="row">
 <div><label>Name</label><input id="billName" placeholder="Rent, Internet…" /></div>
 <div><label>Amount</label><input id="billAmt" type="number" step="0.01" min="0" placeholder="1200" /></div>
 <div><label>&nbsp;</label><button id="addBillBtn">Add Bill</button></div>
 </div>
 <div id="bills" class="list" aria-live="polite"></div>

 </section>

 <!-- RIGHT: Tracking -->
 <section class="card" aria-labelledby="trackTitle">
 <h2 id="trackTitle">Track Spending</h2>

 <div class="row">
 <div>
 <label>Category</label>
 <select id="txCat">
 <option value="needs">Needs</option>
 <option value="wants">Wants</option>
 <option value="save">Savings/Debt</option>
 </select>
 </div>
 <div>
 <label>Note</label>
 <input id="txNote" placeholder="Groceries @ Market" />
 </div>
 <div>
 <label>Amount</label>
 <input id="txAmt" type="number" step="0.01" min="0" placeholder="54.20" />
 </div>
 <div>
 <label>&nbsp;</label>
 <button id="addTxBtn">Add</button>
 </div>
 </div>

 <div class="spacer"></div>

 <h3>Progress</h3>
 <div class="list" id="progressList">
 <!-- Progress rows injected here -->
 </div>

 <h3 style="margin-top:16px">Transactions</h3>
 <div id="txList" class="list"></div>

 <div class="footer">
 <div class="hint">Export to back up, import to restore or move devices.</div>
 <div class="right">
 <button id="clearTxBtn" class="ghost danger-inline">Clear Transactions</button>
 </div>
 </div>
 </section>
 </main>

<script>
/* ============== Data Layer ============== */
const $ = sel => document.querySelector(sel);
const $$ = sel => Array.from(document.querySelectorAll(sel));

const state = {
 month: '', income: 0,
 targets: { needs: 50, wants: 30, save: 20 },
 planned: { needs: 0, wants: 0, save: 0 },
 bills: [], // {id, name, amt}
 tx: [] // {id, ts, cat, note, amt}
};

const KEY = 'auto-budgeter-v1';

function load() {
 try {
 const raw = localStorage.getItem(KEY);
 if (!raw) return;
 const saved = JSON.parse(raw);
 Object.assign(state, saved);
 } catch(e){ console.warn('Load failed', e) }
}
function persist() {
 localStorage.setItem(KEY, JSON.stringify(state));
}

/* ============== Utils ============== */
const fmt = n => (isNaN(n)? '$0' : n.toLocaleString(undefined,{style:'currency',currency:'USD'}));
const sum = arr => arr.reduce((a,b)=>a+b,0);
const uid = () => Math.random().toString(36).slice(2,9);

/* ============== DOM Bindings ============== */
const kIncome = $('#kIncome'), kPlanned = $('#kPlanned'), kLeft = $('#kLeft'), kMonth = $('#kMonth');
const incomeEl = $('#income'), monthEl = $('#month');
const pNeeds = $('#pNeeds'), pWants = $('#pWants'), pSave = $('#pSave');
const aNeeds = $('#aNeeds'), aWants = $('#aWants'), aSave = $('#aSave');
const autoBtn = $('#autoBtn'), recalcBtn = $('#recalcBtn');
const billName = $('#billName'), billAmt = $('#billAmt'), addBillBtn = $('#addBillBtn'), billsEl = $('#bills');

const txCat = $('#txCat'), txNote = $('#txNote'), txAmt = $('#txAmt'), addTxBtn = $('#addTxBtn');
const progressList = $('#progressList'), txList = $('#txList');
const exportBtn = $('#exportBtn'), importInput = $('#importInput'), resetBtn = $('#resetBtn'), clearTxBtn = $('#clearTxBtn');

/* ============== Rendering ============== */
function renderKPIs(){
 const plannedTotal = state.planned.needs + state.planned.wants + state.planned.save;
 const unalloc = Math.max(0, state.income - plannedTotal);
 kIncome.textContent = fmt(state.income);
 kPlanned.textContent = fmt(plannedTotal);
 kLeft.textContent = fmt(unalloc);
 kMonth.textContent = state.month ? new Date(state.month+'-01').toLocaleString(undefined,{month:'long', year:'numeric'}) : '—';
}

function renderAllocFields(){
 pNeeds.value = state.targets.needs;
 pWants.value = state.targets.wants;
 pSave.value = state.targets.save;
 aNeeds.value = state.planned.needs.toFixed(2);
 aWants.value = state.planned.wants.toFixed(2);
 aSave.value = state.planned.save.toFixed(2);
 incomeEl.value = state.income || '';
 monthEl.value = state.month || '';
}

function renderBills(){
 billsEl.innerHTML = '';
 if (state.bills.length === 0){
 const d = document.createElement('div');
 d.className='empty';
 d.textContent = 'No fixed bills yet.';
 billsEl.appendChild(d);
 return;
 }
 for (const b of state.bills){
 const row = document.createElement('div');
 row.className='item';
 row.innerHTML = `
 <div><strong>${b.name}</strong></div>
 <div>${fmt(b.amt)}</div>
 <div class="hint">Needs</div>
 <button aria-label="Delete bill" data-id="${b.id}">✕</button>
 `;
 row.querySelector('button').addEventListener('click', () => {
 const idx = state.bills.findIndex(x=>x.id===b.id);
 if (idx>-1){ state.bills.splice(idx,1); persist(); syncPlannedMinNeeds(); renderAll(); }
 });
 billsEl.appendChild(row);
 }
}

function categorySpent(cat){
 return sum(state.tx.filter(t=>t.cat===cat).map(t=>t.amt));
}

function progressRow(label, cat, planned){
 const spent = categorySpent(cat);
 const pct = planned>0 ? Math.min(100, (spent/planned)*100) : 0;
 const left = Math.max(0, planned - spent);
 const color = pct<70 ? 'var(--good)' : (pct<100 ? 'var(--warn)' : 'var(--bad)');
 const row = document.createElement('div');
 row.className = 'progress-row';
 row.innerHTML = `
 <div>
 <div style="display:flex; justify-content:space-between; margin-bottom:8px">
 <div><strong>${label}</strong> <span class="hint">(${fmt(spent)} / ${fmt(planned)})</span></div>
 <div class="hint">${fmt(left)} left</div>
 </div>
 <div class="bar" aria-label="${label} progress"><span style="width:${pct}%; background:${color}"></span></div>
 </div>
 <div class="chip">${Math.round(pct)}%</div>
 `;
 return row;
}

function renderProgress(){
 progressList.innerHTML = '';
 progressList.appendChild(progressRow('Needs','needs',state.planned.needs));
 progressList.appendChild(progressRow('Wants','wants',state.planned.wants));
 progressList.appendChild(progressRow('Savings/Debt','save',state.planned.save));
}

function renderTx(){
 txList.innerHTML = '';
 if (state.tx.length===0){
 const d = document.createElement('div');
 d.className='empty'; d.textContent='No transactions yet.';
 txList.appendChild(d); return;
 }
 for (const t of [...state.tx].sort((a,b)=>b.ts-a.ts)){
 const row = document.createElement('div');
 row.className='item';
 const label = t.cat==='needs'?'Needs':(t.cat==='wants'?'Wants':'Savings/Debt');
 row.innerHTML = `
 <div><strong>${label}</strong> <span class="hint">• ${new Date(t.ts).toLocaleString()}</span><br />
 <span class="hint">${t.note || '—'}</span></div>
 <div>${fmt(t.amt)}</div>
 <div></div>
 <button aria-label="Delete transaction" data-id="${t.id}">✕</button>
 `;
 row.querySelector('button').addEventListener('click', () => {
 const idx = state.tx.findIndex(x=>x.id===t.id);
 if (idx>-1){ state.tx.splice(idx,1); persist(); renderAll(); }
 });
 txList.appendChild(row);
 }
}

function renderAll(){
 renderAllocFields();
 renderBills();
 renderKPIs();
 renderProgress();
 renderTx();
}

/* ============== Logic ============== */
function autoAllocate(){
 // Respect fixed bills first: Needs must be at least bills total
 const billsTotal = sum(state.bills.map(b=>b.amt));
 const needsPct = clamp(parseFloat(pNeeds.value)||0, 0, 100);
 const wantsPct = clamp(parseFloat(pWants.value)||0, 0, 100);
 const savePct = clamp(parseFloat(pSave.value)||0, 0, 100);
 const pctSum = needsPct + wantsPct + savePct;
 if (pctSum === 0){
 state.planned.needs = state.planned.wants = state.planned.save = 0;
 } else {
 state.planned.needs = round2(state.income * (needsPct/pctSum));
 state.planned.wants = round2(state.income * (wantsPct/pctSum));
 state.planned.save = round2(state.income * (savePct/pctSum));
 }
 // Ensure needs covers bills
 if (state.planned.needs < billsTotal){
 const diff = billsTotal - state.planned.needs;
 state.planned.needs = billsTotal;
 // Steal from wants then save
 const stealFromWants = Math.min(diff, state.planned.wants);
 state.planned.wants -= stealFromWants;
 const remaining = diff - stealFromWants;
 state.planned.save = Math.max(0, state.planned.save - remaining);
 }
 persist();
}

function syncPlannedMinNeeds(){
 const billsTotal = sum(state.bills.map(b=>b.amt));
 if (state.planned.needs < billsTotal) state.planned.needs = billsTotal;
}

function recalcFromDollarEdits(){
 // When user edits planned $ directly, keep total <= income if possible.
 const n = toNum(aNeeds.value), w = toNum(aWants.value), s = toNum(aSave.value);
 const total = n+w+s;
 if (total > state.income){
 // Scale down proportionally
 const ratio = state.income / (total || 1);
 state.planned.needs = round2(n*ratio);
 state.planned.wants = round2(w*ratio);
 state.planned.save = round2(s*ratio);
 } else {
 state.planned.needs = n; state.planned.wants = w; state.planned.save = s;
 }
 syncPlannedMinNeeds();
 persist();
}

function toNum(v){ const n = parseFloat(v); return isNaN(n)?0:n }
function round2(n){ return Math.round((n + Number.EPSILON) * 100) / 100 }
function clamp(n,min,max){ return Math.max(min, Math.min(max, n)) }

/* ============== Events ============== */
incomeEl.addEventListener('input', () => {
 state.income = toNum(incomeEl.value); persist(); autoAllocate(); renderAll();
});
monthEl.addEventListener('input', () => {
 state.month = monthEl.value; persist(); renderAll();
});

[pNeeds,pWants,pSave].forEach(el => el.addEventListener('input', () => {
 state.targets.needs = toNum(pNeeds.value)||0;
 state.targets.wants = toNum(pWants.value)||0;
 state.targets.save = toNum(pSave.value)||0;
 persist();
}));

[aNeeds,aWants,aSave].forEach(el => el.addEventListener('change', () => {
 recalcFromDollarEdits(); renderAll();
}));

autoBtn.addEventListener('click', () => { autoAllocate(); renderAll(); });
recalcBtn.addEventListener('click', () => { recalcFromDollarEdits(); renderAll(); });

addBillBtn.addEventListener('click', () => {
 const name = billName.value.trim(); const amt = toNum(billAmt.value);
 if (!name || amt<=0) return;
 state.bills.push({id:uid(), name, amt});
 billName.value=''; billAmt.value='';
 persist(); syncPlannedMinNeeds(); renderAll();
});

addTxBtn.addEventListener('click', () => {
 const cat = txCat.value;
 const note = txNote.value.trim();
 const amt = toNum(txAmt.value);
 if (amt<=0) return;
 state.tx.push({id:uid(), ts:Date.now(), cat, note, amt});
 txNote.value=''; txAmt.value='';
 persist(); renderAll();
});

clearTxBtn.addEventListener('click', () => {
 if (!confirm('Clear ALL transactions?')) return;
 state.tx = []; persist(); renderAll();
});

exportBtn.addEventListener('click', () => {
 const blob = new Blob([JSON.stringify(state,null,2)], {type:'application/json'});
 const url = URL.createObjectURL(blob);
 const a = document.createElement('a');
 a.href = url; a.download = `auto-budgeter-${(state.month||'current')}.json`;
 document.body.appendChild(a); a.click(); a.remove(); URL.revokeObjectURL(url);
});

importInput.addEventListener('change', async (e) => {
 const file = e.target.files?.[0]; if (!file) return;
 const text = await file.text();
 try{
 const incoming = JSON.parse(text);
 // minimal sanity check
 if (!('targets' in incoming) || !('planned' in incoming)) throw new Error('Invalid file');
 Object.assign(state, incoming);
 persist(); renderAll();
 } catch(err){
 alert('Import failed: ' + err.message);
 } finally {
 importInput.value = '';
 }
});

resetBtn.addEventListener('click', () => {
 if (!confirm('Reset everything? This clears bills, transactions, and plan.')) return;
 localStorage.removeItem(KEY);
 state.month=''; state.income=0;
 state.targets={needs:50,wants:30,save:20};
 state.planned={needs:0,wants:0,save:0};
 state.bills=[]; state.tx=[];
 renderAll();
});

/* ============== Init ============== */
load();
renderAll();
// First-time hint: try an auto-allocate if fresh
if (!state.planned.needs && !state.planned.wants && !state.planned.save){
 autoAllocate(); renderAll();
}
</script>
</body>
</html>
