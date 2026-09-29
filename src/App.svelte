<script>
  import MapView from './lib/MapView.svelte';

  let basemap = 'street';
  let mapView;
  let mvtOn = false;

  function toggleMvt() {
    mvtOn = !mvtOn;
    if (mvtOn) {
      mapView.useMvt();
    } else {
      mapView.disableMvt?.();
    }
  }

  const options = [
    { id: 'street', label: 'MAPID Street 2D' },
    { id: 'light', label: 'MAPID Light' },
    { id: 'satellite', label: 'MAPID Satellite' },
    { id: 'demo', label: 'MapLibre Demo (fallback)' }
  ];
</script>

<div class="app">
  <!-- Navbar -->
  <header class="navbar">
    <div class="brand">
      <span class="brand-mark">MAPID × BINUS</span>
      <h1 class="brand-title">Supplier WebGIS</h1>
    </div>
  </header>

  <!-- Map + floating cards -->
  <main class="stage">
    <MapView bind:this={mapView} {basemap} />

    <div class="panel">
      <div class="card">
        <div class="card-row">
          <label for="basemap">Basemap</label>
          <select id="basemap" bind:value={basemap}>
            {#each options as option}
              <option value={option.id}>{option.label}</option>
            {/each}
          </select>
        </div>
      </div>

      <div class="card">
        <div class="card-row">
          <label for="mvt-toggle">Vector tiles</label>
          <button
            id="mvt-toggle"
            type="button"
            class="toggle"
            class:on={mvtOn}
            role="switch"
            aria-checked={mvtOn}
            on:click={toggleMvt}
          >
            <span class="track" aria-hidden="true">
              <span class="thumb"></span>
            </span>
            <span class="toggle-label">{mvtOn ? 'On' : 'Off'}</span>
          </button>
        </div>
      </div>
    </div>
  </main>
</div>

<style>
  /* ============================================
     App shell
     ============================================ */
  .app {
    display: flex;
    flex-direction: column;
    height: 100vh;
    min-height: 100vh;
    background: var(--canvas);
  }

  /* ============================================
     Navbar
     ============================================ */
  .navbar {
    flex-shrink: 0;
    padding: var(--space-base) var(--space-lg);
    background: var(--canvas);
    border-bottom: 1px solid var(--surface-warm);
  }

  .brand {
    display: flex;
    align-items: baseline;
    gap: var(--space-base);
    flex-wrap: wrap;
  }

  .brand-mark {
    font-family: var(--font-sans);
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 0.14em;
    text-transform: uppercase;
    color: var(--text-muted);
  }

  .brand-title {
    margin: 0;
    font-family: var(--font-serif);
    font-size: 22px;
    font-weight: 400;
    letter-spacing: -0.3px;
    line-height: 1;
    color: var(--ink);
  }

  /* ============================================
     Stage (map area)
     ============================================ */
  .stage {
    position: relative;
    flex: 1;
    min-height: 0;
  }

  /* ============================================
     Floating panel (bottom-left, horizontal)
     ============================================ */
  .panel {
    position: absolute;
    bottom: var(--space-lg);
    left: var(--space-lg);
    display: flex;
    flex-direction: row;
    align-items: stretch;
    gap: var(--space-md);
    z-index: 5;
    max-width: calc(100% - 2 * var(--space-lg));
  }

  /* ============================================
     Card
     ============================================ */
  .card {
    background: var(--canvas);
    border: 1px solid var(--surface-warm);
    border-radius: 26px;
    padding: var(--space-md) var(--space-lg);
    display: flex;
    align-items: center;
    min-width: 0;
  }

  .card-row {
    display: flex;
    align-items: center;
    gap: var(--space-md);
    width: 100%;
  }

  .card label {
    flex-shrink: 0;
    font-family: var(--font-sans);
    font-size: 12px;
    font-weight: 500;
    letter-spacing: 0.02em;
    color: var(--text-muted);
    white-space: nowrap;
  }

  /* ============================================
     Select — transparan, chevron via SVG
     ============================================ */
  select {
    width: max-content;
    min-width: 0;
    max-width: 100%;
    height: 36px;
    padding: 0 20px 0 0;
    font-family: var(--font-serif);
    font-size: 15px;
    color: var(--ink);
    background-color: transparent;
    border: none;
    border-radius: var(--radius-sm);
    appearance: none;
    -webkit-appearance: none;
    background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='10' height='6' viewBox='0 0 10 6'><path d='M1 1l4 4 4-4' stroke='%23141413' stroke-width='1.5' fill='none' stroke-linecap='round' stroke-linejoin='round'/></svg>");
    background-repeat: no-repeat;
    background-position: right 2px center;
    background-size: 10px 6px;
    cursor: pointer;
    transition: color 140ms ease;
  }

  select:hover,
  select:focus-visible {
    outline: none;
    color: var(--ink);
  }

  /* Dark mode: chevron jadi cream */
  @media (prefers-color-scheme: dark) {
    select {
      background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='10' height='6' viewBox='0 0 10 6'><path d='M1 1l4 4 4-4' stroke='%23faf9f5' stroke-width='1.5' fill='none' stroke-linecap='round' stroke-linejoin='round'/></svg>");
    }
  }

  /* ============================================
     Toggle switch — transparan
     ============================================ */
  .toggle {
    display: flex;
    align-items: center;
    gap: var(--space-md);
    height: 36px;
    padding: 0;
    background-color: transparent;
    border: none;
    border-radius: var(--radius-sm);
    font-family: var(--font-serif);
    font-size: 15px;
    color: var(--ink);
    cursor: pointer;
    text-align: left;
    transition: color 140ms ease;
  }

  .toggle:hover,
  .toggle:focus-visible {
    outline: none;
    color: var(--ink);
  }

  /* Track */
  .track {
    position: relative;
    flex-shrink: 0;
    width: 36px;
    height: 20px;
    border-radius: 999px;
    background: var(--surface-warm);
    border: none;
    transition: background-color 160ms ease;
  }

  /* Thumb */
  .thumb {
    position: absolute;
    top: 2px;
    left: 2px;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: var(--canvas);
    border: none;
    transition: transform 180ms cubic-bezier(0.2, 0.8, 0.2, 1);
  }

  .toggle.on .track {
    background: var(--ink);
  }

  .toggle.on .thumb {
    transform: translateX(16px);
    background: var(--canvas);
  }

  /* Label On/Off */
  .toggle-label {
    font-family: var(--font-sans);
    font-size: 12px;
    font-weight: 600;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--text-muted);
    min-width: 24px;
    transition: color 140ms ease;
  }

  .toggle.on .toggle-label {
    color: var(--ink);
  }

  /* ============================================
     Responsive
     ============================================ */

  /* Tablet */
  @media (max-width: 900px) {
    .card {
      padding: var(--space-sm) var(--space-base);
    }
  }

  /* Mobile — panel tumpuk vertikal, card tetap menyamping */
  @media (max-width: 640px) {
    .navbar {
      padding: var(--space-md) var(--space-base);
    }

    .brand {
      gap: var(--space-sm);
    }

    .brand-title {
      font-size: 20px;
    }

    .panel {
      left: var(--space-base);
      right: var(--space-base);
      bottom: var(--space-base);
      flex-direction: column;
      gap: var(--space-sm);
      max-width: none;
    }

    .card {
      width: 100%;
      border-radius: 22px;
      padding: var(--space-sm) var(--space-base);
    }

    .card-row {
      justify-content: space-between;
    }
  }

  /* Mobile sangat kecil */
  @media (max-width: 380px) {
    .brand-mark {
      font-size: 11px;
      letter-spacing: 0.12em;
    }

    .brand-title {
      font-size: 18px;
    }

    .card {
      border-radius: 18px;
    }
  }
</style>