<script>
  // Generate all multiplication pairs 1–12
  function generateCards(randomOrder) {
    const cards = []
    for (let a = 1; a <= 12; a++) {
      for (let b = 1; b <= 12; b++) {
        cards.push({ a, b, answer: a * b })
      }
    }
    if (randomOrder) {
      for (let i = cards.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [cards[i], cards[j]] = [cards[j], cards[i]]
      }
    }
    return cards
  }

  // Screens: 'start' | 'quiz' | 'done'
  let screen = 'start'
  let randomOrder = true
  let lightMode = false

  $: if (typeof document !== 'undefined') {
    document.body.dataset.theme = lightMode ? 'light' : 'dark'
  }
  let cards = []
  let index = 0
  let revealed = false

  $: card = cards[index]
  $: progress = `${index + 1} / ${cards.length}`

  function start() {
    cards = generateCards(randomOrder)
    index = 0
    revealed = false
    screen = 'quiz'
  }

  function advance() {
    if (!revealed) {
      revealed = true
    } else {
      if (index + 1 >= cards.length) {
        screen = 'done'
      } else {
        index++
        revealed = false
      }
    }
  }

  function handleKey(e) {
    if (e.code === 'Space') {
      e.preventDefault()
      if (screen === 'start') {
        start()
      } else if (screen === 'quiz') {
        advance()
      } else if (screen === 'done') {
        screen = 'start'
      }
    }
  }
</script>

<svelte:window on:keydown={handleKey} />

