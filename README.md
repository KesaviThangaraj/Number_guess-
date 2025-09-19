# Number_guess-
A simple, interesting number guess game using python with reliable features of counting the number of guess taken for the correct guess , indicating the current position of answer to make a logical guess of the correct answer. Try and Have Fun!!position of answer to make a logical guess of the correct answer. Try and Have Fun!!

<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Number Guessing Game</title>
  <style>
    :root{
      --bg:#0f1724;--card:#0b1220;--accent:#7c3aed;--muted:#94a3b8;--ok:#10b981;--danger:#ef4444;
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; background:linear-gradient(180deg,var(--bg),#071025); color:#e6eef8; display:flex; align-items:center; justify-content:center; padding:32px;
    }
    .card{
      width:100%; max-width:720px; background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border:1px solid rgba(255,255,255,0.04); padding:28px; border-radius:14px; box-shadow: 0 10px 30px rgba(2,6,23,0.6);
    }
    header{display:flex; align-items:center; gap:16px}
    .logo{width:56px; height:56px; display:grid; place-items:center; border-radius:10px; background:linear-gradient(135deg,var(--accent),#5b21b6); font-weight:700}
    h1{margin:0;font-size:20px}
    p.lead{margin:6px 0 18px; color:var(--muted); font-size:14px}.controls{display:flex; gap:12px; align-items:center; margin-bottom:16px}
input[type=number]{flex:1; min-width:0; padding:12px 14px; border-radius:10px; border:1px solid rgba(255,255,255,0.06); background:transparent; color:inherit; font-size:16px}
button{padding:10px 14px; border-radius:10px; border:0; cursor:pointer; font-weight:600}
.btn-primary{background:var(--accent); color:white}
.btn-ghost{background:transparent; color:var(--muted); border:1px solid rgba(255,255,255,0.04)}

.status{padding:14px; border-radius:10px; background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005)); border:1px solid rgba(255,255,255,0.02); margin-bottom:14px}
.hint{font-size:16px; margin:0}
.meta{color:var(--muted); font-size:13px; margin-top:6px}

.guesses{display:flex; gap:8px; flex-wrap:wrap; margin-top:10px}
.chip{padding:8px 10px; border-radius:999px; background:rgba(255,255,255,0.03); border:1px solid rgba(255,255,255,0.02); font-weight:600}

.footer{display:flex; justify-content:space-between; align-items:center; margin-top:18px; color:var(--muted); font-size:13px}

.success{background:linear-gradient(90deg, rgba(16,185,129,0.12), rgba(124,58,237,0.06)); border:1px solid rgba(16,185,129,0.18); color:var(--ok)}
.danger{border:1px solid rgba(239,68,68,0.18); color:var(--danger)}

@media (max-width:520px){ .controls{flex-direction:column; align-items:stretch} button{width:100%} header{gap:12px} }

  </style>
</head>
<body>
  <div class="card" role="application" aria-label="Number guessing game">
    <header>
      <div class="logo">NG</div>
      <div>
        <h1>Number Guessing Game</h1>
        <p class="lead">Guess the secret number between the chosen range. I'll tell you higher or lower. Good luck!</p>
      </div>
    </header><section style="margin-top:12px">
  <div style="display:flex; gap:10px; align-items:center; margin-bottom:12px">
    <label for="min" style="color:var(--muted); font-size:13px">Min</label>
    <input id="min" type="number" value="1" min="0" style="width:90px" />
    <label for="max" style="color:var(--muted); font-size:13px">Max</label>
    <input id="max" type="number" value="100" min="1" style="width:110px" />
    <button id="setRange" class="btn-ghost" title="Set range">Set Range</button>
  </div>

  <div class="controls">
    <input id="guessInput" type="number" placeholder="Enter your guess" aria-label="Enter your guess" />
    <button id="guessBtn" class="btn-primary">Guess</button>
    <button id="restartBtn" class="btn-ghost">Restart</button>
  </div>

  <div id="status" class="status" aria-live="polite">
    <p id="hint" class="hint">Range: <span id="rangeText">1 — 100</span>. I have a secret number. Make your first guess.</p>
    <div id="meta" class="meta">Attempts: <span id="attempts">0</span></div>
    <div class="guesses" id="guessesList" aria-hidden="false"></div>
  </div>

  <div class="footer">
    <div>Best streak: <span id="best">0</span></div>
    <div style="opacity:0.9">Tip: press Enter to submit your guess</div>
  </div>
</section>

  </div>  <script>
    (function(){
      const minInput = document.getElementById('min');
      const maxInput = document.getElementById('max');
      const setRangeBtn = document.getElementById('setRange');
      const guessInput = document.getElementById('guessInput');
      const guessBtn = document.getElementById('guessBtn');
      const restartBtn = document.getElementById('restartBtn');
      const hint = document.getElementById('hint');
      const attemptsEl = document.getElementById('attempts');
      const guessesList = document.getElementById('guessesList');
      const rangeText = document.getElementById('rangeText');
      const bestEl = document.getElementById('best');
      const status = document.getElementById('status');

      let min = 1, max = 100, secret = randBetween(min,max);
      let attempts = 0; let guesses = []; let best = 0; let finished = false;

      function randBetween(a,b){
        a = Math.ceil(Number(a)); b = Math.floor(Number(b));
        return Math.floor(Math.random()*(b-a+1))+a;
      }

      function setRange(){
        const newMin = Number(minInput.value);
        const newMax = Number(maxInput.value);
        if(!Number.isFinite(newMin) || !Number.isFinite(newMax) || newMin>=newMax){
          flashMessage('Invalid range — ensure min < max and numbers are valid', 'danger');
          return;
        }
        min = Math.floor(newMin); max = Math.floor(newMax);
        rangeText.textContent = `${min} — ${max}`;
        restart();
      }

      function flashMessage(msg, kind){
        status.classList.remove('success','danger');
        if(kind==='success') status.classList.add('success');
        if(kind==='danger') status.classList.add('danger');
        hint.textContent = msg;
        setTimeout(()=>{ status.classList.remove('success','danger'); updateHint(); }, 2200);
      }

      function updateHint(){
        if(finished) return;
        hint.textContent = `Range: ${min} — ${max}. Make a guess.`;
        attemptsEl.textContent = attempts;
      }

      function renderGuesses(){
        guessesList.innerHTML = '';
        guesses.forEach(g=>{
          const d = document.createElement('div'); d.className='chip'; d.textContent = g;
          guessesList.appendChild(d);
        })
      }

      function handleGuess(){
        if(finished) return flashMessage('Game over — press Restart to play again', 'danger');
        const val = Number(guessInput.value);
        if(!Number.isFinite(val) || !Number.isInteger(val)) return flashMessage('Please enter a whole number', 'danger');
        if(val < min || val > max) return flashMessage(`Out of range: enter a number between ${min} and ${max}`, 'danger');

        attempts++;
        guesses.unshift(val);
        renderGuesses();
        attemptsEl.textContent = attempts;

        if(val === secret){
          finished = true;
          flashMessage(`Correct! The number was ${secret}. You took ${attempts} attempt${attempts>1?'s':''}.`, 'success');
          if(attempts > best) { best = attempts; bestEl.textContent = best; }
          guessInput.value = '';
          return;
        }

        if(val < secret){ flashMessage('Too low — try a higher number', 'danger'); }
        else { flashMessage('Too high — try a lower number', 'danger'); }
        guessInput.select();
      }

      function restart(){
        secret = randBetween(min,max); attempts = 0; guesses = []; finished = false;
        attemptsEl.textContent = attempts; renderGuesses(); updateHint(); guessInput.value=''; guessInput.focus();
      }

      // events
      setRangeBtn.addEventListener('click', setRange);
      guessBtn.addEventListener('click', handleGuess);
      restartBtn.addEventListener('click', restart);
      guessInput.addEventListener('keydown', (e)=>{ if(e.key === 'Enter'){ handleGuess(); } });

      // initialize
      rangeText.textContent = `${min} — ${max}`;
      updateHint();

    })();
  </script></body>
</html>
