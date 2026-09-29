<script>
  import { onMount, onDestroy } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import {
    basemaps,
    MVT_TILES,
    MVT_SOURCE_LAYER,
    MAP_CENTER,
    MAP_ZOOM
  } from './config.js';

  export let basemap = 'street';

  let container;
  let map;
  let appliedBasemap = basemap;
  let popup;
  let layerMode = 'none';

  const SOURCE_ID = 'suppliers';
  const LAYER_ID = 'suppliers-circles';

  const circlePaint = {
    'circle-radius': ['interpolate', ['linear'], ['zoom'], 6, 3, 12, 6, 16, 9],
    'circle-color': '#788c5d',
    'circle-stroke-width': 1.5,
    'circle-stroke-color': '#faf9f5'
  };

  function escapeHtml(value) {
    return String(value ?? '—')
      .replaceAll('&', '&amp;')
      .replaceAll('<', '&lt;')
      .replaceAll('>', '&gt;');
  }

  function onSupplierClick(event) {
    const props = event.features?.[0]?.properties;
    if (!props) return;
    const area = props.plotareaha ?? '—';
    popup
      .setLngLat(event.lngLat)
      .setHTML(
        `<div class="popup">` +
          `<p class="popup-eyebrow">Supplier</p>` +
          `<h3 class="popup-title">${escapeHtml(props.entityname)}</h3>` +
          `<dl class="popup-meta">` +
            `<dt>Plot area</dt><dd>${escapeHtml(area)} ha</dd>` +
            `<dt>Region</dt><dd>${escapeHtml(props.regionlabel)}</dd>` +
          `</dl>` +
          `<span class="popup-tail" aria-hidden="true"></span>` +
        `</div>`
      )
      .addTo(map);
  }

  function onPointer() {
    map.getCanvas().style.cursor = 'pointer';
  }

  function onPointerOut() {
    map.getCanvas().style.cursor = '';
  }

  function clearSupplierLayer() {
    if (!map.getStyle()) return;
    if (map.getLayer(LAYER_ID)) {
      map.off('click', LAYER_ID, onSupplierClick);
      map.off('mouseenter', LAYER_ID, onPointer);
      map.off('mouseleave', LAYER_ID, onPointerOut);
      map.removeLayer(LAYER_ID);
    }
    if (map.getSource(SOURCE_ID)) map.removeSource(SOURCE_ID);
  }

  function bindClicks() {
    map.on('click', LAYER_ID, onSupplierClick);
    map.on('mouseenter', LAYER_ID, onPointer);
    map.on('mouseleave', LAYER_ID, onPointerOut);
  }

  function applyLayer() {
    if (!map.getStyle()) return;
    clearSupplierLayer();
    if (layerMode !== 'mvt') return;
    map.addSource(SOURCE_ID, {
      type: 'vector',
      tiles: [MVT_TILES],
      minzoom: 0,
      maxzoom: 14
    });
    map.addLayer({
      id: LAYER_ID,
      type: 'circle',
      source: SOURCE_ID,
      'source-layer': MVT_SOURCE_LAYER,
      paint: circlePaint
    });
    bindClicks();
  }

  export function useMvt() {
    layerMode = 'mvt';
    applyLayer();
  }

  export function disableMvt() {
    layerMode = 'none';
    popup?.remove();
    clearSupplierLayer();
  }

  onMount(() => {
    popup = new maplibregl.Popup({
      closeButton: true,
      closeOnClick: true,
      className: 'anthropic-popup',
      offset: 20
    });
    map = new maplibregl.Map({
      container,
      style: basemaps[basemap],
      center: MAP_CENTER,
      zoom: MAP_ZOOM,
      attributionControl: { compact: true }
    });

    // Zoom in/out — tanpa compass
    map.addControl(
      new maplibregl.NavigationControl({ showCompass: false }),
      'top-right'
    );
    // Compass terpisah
    map.addControl(
      new maplibregl.NavigationControl({ showZoom: false }),
      'top-right'
    );

    map.on('style.load', applyLayer);
  });

  $: if (map && basemap !== appliedBasemap) {
    appliedBasemap = basemap;
    popup?.remove();
    map.setStyle(basemaps[basemap], { diff: false });
  }

  onDestroy(() => {
    popup?.remove();
    map?.remove();
  });
</script>

<div class="map" bind:this={container}></div>

