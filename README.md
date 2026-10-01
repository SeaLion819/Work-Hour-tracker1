
<!doctype html>
<html lang="it">

<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Gestione ore di lavoro</title>
<style>
:root{--ink:#17231f;--muted:#68766f;--line:#e3e9e5;--green:#176b4b;--pale:#eff7f2;--bg:#f6f8f6}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);font:16px/1.5 system-ui,-apple-system,"Segoe UI",sans-serif;color:var(--ink)}
main{max-width:850px;margin:48px auto;padding:0 20px}
.top{margin-bottom:24px}
.eyebrow{color:var(--green);font-weight:700;font-size:13px;letter-spacing:.08em;text-transform:uppercase}
h1{font-size:clamp(28px,5vw,38px);line-height:1.15;margin:8px 0}
p{color:var(--muted);margin:6px 0}
.card{background:#fff;border:1px solid var(--line);border-radius:18px;padding:24px;margin:16px 0;box-shadow:0 5px 22px #192d2010}
.status{display:flex;align-items:center;gap:10px;font-weight:650}
.dot{width:11px;height:11px;background:#9aa69e;border-radius:50%}
.dot.on{background:#20a267;box-shadow:0 0 0 5px #20a26720}
.timer{font-size:42px;font-variant-numeric:tabular-nums;font-weight:750;letter-spacing:-.04em;margin:12px 0 18px}
.actions{display:flex;gap:10px;flex-wrap:wrap}
button{font:inherit;font-weight:700;border:0;border-radius:10px;padding:12px 18px;cursor:pointer}
.primary{background:var(--green);color:#fff}
.primary:disabled{opacity:.45;cursor:default}
.secondary{background:#edf2ee;color:var(--ink);border:1px solid var(--line)}
.note{font-size:13px;margin-top:12px}
.stats{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.stat{background:var(--pale);border-radius:12px;padding:16px}
.stat small{display:block;color:var(--muted)}
.stat strong{font-size:24px}
.rowhead{display:flex;justify-content:space-between;align-items:center;gap:12px}
h2{font-size:19px;margin:0 0 12px}
input[type=month], input[type=datetime-local]{font:inherit;border:1px solid var(--line);padding:8px 10px;border-radius:9px;color:var(--ink)}
.tablewrap{overflow:auto}
table{border-collapse:collapse;width:100%;font-size:14px}
th,td{text-align:left;padding:11px 8px;border-bottom:1px solid var(--line);white-space:nowrap}
th{color:var(--muted);font-size:12px;text-transform:uppercase;letter-spacing:.04em}
.empty{text-align:center;color:var(--muted);padding:26px}
.btn-sm{padding:5px 8px;font-size:13px;background:transparent;border:0;cursor:pointer;font-weight:600}
.edit{color:var(--green)}
.del{color:#8a3b37}
.foot{font-size:12px;color:var(--muted);padding:0 4px 24px}

/* Stili Finestra Modale */
dialog{border:1px solid var(--line);border-radius:18px;padding:24px;background:#fff;max-width:450px;width:90%;box-shadow:0 10px 30px #0002}
dialog::backdrop{background:rgba(0,0,0,0.4)}
.form-group{margin-bottom:14px;display:flex;flex-direction:column;gap:6px}
.form-group label{font-size:13px;color:var(--muted);font-weight:600}

@media(max-width:560px){main{margin:28px auto}.card{padding:18px}.timer{font-size:36px}.stats{grid-template-columns:1fr 1fr}.rowhead{align-items:flex-start;flex-direction:column}}
</style>
</head>
<body>
<main>
<header class="top">
  <div class="eyebrow">LE TUE ORE DI LAVORO</div>
  <h1>Gestione ore di lavoro</h1>
  <p>Avvia il tuo turno con un click. Le tue ore verranno salvate al termine.</p>
</header>

<section class="card">
  <div class="status"><span class="dot" id="dot"></span><span id="status">Non stai lavorando</span></div>
  <div class="timer" id="timer">00:00:00</div>
  <div class="actions">
    <button class="primary" id="start">▶&nbsp; Inizia turno</button>
    <button class="primary" id="stop" disabled>■&nbsp; Termina turno</button>
  </div>
  <p class="note" id="startedAt">L'orario di inizio verrà registrato automaticamente.</p>
</section>

<section class="card">
  <h2>Riepilogo mensile</h2>
  <div class="stats">
    <div class="stat"><small>Turni effettuati</small><strong id="count">0</strong></div>
    <div class="stat"><small>Ore lavorate</small><strong id="total">0 h 00 min</strong></div>
  </div>
</section>

<section class="card">
  <div class="rowhead">
    <h2>Storico turni</h2>
    <input id="month" type="month" aria-label="Scegli mese">
  </div>
  <div class="tablewrap">
    <table>
      <thead>
        <tr>
          <th>Data</th>
          <th>Inizio</th>
          <th>Fine</th>
          <th>Ore lavorate</th>
          <th>Azioni</th>
        </tr>
      </thead>
      <tbody id="rows"></tbody>
    </table>
  </div>
  <div class="actions" style="margin-top:16px">
    <button class="secondary" id="addManual">+ Aggiungi turno manuale</button>
    <button class="secondary" id="export">Scarica report CSV</button>
  </div>
  <p class="note">Apri il file CSV in Excel o invialo al tuo datore di lavoro.</p>
</section>

<div class="foot">
  I tuoi dati sono salvati nel browser su questo dispositivo e non vengono inviati a nessun server. Ricordati di cliccare “Termina turno” alla fine del lavoro.
</div>
</main>

<!-- Finestra Modale per Inserimento/Modifica Turni -->
<dialog id="shiftDialog">
  <form id="shiftForm" method="dialog">
    <h3 id="dialogTitle" style="margin-top:0">Aggiungi turno manuale</h3>
    <input type="hidden" id="editId">
    <div class="form-group">
      <label for="manualStart">Data e ora di inizio</label>
      <input type="datetime-local" id="manualStart" required>
    </div>
    <div class="form-group">
      <label for="manualEnd">Data e ora di fine</label>
      <input type="datetime-local" id="manualEnd" required>
    </div>
    <div class="actions" style="margin-top:20px; justify-content: flex-end;">
      <button type="button" class="secondary" id="closeDialog">Annulla</button>
      <button type="submit" class="primary">Salva</button>
    </div>
  </form>
</dialog>

<script>
const KEY='work-time-tracker-v1';
let data;
try {
  data = JSON.parse(localStorage.getItem(KEY)) || { sessions: [], active: null };
} catch {
  data = { sessions: [], active: null };
}

const $ = id => document.getElementById(id), pad = n => String(n).padStart(2,'0');

function save() {
  localStorage.setItem(KEY, JSON.stringify(data));
  render();
}

function time(d) {
  return new Intl.DateTimeFormat('it-IT', { hour: '2-digit', minute: '2-digit' }).format(new Date(d));
}

function duration(a, b) {
  let ms = new Date(b) - new Date(a);
  let mins = Math.max(0, Math.round(ms / 60000));
  return { mins, text: `${Math.floor(mins / 60)} h ${pad(mins % 60)} min` };
}

function toLocalDatetimeInput(isoStr) {
  const d = new Date(isoStr);
  return `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())}T${pad(d.getHours())}:${pad(d.getMinutes())}`;
}

function render() {
  let active = !!data.active;
  $('dot').className = 'dot' + (active ? ' on' : '');
  $('status').textContent = active ? 'Al lavoro — turno in corso' : 'Non stai lavorando';
  $('start').disabled = active;
  $('stop').disabled = !active;
  $('startedAt').textContent = active 
    ? 'Inizio: ' + new Intl.DateTimeFormat('it-IT', { dateStyle: 'medium', timeStyle: 'short' }).format(new Date(data.active))
    : 'L\'orario di inizio verrà registrato automaticamente.';

  let month = $('month').value || new Date().toISOString().slice(0,7);
  let sessions = data.sessions.filter(s => s.start.slice(0,7) === month);
  let mins = sessions.reduce((n, s) => n + duration(s.start, s.end).mins, 0);

  $('count').textContent = sessions.length;
  $('total').textContent = `${Math.floor(mins / 60)} h ${pad(mins % 60)} min`;

  let sorted = [...sessions].sort((a, b) => b.start.localeCompare(a.start));
  $('rows').innerHTML = sorted.length 
    ? sorted.map(s => `
        <tr>
          <td>${new Intl.DateTimeFormat('it-IT', { day: '2-digit', month: '2-digit', year: 'numeric' }).format(new Date(s.start))}</td>
          <td>${time(s.start)}</td>
          <td>${time(s.end)}</td>
          <td>${duration(s.start, s.end).text}</td>
          <td>
            <button class="btn-sm edit" data-id="${s.id}">Modifica</button>
            <button class="btn-sm del" data-id="${s.id}">Elimina</button>
          </td>
        </tr>
      `).join('')
    : '<tr><td colspan="5" class="empty">Nessun turno registrato in questo mese.</td></tr>';

  // Gestione pulsanti elimina e modifica
  document.querySelectorAll('.del').forEach(b => b.onclick = () => {
    data.sessions = data.sessions.filter(s => s.id !== b.dataset.id);
    save();
  });

  document.querySelectorAll('.edit').forEach(b => b.onclick = () => {
    let session = data.sessions.find(s => s.id === b.dataset.id);
    if (!session) return;
    $('editId').value = session.id;
    $('dialogTitle').textContent = 'Modifica turno';$('manualStart').value = toLocalDatetimeInput(session.start);
    $('manualEnd').value = toLocalDatetimeInput(session.end);$('shiftDialog').showModal();
  });
}

// Eventi Avvio/Fine turno live
$('start').onclick = () => { data.active = new Date().toISOString(); save(); };$('stop').onclick = () => {
  if (!data.active) return;
  let end = new Date().toISOString();
  data.sessions.push({
    id: crypto.randomUUID ? crypto.randomUUID() : String(Date.now()),
    start: data.active,
    end
  });
  data.active = null;
  save();
};

$('month').value = new Date().toISOString().slice(0,7);$('month').onchange = render;

// Gestione Modale (Aggiunta/Modifica manuale)
$('addManual').onclick = () => {
  $('editId').value = '';$('dialogTitle').textContent = 'Aggiungi turno manuale';
  let now = new Date();
  let eightHoursAgo = new Date(now.getTime() - 8 * 60 * 60 * 1000);
  $('manualStart').value = toLocalDatetimeInput(eightHoursAgo);
  $('manualEnd').value = toLocalDatetimeInput(now);$('shiftDialog').showModal();
};

$('closeDialog').onclick = () =>$('shiftDialog').close();

$('shiftForm').onsubmit = (e) => {
  e.preventDefault();
  let startVal = new Date($('manualStart').value);
  let endVal = new Date($('manualEnd').value);

  if (endVal <= startVal) {
    alert('L\'ora di fine deve essere successiva all\'ora di inizio.');
    return;
  }

  let editId = $('editId').value;
  if (editId) {
    // Modifica turno esistente
    let session = data.sessions.find(s => s.id === editId);
    if (session) {
      session.start = startVal.toISOString();
      session.end = endVal.toISOString();
    }
  } else {
    // Inserimento nuovo turno
    data.sessions.push({
      id: crypto.randomUUID ? crypto.randomUUID() : String(Date.now()),
      start: startVal.toISOString(),
      end: endVal.toISOString()
    });
  }

  save();
  $('shiftDialog').close();
};

// Esportazione CSV in Italiano
$('export').onclick = () => {
  let month = $('month').value;
  let rows = data.sessions.filter(s => s.start.slice(0,7) === month).sort((a,b) => a.start.localeCompare(b.start));
  let lines = [
    ['Data', 'Ora Inizio', 'Ora Fine', 'Ore Lavorate'],
    ...rows.map(s => [
      new Intl.DateTimeFormat('it-IT').format(new Date(s.start)),
      time(s.start),
      time(s.end),
      (duration(s.start, s.end).mins / 60).toFixed(2).replace('.', ',')
    ])
  ];
  let csv = '\ufeff' + lines.map(r => r.map(v => '"' + String(v).replaceAll('"', '""') + '"').join(';')).join('\r\n');
  let a = document.createElement('a');
  a.href = URL.createObjectURL(new Blob([csv], { type: 'text/csv;charset=utf-8' }));
  a.download = `report-ore-lavoro-${month}.csv`;
  a.click();
  URL.revokeObjectURL(a.href);
};

// Timer live
setInterval(() => {
  if (!data.active) { $('timer').textContent = '00:00:00'; return; }
  let s = Math.floor((Date.now() - new Date(data.active).getTime()) / 1000);
  $('timer').textContent = `${pad(Math.floor(s / 3600))}:${pad(Math.floor(s % 3600 / 60))}:${pad(s % 60)}`;
}, 500);

render();
</script>
</body>
</html>
