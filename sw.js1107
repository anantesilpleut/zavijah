// Palette° service worker — v0.18
// Strategy:
//   - App shell (index.html, manifest, icons): network-first with cache fallback,
//     so updates propagate fast but the app opens offline.
//   - CDN assets (fonts, MediaPipe WASM + models): cache-first with background
//     population — these are versioned/immutable and heavy (the segmenter model
//     is ~16 MB), caching them makes cold offline starts possible after one
//     online visit.
//   - Shopify product data/images: network-only (never cached here; the app has
//     its own 24h IDB cache for product JSON).

const VERSION = 'palette-v022';
const SHELL_CACHE = `${VERSION}-shell`;
const CDN_CACHE = `${VERSION}-cdn`;

const SHELL_ASSETS = [
  './',
  './index.html',
  './manifest.webmanifest',
  './icons/icon-192.png',
  './icons/icon-512.png',
  './icons/icon-512-maskable.png'
];

const CDN_HOSTS = [
  'fonts.googleapis.com',
  'fonts.gstatic.com',
  'cdn.jsdelivr.net',
  'storage.googleapis.com'
];

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(SHELL_CACHE).then((c) => c.addAll(SHELL_ASSETS)).then(() => self.skipWaiting())
  );
});

self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(keys.filter((k) => !k.startsWith(VERSION)).map((k) => caches.delete(k)))
    ).then(() => self.clients.claim())
  );
});

self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);
  if (event.request.method !== 'GET') return;

  // Never intercept product/brand fetches — the app manages those itself
  if (/products\.json|corsproxy|allorigins|codetabs|thingproxy|cors\.sh|cdn\.shopify\.com/.test(url.href)) {
    return; // default network behavior
  }

  // CDN assets: cache-first
  if (CDN_HOSTS.includes(url.host)) {
    event.respondWith(
      caches.open(CDN_CACHE).then(async (cache) => {
        const hit = await cache.match(event.request);
        if (hit) return hit;
        const res = await fetch(event.request);
        if (res.ok || res.type === 'opaque') cache.put(event.request, res.clone());
        return res;
      })
    );
    return;
  }

  // Same-origin shell: network-first, cache fallback
  if (url.origin === location.origin) {
    event.respondWith(
      fetch(event.request)
        .then((res) => {
          if (res.ok) {
            const copy = res.clone();
            caches.open(SHELL_CACHE).then((c) => c.put(event.request, copy));
          }
          return res;
        })
        .catch(() => caches.match(event.request).then((hit) => hit || caches.match('./index.html')))
    );
  }
});
