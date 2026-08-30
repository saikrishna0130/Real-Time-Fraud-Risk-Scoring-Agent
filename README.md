# Real-Time-Fraud-Risk-Scoring-Agent
An AI agent that scores every payment in real time — spotting new devices, impossible location jumps, and sudden transaction bursts — and acts instantly: approving safe payments, flagging suspicious ones, and blocking fraud before the money ever leaves the account.
CODE:
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Fraud Risk Scoring Agent — Live Console</title>
<style>
  :root{
    --bg:#0B1120;
    --panel:#111A2E;
    --panel-border:#1E2A44;
    --teal:#2DD4BF;
    --amber:#F59E0B;
    --red:#F0665A;
    --purple:#A78BFA;
    --text:#E6EAF2;
    --muted:#7C89A8;
    --mono: ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas, monospace;
    --sans: -apple-system, BlinkMacSystemFont, "Segoe UI", system-ui, sans-serif;
  }
  *{box-sizing:border-box;}
  body{
    margin:0;
    background:radial-gradient(1200px 600px at 50% -10%, #16213A 0%, var(--bg) 60%);
    color:var(--text);
    font-family:var(--sans);
    min-height:100vh;
    padding:28px 20px 60px;
  }
  .wrap{max-width:1180px;margin:0 auto;}

  header{display:flex;justify-content:space-between;align-items:flex-end;flex-wrap:wrap;gap:16px;margin-bottom:22px;border-bottom:1px solid var(--panel-border);padding-bottom:18px;}
  .eyebrow{font-family:var(--mono);font-size:11px;letter-spacing:.14em;color:var(--purple);text-transform:uppercase;margin-bottom:6px;}
  h1{font-size:26px;margin:0;font-weight:650;letter-spacing:-0.01em;}
  .sub{color:var(--muted);font-size:13px;margin-top:6px;max-width:580px;line-height:1.5;}
  .status{display:flex;align-items:center;gap:8px;font-family:var(--mono);font-size:12px;color:var(--teal);}
  .dot{width:8px;height:8px;border-radius:50%;background:var(--teal);box-shadow:0 0 0 0 rgba(45,212,191,.7);animation:pulse 1.8s infinite;}
  @keyframes pulse{
    0%{box-shadow:0 0 0 0 rgba(45,212,191,.55);}
    70%{box-shadow:0 0 0 8px rgba(45,212,191,0);}
    100%{box-shadow:0 0 0 0 rgba(45,212,191,0);}
  }

  .stats{display:grid;grid-template-columns:repeat(4,1fr);gap:14px;margin-bottom:20px;}
  .stat{background:var(--panel);border:1px solid var(--panel-border);border-radius:10px;padding:16px 18px;}
  .stat .label{font-size:11px;color:var(--muted);text-transform:uppercase;letter-spacing:.08em;font-family:var(--mono);}
  .stat .value{font-size:26px;font-weight:650;margin-top:6px;font-family:var(--mono);}
  .stat.blocked .value{color:var(--red);}
  .stat.flagged .value{color:var(--amber);}
  .stat.clean .value{color:var(--teal);}
  .stat .delta{font-size:11px;color:var(--muted);margin-top:4px;}

  .gauge-panel{background:var(--panel);border:1px solid var(--panel-border);border-radius:10px;padding:20px 22px;margin-bottom:20px;}
  .gauge-panel h3{margin:0 0 14px;font-size:13px;font-family:var(--mono);color:var(--muted);text-transform:uppercase;letter-spacing:.08em;font-weight:600;}
  .risk-scale{position:relative;height:14px;border-radius:8px;background:linear-gradient(90deg,#2DD4BF 0%,#F59E0B 55%,#F0665A 100%);}
  .risk-marker{position:absolute;top:-8px;width:3px;height:30px;background:#fff;border-radius:2px;box-shadow:0 0 8px rgba(255,255,255,.6);transition:left .5s ease;}
  .risk-labels{display:flex;justify-content:space-between;font-family:var(--mono);font-size:11px;color:var(--muted);margin-top:10px;}
  .risk-current{margin-top:14px;font-family:var(--mono);font-size:13px;}

  .grid{display:grid;grid-template-columns:1.3fr 1fr;gap:16px;}
  @media(max-width:860px){.grid{grid-template-columns:1fr;} .stats{grid-template-columns:repeat(2,1fr);}}

  .panel{background:var(--panel);border:1px solid var(--panel-border);border-radius:10px;overflow:hidden;}
  .panel-head{padding:14px 18px;border-bottom:1px solid var(--panel-border);font-family:var(--mono);font-size:12px;color:var(--muted);text-transform:uppercase;letter-spacing:.08em;display:flex;justify-content:space-between;align-items:center;}
  .feed{max-height:440px;overflow-y:auto;}
  .row{padding:12px 18px;border-bottom:1px solid #16213866;font-family:var(--mono);font-size:12px;animation:in .35s ease;}
  @keyframes in{from{opacity:0;transform:translateY(-4px);}to{opacity:1;transform:translateY(0);}}
  .row-top{display:flex;gap:10px;align-items:center;margin-bottom:4px;}
  .row .tag{flex:0 0 auto;font-size:10px;padding:2px 7px;border-radius:20px;font-weight:700;letter-spacing:.03em;}
  .tag.clean{background:#0F3B33;color:var(--teal);}
  .tag.flagged{background:#3A2E10;color:var(--amber);}
  .tag.blocked{background:#3B1E1B;color:var(--red);}
  .row .id{color:var(--muted);flex:0 0 auto;}
  .row .score{margin-left:auto;font-weight:700;}
  .row .signals{color:var(--muted);font-size:11px;padding-left:2px;}
  .row .signals b{color:var(--purple);font-weight:600;}

  .breakdown{padding:16px 18px;}
  .cause-row{display:flex;align-items:center;gap:10px;margin-bottom:12px;font-family:var(--mono);font-size:12px;}
  .cause-row .name{width:170px;flex:0 0 auto;color:var(--muted);}
  .cause-row .bar{flex:1;height:8px;background:#0D1526;border-radius:6px;overflow:hidden;}
  .cause-row .bar-fill{height:100%;background:linear-gradient(90deg,#A78BFA,#F0665A);border-radius:6px;transition:width .6s ease;}
  .cause-row .count{width:34px;text-align:right;color:var(--text);}

  .actions{padding:6px 18px 16px;}
  .action-item{display:flex;gap:10px;padding:10px 0;border-bottom:1px solid #16213866;}
  .action-item:last-child{border-bottom:none;}
  .action-item .icon{width:22px;height:22px;border-radius:6px;background:#3B1E1B;color:var(--red);display:flex;align-items:center;justify-content:center;font-size:12px;flex:0 0 auto;}
  .action-item .body{font-size:12.5px;line-height:1.5;}
  .action-item .body .t{color:var(--text);}
  .action-item .body .m{color:var(--muted);font-family:var(--mono);font-size:11px;margin-top:2px;}

  .agent-logic{margin-top:20px;background:var(--panel);border:1px solid var(--panel-border);border-radius:10px;padding:18px 22px;}
  .agent-logic h3{margin:0 0 12px;font-size:13px;font-family:var(--mono);color:var(--muted);text-transform:uppercase;letter-spacing:.08em;}
  .steps{display:grid;grid-template-columns:repeat(4,1fr);gap:12px;}
  @media(max-width:860px){.steps{grid-template-columns:repeat(2,1fr);}}
  .step{border:1px solid var(--panel-border);border-radius:8px;padding:12px 14px;background:#0D1526;}
  .step .n{font-family:var(--mono);font-size:11px;color:var(--purple);}
  .step .t{font-size:12.5px;margin-top:4px;line-height:1.4;color:var(--text);}

  .controls{display:flex;gap:10px;margin-top:18px;}
  button{background:var(--purple);color:#1A0F33;border:none;font-weight:650;font-family:var(--sans);font-size:13px;padding:9px 16px;border-radius:7px;cursor:pointer;}
  button.secondary{background:transparent;color:var(--muted);border:1px solid var(--panel-border);}
  button:active{transform:scale(0.98);}
</style>
</head>
<body>
<div class="wrap">

  <header>
    <div>
      <div class="eyebrow">Track 2 · AI Risk Manager</div>
      <h1>Real-Time Fraud Risk Scoring Agent</h1>
      <div class="sub">Simulated live console: the agent scores every transaction for fraud risk in real time using behavioral signals, and autonomously flags or blocks high-risk transactions before they complete — no manual review queue needed.</div>
    </div>
    <div class="status"><span class="dot"></span> AGENT RUNNING</div>
  </header>

  <div class="stats">
    <div class="stat"><div class="label">Transactions Scored</div><div class="value" id="stat-total">0</div><div class="delta">simulated stream</div></div>
    <div class="stat clean"><div class="label">Clean / Approved</div><div class="value" id="stat-clean">0</div><div class="delta" id="stat-clean-pct">0% of volume</div></div>
    <div class="stat flagged"><div class="label">Flagged for Review</div><div class="value" id="stat-flagged">0</div><div class="delta" id="stat-flagged-pct">0% of volume</div></div>
    <div class="stat blocked"><div class="label">Auto-Blocked</div><div class="value" id="stat-blocked">0</div><div class="delta" id="stat-blocked-pct">0% of volume</div></div>
  </div>

  <div class="gauge-panel">
    <h3>Current Risk Gauge (last transaction)</h3>
    <div class="risk-scale"><div class="risk-marker" id="risk-marker" style="left:0%"></div></div>
    <div class="risk-labels"><span>0 · Low Risk</span><span>50 · Moderate</span><span>100 · Critical</span></div>
    <div class="risk-current" id="risk-current">Waiting for first transaction…</div>
  </div>

  <div class="grid">
    <div class="panel">
      <div class="panel-head"><span>Live Transaction Feed</span><span id="feed-count">0 events</span></div>
      <div class="feed" id="feed"></div>
    </div>

    <div>
      <div class="panel" style="margin-bottom:16px;">
        <div class="panel-head"><span>Risk Signal Breakdown</span></div>
        <div class="breakdown" id="breakdown"></div>
      </div>
      <div class="panel">
        <div class="panel-head"><span>Actions Dispatched</span></div>
        <div class="actions" id="actions"></div>
      </div>
    </div>
  </div>

  <div class="agent-logic">
    <h3>How the agent decides</h3>
    <div class="steps">
      <div class="step"><div class="n">01 · OBSERVE</div><div class="t">Reads each transaction's behavioral signals: device change, location jump, velocity, amount deviation from user history.</div></div>
      <div class="step"><div class="n">02 · SCORE</div><div class="t">Combines weighted signals into a single 0-100 real-time risk score, updated per transaction, not in nightly batches.</div></div>
      <div class="step"><div class="n">03 · DECIDE</div><div class="t">Applies thresholds: below 40 → approve, 40-74 → flag for human review, 75+ → auto-block before settlement.</div></div>
      <div class="step"><div class="n">04 · EXPLAIN</div><div class="t">Attaches the specific signals that drove the score, so reviewers see *why* — not just a number.</div></div>
    </div>
  </div>

  <div class="controls">
    <button id="btn-toggle">Pause Agent</button>
    <button class="secondary" id="btn-reset">Reset Simulation</button>
  </div>

</div>

<script>
const SIGNALS = [
  {key:"device_change", label:"New/Unrecognized Device", weight:22},
  {key:"location_jump", label:"Impossible Travel / Location Jump", weight:30},
  {key:"velocity", label:"High Transaction Velocity", weight:20},
  {key:"amount_deviation", label:"Amount Far Above User Average", weight:18},
  {key:"odd_hour", label:"Unusual Hour for This User", weight:10},
];

let total=0, clean=0, flagged=0, blocked=0;
const signalCounts = {};
SIGNALS.forEach(s=>signalCounts[s.key]=0);
let running = true;

function genTxnId(){
  return "TXN" + Math.floor(100000 + Math.random()*900000);
}

function pushFeedRow(html){
  const feed = document.getElementById("feed");
  const div = document.createElement("div");
  div.className = "row";
  div.innerHTML = html;
  feed.prepend(div);
  while(feed.children.length > 40) feed.removeChild(feed.lastChild);
  document.getElementById("feed-count").textContent = total + " events";
}

function pushAction(txnId, score, verdict, activeSignals){
  const actions = document.getElementById("actions");
  const div = document.createElement("div");
  div.className = "action-item";
  const label = verdict === "blocked" ? "Auto-blocked before settlement" : "Flagged and routed to human reviewer";
  div.innerHTML = `<div class="icon">${verdict === "blocked" ? "⛔" : "⚑"}</div>
    <div class="body">
      <div class="t">${label}</div>
      <div class="m">txn ${txnId} · risk score ${score} · ${activeSignals.map(s=>s.label).join(", ") || "no dominant signal"}</div>
    </div>`;
  actions.prepend(div);
  while(actions.children.length > 12) actions.removeChild(actions.lastChild);
}

function renderBreakdown(){
  const el = document.getElementById("breakdown");
  el.innerHTML = "";
  const max = Math.max(1, ...Object.values(signalCounts));
  SIGNALS.forEach(s=>{
    const count = signalCounts[s.key];
    const pct = Math.round((count/max)*100);
    el.innerHTML += `<div class="cause-row">
      <div class="name">${s.label}</div>
      <div class="bar"><div class="bar-fill" style="width:${pct}%"></div></div>
      <div class="count">${count}</div>
    </div>`;
  });
}

function renderStats(){
  document.getElementById("stat-total").textContent = total;
  document.getElementById("stat-clean").textContent = clean;
  document.getElementById("stat-flagged").textContent = flagged;
  document.getElementById("stat-blocked").textContent = blocked;
  document.getElementById("stat-clean-pct").textContent = (total? Math.round(clean/total*100):0) + "% of volume";
  document.getElementById("stat-flagged-pct").textContent = (total? Math.round(flagged/total*100):0) + "% of volume";
  document.getElementById("stat-blocked-pct").textContent = (total? Math.round(blocked/total*100):0) + "% of volume";
}

function tick(){
  if(!running) return;
  total++;
  const txnId = genTxnId();

  // simulate active signals
  const activeSignals = SIGNALS.filter(()=>Math.random() < 0.22);
  let score = activeSignals.reduce((sum,s)=>sum+s.weight, 0);
  score += Math.floor(Math.random()*12); // base noise
  score = Math.min(100, score);

  activeSignals.forEach(s=>signalCounts[s.key]++);

  let verdict, tagClass, tagText;
  if(score >= 75){
    verdict = "blocked"; blocked++; tagClass="blocked"; tagText="BLOCKED";
  } else if(score >= 40){
    verdict = "flagged"; flagged++; tagClass="flagged"; tagText="FLAGGED";
  } else {
    verdict = "clean"; clean++; tagClass="clean"; tagText="APPROVED";
  }

  const signalText = activeSignals.length
    ? activeSignals.map(s=>`<b>${s.label}</b>`).join(" · ")
    : "no elevated signals detected";

  pushFeedRow(`
    <div class="row-top">
      <span class="tag ${tagClass}">${tagText}</span>
      <span class="id">${txnId}</span>
      <span class="score">${score}/100</span>
    </div>
    <div class="signals">${signalText}</div>
  `);

  if(verdict !== "clean"){
    pushAction(txnId, score, verdict, activeSignals);
  }

  document.getElementById("risk-marker").style.left = score + "%";
  document.getElementById("risk-current").innerHTML =
    `Last: <b>${txnId}</b> scored <b>${score}/100</b> → <b>${tagText}</b>`;

  renderBreakdown();
  renderStats();
}

let interval = setInterval(tick, 900);

document.getElementById("btn-toggle").addEventListener("click", (e)=>{
  running = !running;
  e.target.textContent = running ? "Pause Agent" : "Resume Agent";
});

document.getElementById("btn-reset").addEventListener("click", ()=>{
  total=0; clean=0; flagged=0; blocked=0;
  Object.keys(signalCounts).forEach(k=>signalCounts[k]=0);
  document.getElementById("feed").innerHTML = "";
  document.getElementById("actions").innerHTML = "";
  document.getElementById("risk-marker").style.left = "0%";
  document.getElementById("risk-current").textContent = "Waiting for first transaction…";
  renderBreakdown();
  renderStats();
});

renderBreakdown();
renderStats();
</script>
</body>
</html>
