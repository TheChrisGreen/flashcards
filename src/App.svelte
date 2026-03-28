<script>
  const nums = Array.from({ length: 12 }, (_, i) => i + 1)

  // Filter state (applied values)
  let filterNum = 12
  let filterMode = 'upper' // 'upper' = 1..N, 'lower' = N..12, 'only' = ×N only

  // Modal draft state
  let showSettings = false
  let draftNum = 12
  let draftMode = 'upper'

  function inRange(a, b) {
    if (filterMode === 'upper') return a <= filterNum && b <= filterNum
    if (filterMode === 'lower') return a >= filterNum && b >= filterNum
    return a === filterNum || b === filterNum
  }

  // Reactive set — filterNum and filterMode must be referenced directly
  // so Svelte's compiler detects them as dependencies
  $: excludedCells = (() => {
    const s = new Set()
    for (const a of nums)
      for (const b of nums) {
        const included = filterMode === 'upper'
          ? (a <= filterNum && b <= filterNum)
          : filterMode === 'lower'
            ? (a >= filterNum && b >= filterNum)
            : (a === filterNum || b === filterNum)
        if (!included) s.add(`${a},${b}`)
      }
    return s
  })()

  function rangeLabel(num, mode) {
    if (mode === 'upper') return `1 – ${num}`
    if (mode === 'lower') return `${num} – 12`
    return `${num} times table`
  }

  function rangeCount(num, mode) {
    if (mode === 'upper') return num * num
    if (mode === 'lower') return (13 - num) * (13 - num)
    return 23 // 12 + 12 - 1 (row + column – corner)
  }

  function openSettings() {
    draftNum = filterNum
    draftMode = filterMode
    showSettings = true
  }

  function cancelSettings() {
    showSettings = false
  }

  function applySettings() {
    filterNum = draftNum
    filterMode = draftMode
    showSettings = false
    init()
  }

  function generateCards() {
    const cards = []
    for (let a = 1; a <= 12; a++)
      for (let b = 1; b <= 12; b++)
        if (inRange(a, b))
          cards.push({ a, b, answer: a * b })
    for (let i = cards.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1))
      ;[cards[i], cards[j]] = [cards[j], cards[i]]
    }
    return cards
  }

  function generateChoices(correct) {
    const allProducts = new Set()
    for (let a = 1; a <= 12; a++)
      for (let b = 1; b <= 12; b++)
        allProducts.add(a * b)

    const distractors = [...allProducts]
      .filter(v => v !== correct)
      .sort((x, y) => Math.abs(x - correct) - Math.abs(y - correct))
      .slice(0, 3)

    const options = [correct, ...distractors]
    for (let i = options.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1))
      ;[options[i], options[j]] = [options[j], options[i]]
    }
    return options
  }

  let cards, index, answered, wrongClicks, completed, screen, choices

  function init() {
    cards = generateCards()
    index = 0
    completed = new Set()
    screen = 'quiz'
    answered = false
    wrongClicks = new Set()
    choices = generateChoices(cards[0].answer)
  }

  function loadCard() {
    answered = false
    wrongClicks = new Set()
    choices = generateChoices(cards[index].answer)
  }

  init()

  $: card = cards[index]

  function selectChoice(i) {
    if (answered || wrongClicks.has(i)) return
    if (choices[i] === card.answer) {
      answered = true
      completed = new Set([...completed, `${card.a},${card.b}`])
    } else {
      wrongClicks = new Set([...wrongClicks, i])
    }
  }

  function next() {
    if (index + 1 >= cards.length) {
      screen = 'done'
    } else {
      index++
      loadCard()
    }
  }
</script>

