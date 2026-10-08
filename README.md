<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ПУЛЬТ УПРАВЛЕНИЯ · КОРАБЛЬ «ГОРЯЩИЙ»</title>
<style>
  :root {
    --neon-cyan: #00e5ff;
    --neon-pink: #ff2d95;
    --neon-amber: #ffb347;
    --panel-bg: #0a0e1a;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  html, body {
    height: 100%;
    overflow: hidden;
    background: #05070f;
    font-family: 'Courier New', monospace;
    color: var(--neon-cyan);
    user-select: none;
  }

  /* ---------- Сцена с пультом ---------- */
  .scene {
    position: fixed;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  /* Обёртка пульта — задаём пропорции через aspect-ratio,
     чтобы кнопки масштабировались вместе с картинкой */
  .console {
    position: relative;
    width: 100vw;
    height: 100vh;
    max-width: calc(100vh * (16 / 9)); /* если картинка 16:9 */
    aspect-ratio: 16 / 9;
    margin: auto;
    background: url('images/console.png') center/contain no-repeat;

  /* ---------- Кнопки поверх пульта ---------- */
  .btn {
    position: absolute;
    cursor: pointer;
    background: transparent;
    border: none;
    padding: 0;
    transition: transform .15s ease, filter .15s ease;
    /* позиция и размер задаются инлайном в % от размеров пульта */
  }

  .btn img {
    display: block;
    width: 100%;
    height: 100%;
    pointer-events: none;
    -webkit-user-drag: none;
  }

  .btn:hover:not(.activated) {
    transform: scale(1.08);
    filter: drop-shadow(0 0 12px var(--neon-cyan));
  }

  .btn:active:not(.activated) {
    transform: scale(0.96);
  }

  .btn.activated {
    cursor: default;
    filter: drop-shadow(0 0 8px var(--neon-pink));
  }

  /* Пульсация активной кнопки, чтобы игрок понял, куда жать */
  .btn:not(.activated) {
    animation: pulse 2.4s ease-in-out infinite;
  }
  @keyframes pulse {
    0%, 100% { filter: drop-shadow(0 0 2px rgba(0,229,255,.4)); }
    50%      { filter: drop-shadow(0 0 14px rgba(0,229,255,.9)); }
  }

  /* ---------- Вступительная плашка с заданием ---------- */
  .mission-overlay {
    position: fixed;
    inset: 0;
    background: rgba(2, 5, 15, .78);
    backdrop-filter: blur(3px);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    opacity: 1;
    transition: opacity .6s ease;
  }

  .mission-overlay.hidden {
    opacity: 0;
    pointer-events: none;
  }

  .mission-card {
    position: relative;
    max-width: 620px;
    width: 90%;
    padding: 40px 34px 34px;
    background:
      linear-gradient(180deg, rgba(10,14,26,.98), rgba(5,8,18,.98));
    border: 2px solid var(--neon-cyan);
    box-shadow:
      0 0 0 1px rgba(0,229,255,.3) inset,
      0 0 30px rgba(0,229,255,.5),
      0 0 60px rgba(0,229,255,.25);
    font-family: 'Courier New', monospace;
    color: var(--neon-cyan);
    text-align: center;
    letter-spacing: 1px;
    animation: cardIn .5s ease;
  }

  @keyframes cardIn {
    from { transform: translateY(20px) scale(.96); opacity: 0; }
    to   { transform: translateY(0)   scale(1);   opacity: 1; }
  }

  .mission-card::before,
  .mission-card::after {
    content: '';
    position: absolute;
    width: 18px; height: 18px;
    border: 2px solid var(--neon-pink);
  }
  .mission-card::before { top: -6px; left: -6px; border-right: none; border-bottom: none; }
  .mission-card::after  { bottom: -6px; right: -6px; border-left: none; border-top: none; }

  .mission-header {
    font-size: 13px;
    color: var(--neon-amber);
    letter-spacing: 4px;
    margin-bottom: 18px;
    text-transform: uppercase;
    opacity: .9;
  }

  .mission-title {
    font-size: 26px;
    color: var(--neon-pink);
    margin-bottom: 20px;
    text-shadow: 0 0 12px rgba(255,45,149,.8);
    letter-spacing: 3px;
    text-transform: uppercase;
  }

  .mission-text {
    font-size: 15px;
    line-height: 1.8;
    color: #b9e9f5;
    margin-bottom: 28px;
    text-align: left;
  }

  .mission-text .cursor {
    display: inline-block;
    width: 8px;
    background: var(--neon-cyan);
    animation: blink 1s steps(1) infinite;
    margin-left: 2px;
  }
  @keyframes blink { 50% { opacity: 0; } }

  .mission-btn {
    display: inline-block;
    padding: 12px 34px;
    font-family: inherit;
    font-size: 15px;
    letter-spacing: 3px;
    text-transform: uppercase;
    color: var(--neon-cyan);
    background: transparent;
    border: 2px solid var(--neon-cyan);
    cursor: pointer;
    transition: all .2s ease;
  }
  .mission-btn:hover {
    background: var(--neon-cyan);
    color: #05070f;
    box-shadow: 0 0 20px var(--neon-cyan);
  }

  /* ---------- Индикатор-подсказка внизу ---------- */
  .hint {
    position: fixed;
    bottom: 18px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 12px;
    letter-spacing: 3px;
    color: rgba(0,229,255,.55);
    text-transform: uppercase;
    z-index: 50;
    pointer-events: none;
    animation: blinkSlow 3s ease-in-out infinite;
  }
  @keyframes blinkSlow { 0%,100%{opacity:.35;} 50%{opacity:.9;} }

  /* CRT-сканер для атмосферы */
  .scanlines {
    position: fixed;
    inset: 0;
    pointer-events: none;
    z-index: 40;
    background: repeating-linear-gradient(
      to bottom,
      rgba(0,0,0,0) 0px,
      rgba(0,0,0,0) 2px,
      rgba(0,0,0,.18) 3px,
      rgba(0,0,0,.18) 4px
    );
    mix-blend-mode: multiply;
  }
</style>
</head>
<body>

  <!-- Сцена -->
  <div class="scene">
    <div class="console" id="console">

      <!-- ======================= КНОПКИ =======================
           top / left / width / height — в % от размеров картинки-пульта.
           Подгони под свою картинку.
           data-next — куда вести после активации.
           data-id   — уникальный id, чтобы помнить состояние. -->

      <button class="btn" id="btn-main"
              data-next="task1.html"
              data-id="btn-main"
              style="top: 62%; left: 47%; width: 8%; height: 14%;">
        <img src="images/button.png" alt="">
      </button>

      <!-- Пример второй кнопки — можно удалить/добавить -->
      <!--
      <button class="btn" id="btn-extra"
              data-next="task2.html"
              data-id="btn-extra"
              style="top: 30%; left: 20%; width: 6%; height: 10%;">
        <img src="images/button.png" alt="">
      </button>
      -->

    </div>
  </div>

  <div class="scanlines"></div>

  <!-- ======================= ВСТУПИТЕЛЬНАЯ ПЛАШКА ======================= -->
  <div class="mission-overlay" id="missionOverlay">
    <div class="mission-card">
      <div class="mission-header">// ВХОДЯЩАЯ ПЕРЕДАЧА · БОРТ «АВРОРА»</div>
      <div class="mission-title">ЗАДАНИЕ</div>
      <div class="mission-text" id="missionText"></div>
      <button class="mission-btn" id="acceptBtn">ПРИНЯТЬ</button>
    </div>
  </div>

  <div class="hint" id="hint">▼ найди нужную кнопку на пульте ▼</div>

  <script>
    /* ------------------------------------------------------------------
       1. ТЕКСТ ЗАДАНИЯ (печатается по буквам)
       ------------------------------------------------------------------ */
    const MISSION_TEXT =
      "С Днём Рождения, капитан!\\n\\n" +
      "Мы потеряли ваш подарок в одном из отсеков..." +
      "А ещё активировали системы безопасности которые можете открыть только вы....." +
      "Нажимайте на кнопки на пульте чтобы проверить все комнаты!";

    /* ------------------------------------------------------------------
       2. ВСТУПИТЕЛЬНАЯ ПЛАШКА
       ------------------------------------------------------------------ */
    const overlay = document.getElementById('missionOverlay');
    const textEl  = document.getElementById('missionText');
    const accept  = document.getElementById('acceptBtn');
    const hint    = document.getElementById('hint');

    // Печатающийся текст
    function typeText(str, el, speed = 22) {
      el.innerHTML = '';
      let i = 0;
      const cursor = document.createElement('span');
      cursor.className = 'cursor';
      cursor.innerHTML = '&nbsp;';
      el.appendChild(cursor);

      const tick = () => {
        if (i >= str.length) { cursor.remove(); return; }
        const ch = str[i++];
        if (ch === '\\n') {
          el.insertBefore(document.createElement('br'), cursor);
        } else {
          el.insertBefore(document.createTextNode(ch), cursor);
        }
        setTimeout(tick, speed);
      };
      tick();
    }

    // Показываем текст задания после небольшой задержки
    window.addEventListener('load', () => {
      setTimeout(() => typeText(MISSION_TEXT, textEl), 300);
    });

    // Кнопка "Эхбляя ну ладно" — закрываем плашку
    accept.addEventListener('click', () => {
      overlay.classList.add('hidden');
    });

    // Клик по фону плашки тоже закрывает (по желанию)
    overlay.addEventListener('click', (e) => {
      if (e.target === overlay) overlay.classList.add('hidden');
    });

    /* ------------------------------------------------------------------
       3. КНОПКИ ПУЛЬТА: активация + переход
       ------------------------------------------------------------------ */
    const STORAGE_KEY = 'buttons_state';

    function getState() {
      try { return JSON.parse(localStorage.getItem(STORAGE_KEY)) || {}; }
      catch { return {}; }
    }
    function saveState(state) {
      try { localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); }
      catch {}
    }

    // Инициализация всех кнопок
    document.querySelectorAll('.btn').forEach(btn => {
      const id       = btn.dataset.id;
      const nextUrl  = btn.dataset.next;
      const img      = btn.querySelector('img');

      // Восстанавливаем состояние "активирована"
      const state = getState();
      if (state[id]) {
        btn.classList.add('activated');
        img.src = 'ССЫЛКА НА КАРТИНКУ';
      }

      btn.addEventListener('click', () => {
        // Уже активирована — просто переходим (или ничего не делаем)
        if (btn.classList.contains('activated')) return;

        // Меняем картинку на «нажатую» и запоминаем
        img.src = 'ССЫЛКА НА КАРТИНКУ';
        btn.classList.add('activated');

        const s = getState();
        s[id] = true;
        saveState(s);

        // Плавный переход
        setTimeout(() => {
          document.body.style.transition = 'opacity .45s ease';
          document.body.style.opacity = 0;
          setTimeout(() => { window.location.href = nextUrl; }, 450);
        }, 350);
      });
    });

    /* ------------------------------------------------------------------
       4. Подсказка снизу — прячем после первого клика по пульту
       ------------------------------------------------------------------ */
    document.getElementById('console').addEventListener('click', () => {
      hint.style.display = 'none';
    }, { once: true });

  </script>
</body>
</html>