<button class="theme-toggle" on:click={() => lightMode = !lightMode} aria-label="Toggle theme">
  {#if lightMode}
    <!-- Moon icon -->
    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/></svg>
  {:else}
    <!-- Sun icon -->
    <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="5"/><line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/><line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/></svg>
  {/if}
</button>

<main>
  {#if screen === 'start'}
    <div class="start">
      <h1>Math Flash Cards</h1>
      <p class="subtitle">Times tables 1 &times; 1 through 12 &times; 12</p>

      <label class="toggle-row">
        <span>Order</span>
        <div class="toggle-group">
          <button
            class="toggle-btn"
            class:active={!randomOrder}
            on:click={() => randomOrder = false}
          >In Order</button>
          <button
            class="toggle-btn"
            class:active={randomOrder}
            on:click={() => randomOrder = true}
          >Random</button>
        </div>
      </label>

      <button class="start-btn" on:click={start}>Start</button>
      <p class="hint">or press Space</p>
    </div>

  {:else if screen === 'quiz'}
    <div class="progress">{progress}</div>
    <div class="card" on:click={advance}>
      <div class="problem">
        {card.a} &times; {card.b} =
        <span class="answer" class:revealed>
          {#if revealed}
            {card.answer}
          {:else}
            ?
          {/if}
        </span>
      </div>
    </div>
    <p class="hint">Press Space or click the card</p>

  {:else}
    <div class="done">
      <h1>All done!</h1>
      <p>You went through all {cards.length} cards.</p>
      <button on:click={() => screen = 'start'}>Start Again</button>
      <p class="hint">or press Space</p>
    </div>
  {/if}
</main>

<style>
  /* ── Theme tokens ── */
  :global(body) {
    --bg:         #1a1a2e;
    --bg-card:    #16213e;
    --border:     #0f3460;
    --text:       #eee;
    --text-muted: #888;
    --text-dim:   #555;
    --accent:     #4ecca3;
    --danger:     #e94560;
    --btn-bg:     #0f3460;
    --btn-color:  #fff;
    --shadow:     rgba(0,0,0,0.4);
    transition: background 0.25s, color 0.25s;
  }

  :global(body[data-theme="light"]) {
    --bg:         #f0f4f8;
    --bg-card:    #ffffff;
    --border:     #b0c4de;
    --text:       #1a1a2e;
    --text-muted: #6b7a8d;
    --text-dim:   #aab4c0;
    --accent:     #0a8f6e;
    --danger:     #c0143c;
    --btn-bg:     #dce8f5;
    --btn-color:  #1a1a2e;
    --shadow:     rgba(0,0,0,0.12);
  }

  :global(body) {
    margin: 0;
    background: var(--bg);
    color: var(--text);
    font-family: 'Segoe UI', system-ui, sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
  }

  /* ── Theme toggle button ── */
  .theme-toggle {
    position: fixed;
    top: 1rem;
    right: 1rem;
    width: 2.4rem;
    height: 2.4rem;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    background: var(--bg-card);
    color: var(--text);
    border: 2px solid var(--border);
    border-radius: 50%;
    cursor: pointer;
    transition: background 0.2s, color 0.2s, border-color 0.2s;
  }

  .theme-toggle:hover {
    background: var(--accent);
    color: var(--bg);
    border-color: var(--accent);
  }

  /* ── Layout ── */
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1.5rem;
  }

  .progress {
    font-size: 1rem;
    color: var(--text-muted);
    letter-spacing: 0.05em;
  }

  /* ── Flash card ── */
  .card {
    background: var(--bg-card);
    border: 2px solid var(--border);
    border-radius: 1.5rem;
    width: min(420px, calc(100vw - 4rem));
    height: min(260px, calc((100vw - 4rem) * 0.62));
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    box-shadow: 0 8px 32px var(--shadow);
    transition: transform 0.1s, box-shadow 0.1s;
    user-select: none;
  }

  .card:active {
    transform: scale(0.97);
    box-shadow: 0 4px 16px var(--shadow);
  }

  .problem {
    font-size: clamp(2rem, 10vw, 3.5rem);
    font-weight: 700;
    display: flex;
    align-items: center;
    gap: 0.4rem;
  }

  .answer {
    display: inline-block;
    min-width: 2.5rem;
    text-align: center;
    color: var(--danger);
    transition: color 0.2s;
  }

  .answer.revealed {
    color: var(--accent);
  }

  /* ── Done screen ── */
  .done {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
    text-align: center;
  }

  .done h1 {
    font-size: 3rem;
    margin: 0;
    color: var(--accent);
  }

  .done p {
    font-size: 1.2rem;
    color: var(--text-muted);
    margin: 0;
  }

  /* ── Shared buttons ── */
  button {
    padding: 0.8rem 2.5rem;
    font-size: 1.1rem;
    font-weight: 600;
    background: var(--btn-bg);
    color: var(--btn-color);
    border: 2px solid var(--accent);
    border-radius: 0.75rem;
    cursor: pointer;
    transition: background 0.2s, color 0.2s;
  }

  button:hover {
    background: var(--accent);
    color: var(--bg);
  }

  .hint {
    font-size: 0.85rem;
    color: var(--text-dim);
    margin: 0;
  }

  /* ── Start screen ── */
  .start {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1.5rem;
    text-align: center;
  }

  .start h1 {
    font-size: 2.8rem;
    margin: 0;
    color: var(--accent);
  }

  .subtitle {
    font-size: 1rem;
    color: var(--text-muted);
    margin: 0;
  }

  .toggle-row {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.6rem;
    font-size: 0.9rem;
    color: var(--text-muted);
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }

  .toggle-group {
    display: flex;
    border: 2px solid var(--border);
    border-radius: 0.6rem;
    overflow: hidden;
  }

  .toggle-btn {
    padding: 0.55rem 1.4rem;
    font-size: 0.95rem;
    font-weight: 600;
    background: transparent;
    color: var(--text-muted);
    border: none;
    border-radius: 0;
    cursor: pointer;
    transition: background 0.15s, color 0.15s;
  }

  .toggle-btn:first-child {
    border-right: 2px solid var(--border);
  }

  .toggle-btn.active {
    background: var(--btn-bg);
    color: var(--accent);
  }

  .toggle-btn:hover:not(.active) {
    background: var(--bg-card);
    color: var(--text);
  }

  .start-btn {
    padding: 0.8rem 3rem;
    font-size: 1.2rem;
    font-weight: 700;
    background: var(--btn-bg);
    color: var(--btn-color);
    border: 2px solid var(--accent);
    border-radius: 0.75rem;
    cursor: pointer;
    transition: background 0.2s, color 0.2s;
    letter-spacing: 0.05em;
  }

  .start-btn:hover {
    background: var(--accent);
    color: var(--bg);
  }
</style>