<style>
  .map {
    flex: 1;
    height: 100%;
    min-height: 400px;
    background: var(--canvas);
    transition: background-color 200ms ease;
  }

  /* ============================================
     MapLibre controls — 3 tombol bulat terpisah
     ============================================ */
  .map :global(.maplibregl-ctrl-top-right) {
    top: var(--space-base);
    right: var(--space-base);
    display: flex;
    flex-direction: column;
    gap: var(--space-sm);
    align-items: flex-end;
  }

  /* Grup transparan — tombolnya sendiri yang bulat */
  .map :global(.maplibregl-ctrl-top-right .maplibregl-ctrl-group) {
    background: transparent;
    border: none;
    border-radius: 0;
    box-shadow: none;
    overflow: visible;
    display: flex;
    flex-direction: column;
    gap: var(--space-sm);
    margin: 0;
  }

  /* Setiap tombol jadi bulat, solid cream */
  .map :global(.maplibregl-ctrl-top-right .maplibregl-ctrl-group button) {
    width: 40px;
    height: 40px;
    background: var(--canvas);
    border: 1px solid var(--surface-warm);
    border-radius: 50%;
    box-shadow: none;
    color: var(--ink);
    transition:
      background-color 140ms ease,
      border-color 140ms ease,
      transform 140ms ease;
  }

  .map :global(.maplibregl-ctrl-top-right .maplibregl-ctrl-group button:hover) {
    background: var(--surface-secondary);
    border-color: var(--hairline);
  }

  .map :global(.maplibregl-ctrl-top-right .maplibregl-ctrl-group button:active) {
    transform: scale(0.94);
  }

  /* Hapus divider antar tombol di dalam grup */
  .map :global(.maplibregl-ctrl-top-right .maplibregl-ctrl-group button + button) {
    border-top: 1px solid var(--surface-warm);
  }

  /* Ikon default MapLibre — invert sesuai tema */
  .map
    :global(
      .maplibregl-ctrl-top-right
        .maplibregl-ctrl-group
        button
        .maplibregl-ctrl-icon
    ) {
    filter: invert(1) opacity(0.85);
    transition: filter 200ms ease;
  }

  @media (prefers-color-scheme: dark) {
    .map
      :global(
        .maplibregl-ctrl-top-right
          .maplibregl-ctrl-group
          button
          .maplibregl-ctrl-icon
      ) {
      filter: invert(0) opacity(0.9);
    }
  }

  /* ============================================
     Attribution
     ============================================ */
  .map :global(.maplibregl-ctrl-attrib) {
    background: color-mix(in srgb, var(--canvas) 85%, transparent);
    border-radius: var(--radius-sm) 0 0 0;
    font-family: var(--font-serif);
    font-size: 11px;
    color: var(--text-muted);
  }

  .map :global(.maplibregl-ctrl-attrib a) {
    color: var(--ink-soft);
  }

  /* ============================================
     Popup — chat bubble
     ============================================ */
  .map :global(.anthropic-popup .maplibregl-popup-content) {
    background: var(--canvas);
    border: 1px solid var(--surface-warm);
    border-radius: 18px;
    box-shadow: none;
    padding: 18px 22px 16px;
    font-family: var(--font-serif);
    color: var(--ink);
    min-width: 220px;
    position: relative;
    overflow: visible;
  }

  .map :global(.anthropic-popup .maplibregl-popup-tip) {
    display: none;
  }

  .map :global(.anthropic-popup .popup-tail) {
    position: absolute;
    bottom: -8px;
    left: 50%;
    transform: translateX(-50%) rotate(45deg);
    width: 14px;
    height: 14px;
    background: var(--canvas);
    border-right: 1px solid var(--surface-warm);
    border-bottom: 1px solid var(--surface-warm);
    border-bottom-right-radius: 3px;
  }

  .map :global(.anthropic-popup .maplibregl-popup-close-button) {
    width: 26px;
    height: 26px;
    top: 8px;
    right: 8px;
    font-size: 16px;
    line-height: 1;
    padding: 0;
    color: var(--text-muted);
    background: transparent;
    border-radius: var(--radius-sm);
    transition:
      background-color 140ms ease,
      color 140ms ease;
  }

  .map :global(.anthropic-popup .maplibregl-popup-close-button:hover) {
    background: var(--surface-secondary);
    color: var(--ink);
  }

  .map :global(.anthropic-popup .popup-eyebrow) {
    margin: 0 0 6px;
    font-family: var(--font-sans);
    font-size: 11px;
    font-weight: 600;
    letter-spacing: 0.14em;
    text-transform: uppercase;
    color: var(--text-muted);
  }

  .map :global(.anthropic-popup .popup-title) {
    margin: 0 0 14px;
    font-family: var(--font-serif);
    font-size: 20px;
    font-weight: 400;
    line-height: 1.2;
    letter-spacing: -0.2px;
    color: var(--ink);
  }

  .map :global(.anthropic-popup .popup-meta) {
    margin: 0;
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 4px 16px;
    font-size: 14px;
  }

  .map :global(.anthropic-popup .popup-meta dt) {
    font-family: var(--font-sans);
    font-size: 12px;
    font-weight: 500;
    letter-spacing: 0.02em;
    color: var(--text-tertiary);
  }

  .map :global(.anthropic-popup .popup-meta dd) {
    margin: 0;
    font-family: var(--font-serif);
    color: var(--ink);
  }
</style>