{#if screen === 'quiz'}
  <div class="app">
    <header class="app-header">
      <span class="app-title">MathFlow</span>
      <div class="header-right">
        <span class="progress-badge">{completed.size} / {cards.length}</span>
        <button class="settings-btn" on:click={openSettings} aria-label="Settings">
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="12" r="3"/>
            <path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1-2.83 2.83l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-4 0v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83-2.83l.06-.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1 0-4h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 2.83-2.83l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 4 0v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 0 4h-.09a1.65 1.65 0 0 0-1.51 1z"/>
          </svg>
        </button>
      </div>
    </header>

    <main class="content">
      <div class="section-label">
        <div class="challenge-title">Daily Challenge</div>
        <div class="challenge-sub">Times tables {rangeLabel(filterNum, filterMode)}</div>
      </div>

      <div class="flashcard">
        <div class="equation">
          <span class="eq-num">{card.a}</span>
          <span class="eq-op">×</span>
          <span class="eq-num">{card.b}</span>
          <span class="eq-op">=</span>
          <span class="eq-answer" class:revealed={answered}>
            {answered ? card.answer : '?'}
          </span>
        </div>

        <div class="answer-area">
          <div class="choices" class:faded={answered}>
            {#each choices as choice, i}
              <button
                class="choice-btn"
                class:wrong={wrongClicks.has(i)}
                disabled={answered || wrongClicks.has(i)}
                on:click={() => selectChoice(i)}
              >
                {choice}
              </button>
            {/each}
          </div>
          {#if answered}
            <div class="next-overlay">
              <button class="next-btn" on:click={next}>Next →</button>
            </div>
          {/if}
        </div>
      </div>

      <div class="grid-section">
        <div class="grid-title">Mastery Grid</div>
        <div class="grid-scroll">
          <div class="mastery-grid">
            <div class="g-corner"></div>
            {#each nums as b}
              <div class="g-header">{b}</div>
            {/each}
            {#each nums as a}
              <div class="g-row-header">{a}</div>
              {#each nums as b}
                {@const isDone = completed.has(`${a},${b}`)}
                {@const isCurrent = !isDone && card.a === a && card.b === b}
                {@const isExcluded = excludedCells.has(`${a},${b}`)}
                <div class="g-cell" class:done={isDone} class:current={isCurrent} class:excluded={isExcluded}>
                  {#if !isExcluded}{isCurrent ? '?' : a * b}{/if}
                </div>
              {/each}
            {/each}
          </div>
        </div>
      </div>
    </main>
  </div>

{/if}

{#if showSettings}
  <!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions -->
  <div class="modal-overlay" on:click|self={cancelSettings}>
    <div class="modal">
      <div class="modal-header">
        <h2 class="modal-title">Settings</h2>
        <button class="modal-close" on:click={cancelSettings} aria-label="Close">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
        </button>
      </div>

      <div class="modal-section">
        <div class="modal-label">Filter number (1–12)</div>
        <div class="num-picker">
          {#each nums as n}
            <button
              class="num-btn"
              class:active={draftNum === n}
              on:click={() => draftNum = n}
            >{n}</button>
          {/each}
        </div>
      </div>

      <div class="modal-section">
        <div class="modal-label">Mode</div>
        <div class="mode-toggle">
          <button
            class="mode-btn"
            class:active={draftMode === 'upper'}
            on:click={() => draftMode = 'upper'}
          >Upper limit<span class="mode-sub">1 to {draftNum}</span></button>
          <button
            class="mode-btn"
            class:active={draftMode === 'lower'}
            on:click={() => draftMode = 'lower'}
          >Lower limit<span class="mode-sub">{draftNum} to 12</span></button>
          <button
            class="mode-btn"
            class:active={draftMode === 'only'}
            on:click={() => draftMode = 'only'}
          >Only<span class="mode-sub">× {draftNum}</span></button>
        </div>
        <p class="modal-desc">
          {#if draftMode === 'upper'}
            Questions use numbers <strong>1–{draftNum}</strong> · {rangeCount(draftNum, draftMode)} problems
          {:else if draftMode === 'lower'}
            Questions use numbers <strong>{draftNum}–12</strong> · {rangeCount(draftNum, draftMode)} problems
          {:else}
            Only the <strong>{draftNum} times table</strong> · {rangeCount(draftNum, draftMode)} problems
          {/if}
        </p>
      </div>

      <button class="apply-btn" on:click={applySettings}>Apply &amp; Close</button>
    </div>
  </div>
{/if}

{#if screen === 'done' && !showSettings}
  <div class="done-screen">
    <div class="trophy">🎉</div>
    <h1>Congratulations!</h1>
    <p>You completed all {cards.length} multiplication problems!</p>
    <button class="reset-btn" on:click={init}>Try Again?</button>
  </div>
{/if}

<style>
  :global(*, *::before, *::after) {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  :global(body) {
    background: #eef1f6;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    color: #1a2b4a;
    min-height: 100vh;
  }

  /* ── App shell ── */
  .app {
    max-width: 480px;
    margin: 0 auto;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }

  /* ── Header ── */
  .app-header {
    background: white;
    padding: 1rem 1.25rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.08);
    position: sticky;
    top: 0;
    z-index: 10;
  }

  .app-title {
    font-size: 1.5rem;
    font-weight: 800;
    color: #2563eb;
    letter-spacing: -0.02em;
  }

  .header-right {
    display: flex;
    align-items: center;
    gap: 0.6rem;
  }

  .progress-badge {
    background: #eef1f6;
    color: #6b7a9a;
    font-size: 0.8rem;
    font-weight: 600;
    padding: 0.3rem 0.75rem;
    border-radius: 1rem;
  }

  .settings-btn {
    width: 2.2rem;
    height: 2.2rem;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #eef1f6;
    color: #6b7a9a;
    border: none;
    border-radius: 50%;
    cursor: pointer;
    transition: background 0.15s, color 0.15s;
  }

  .settings-btn:hover {
    background: #dbeafe;
    color: #2563eb;
  }

  /* ── Main content ── */
  .content {
    padding: 1.25rem 1rem 2.5rem;
    display: flex;
    flex-direction: column;
    gap: 1.25rem;
    flex: 1;
  }

  .section-label {
    padding: 0 0.25rem;
  }

  .challenge-title {
    font-size: 1.5rem;
    font-weight: 800;
    color: #1a2b4a;
  }

  .challenge-sub {
    font-size: 0.9rem;
    color: #6b7a9a;
    margin-top: 0.2rem;
  }

  /* ── Flashcard ── */
  .flashcard {
    background: white;
    border-radius: 1.5rem;
    padding: 1.75rem 1.5rem 1.5rem;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.07);
  }

  .equation {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    margin-bottom: 1.5rem;
  }

  .eq-num {
    font-size: 3.5rem;
    font-weight: 800;
    color: #2563eb;
    line-height: 1;
  }

  .eq-op {
    font-size: 2rem;
    color: #93a8cc;
    font-weight: 400;
    line-height: 1;
  }

  .eq-answer {
    width: 80px;
    height: 80px;
    border-radius: 50%;
    background: #eef1f6;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2.2rem;
    font-weight: 700;
    color: #93a8cc;
    flex-shrink: 0;
    transition: background 0.25s, color 0.25s;
  }

  .eq-answer.revealed {
    background: #dcfce7;
    color: #16a34a;
  }

  /* ── Answer area (fixed height wrapper) ── */
  .answer-area {
    position: relative;
  }

  /* ── Choice buttons ── */
  .choices {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 0.75rem;
    transition: opacity 0.15s;
  }

  .choices.faded {
    visibility: hidden;
    pointer-events: none;
  }

  .choice-btn {
    padding: 1rem 0.5rem;
    font-size: 1.6rem;
    font-weight: 700;
    background: #f1f4f9;
    color: #1a2b4a;
    border: 2px solid transparent;
    border-radius: 1.5rem;
    cursor: pointer;
    min-height: 66px;
    transition: background 0.12s, border-color 0.12s, transform 0.08s;
  }

  .choice-btn:active:not(:disabled) {
    transform: scale(0.95);
  }

  .choice-btn:hover:not(:disabled) {
    background: #e2e8f4;
    border-color: #cbd5e8;
  }

  .choice-btn.wrong {
    background: #fee2e2;
    color: #dc2626;
    border-color: #fca5a5;
    cursor: not-allowed;
    opacity: 0.75;
  }

  /* ── Next button (overlaid over the hidden choices) ── */
  .next-overlay {
    position: absolute;
    inset: 0;
    display: flex;
    align-items: center;
  }

  .next-btn {
    width: 100%;
    height: 100%;
    padding: 1rem;
    font-size: 1.1rem;
    font-weight: 700;
    background: linear-gradient(135deg, #3b82f6, #2563eb);
    color: white;
    border: none;
    border-radius: 1.5rem;
    cursor: pointer;
    letter-spacing: 0.02em;
    transition: opacity 0.12s, transform 0.08s;
  }

  .next-btn:active {
    opacity: 0.88;
    transform: scale(0.98);
  }

  /* ── Mastery Grid ── */
  .grid-title {
    font-size: 1.1rem;
    font-weight: 700;
    color: #1a2b4a;
    margin-bottom: 0.6rem;
    padding: 0 0.25rem;
  }

  .grid-scroll {
    width: 100%;
  }

  .mastery-grid {
    display: grid;
    grid-template-columns: 28px repeat(12, 1fr);
    gap: 3px;
    padding: 0.75rem;
    background: white;
    border-radius: 1.25rem;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.06);
    width: 100%;
  }

  .g-corner {
    height: 22px;
  }

  .g-header {
    height: 22px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.62rem;
    font-weight: 700;
    color: #2563eb;
  }

  .g-row-header {
    height: 27px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.62rem;
    font-weight: 700;
    color: #2563eb;
  }

  .g-cell {
    height: 27px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 5px;
    font-size: 0.58rem;
    font-weight: 600;
    background: #f1f4f9;
    color: #b0bdd0;
    transition: background 0.2s, color 0.2s;
    min-width: 0;
  }

  .g-cell.current {
    background: #dbeafe;
    color: #2563eb;
    font-weight: 800;
  }

  .g-cell.done {
    background: #dcfce7;
    color: #16a34a;
  }

  .g-cell.excluded {
    background: transparent;
    box-shadow: none;
  }

  /* ── Settings modal ── */
  .modal-overlay {
    position: fixed;
    inset: 0;
    background: rgba(15, 23, 42, 0.45);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    padding: 1rem;
  }

  .modal {
    background: white;
    border-radius: 1.5rem;
    padding: 1.5rem 1.5rem 1.75rem;
    width: 100%;
    max-width: 480px;
    display: flex;
    flex-direction: column;
    gap: 1.25rem;
    box-shadow: 0 -4px 30px rgba(0, 0, 0, 0.12);
  }

  .modal-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .modal-title {
    font-size: 1.2rem;
    font-weight: 800;
    color: #1a2b4a;
  }

  .modal-close {
    width: 2rem;
    height: 2rem;
    padding: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #eef1f6;
    color: #6b7a9a;
    border: none;
    border-radius: 50%;
    cursor: pointer;
    transition: background 0.15s;
  }

  .modal-close:hover {
    background: #fee2e2;
    color: #dc2626;
  }

  .modal-section {
    display: flex;
    flex-direction: column;
    gap: 0.6rem;
  }

  .modal-label {
    font-size: 0.78rem;
    font-weight: 700;
    color: #6b7a9a;
    text-transform: uppercase;
    letter-spacing: 0.06em;
  }

  /* Number picker row */
  .num-picker {
    display: flex;
    gap: 0.4rem;
    flex-wrap: wrap;
  }

  .num-btn {
    width: 2.4rem;
    height: 2.4rem;
    padding: 0;
    font-size: 0.9rem;
    font-weight: 700;
    background: #f1f4f9;
    color: #6b7a9a;
    border: 2px solid transparent;
    border-radius: 0.6rem;
    cursor: pointer;
    transition: background 0.12s, color 0.12s, border-color 0.12s;
  }

  .num-btn.active {
    background: #dbeafe;
    color: #2563eb;
    border-color: #93c5fd;
  }

  .num-btn:hover:not(.active) {
    background: #e2e8f4;
  }

  /* Upper/lower toggle */
  .mode-toggle {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    gap: 0.6rem;
  }

  .mode-btn {
    padding: 0.75rem 0.5rem;
    font-size: 0.88rem;
    font-weight: 700;
    background: #f1f4f9;
    color: #6b7a9a;
    border: 2px solid transparent;
    border-radius: 1rem;
    cursor: pointer;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.2rem;
    transition: background 0.12s, color 0.12s, border-color 0.12s;
  }

  .mode-btn.active {
    background: #dbeafe;
    color: #2563eb;
    border-color: #93c5fd;
  }

  .mode-btn:hover:not(.active) {
    background: #e2e8f4;
  }

  .mode-sub {
    font-size: 0.72rem;
    font-weight: 600;
    opacity: 0.75;
  }

  .modal-desc {
    font-size: 0.82rem;
    color: #6b7a9a;
    line-height: 1.5;
  }

  .modal-desc strong {
    color: #2563eb;
  }

  .apply-btn {
    padding: 1rem;
    font-size: 1rem;
    font-weight: 700;
    background: linear-gradient(135deg, #3b82f6, #2563eb);
    color: white;
    border: none;
    border-radius: 1.25rem;
    cursor: pointer;
    transition: opacity 0.12s, transform 0.08s;
  }

  .apply-btn:active {
    opacity: 0.88;
    transform: scale(0.98);
  }

  /* ── Done screen ── */
  .done-screen {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    gap: 1.25rem;
    padding: 2rem;
    text-align: center;
    background: #eef1f6;
  }

  .trophy {
    font-size: 5rem;
    line-height: 1;
  }

  .done-screen h1 {
    font-size: 2.2rem;
    font-weight: 800;
    color: #1a2b4a;
  }

  .done-screen p {
    font-size: 1.05rem;
    color: #6b7a9a;
    max-width: 280px;
    line-height: 1.5;
  }

  .reset-btn {
    padding: 1rem 2.5rem;
    font-size: 1.1rem;
    font-weight: 700;
    background: linear-gradient(135deg, #3b82f6, #2563eb);
    color: white;
    border: none;
    border-radius: 1.5rem;
    cursor: pointer;
    margin-top: 0.5rem;
    transition: opacity 0.12s, transform 0.08s;
  }

  .reset-btn:active {
    opacity: 0.88;
    transform: scale(0.98);
  }
</style>
