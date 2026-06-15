# Integración de resultados en vivo — Mundial 2026 (Zero-Cost)

## Cómo funciona

La app sigue siendo 100% estática (HTML/JS), pero ahora hace `fetch()` a un
archivo `results.json` alojado gratis en GitHub. CORS viene habilitado por
defecto en `raw.githubusercontent.com`, así que funciona directo desde
WebView (Android `WebView` / iOS `WKWebView`) sin configuración extra,
siempre que el WebView tenga acceso a internet (no requiere `setAllowFileAccess`
ni excepciones de CORS).

## Pasos para activarlo

1. **Crea un repo público en GitHub** (gratis), ej. `tu-usuario/wc2026-data`.
2. Sube `results.json` a la raíz (o a una carpeta).
3. En `index.html`, reemplaza la constante:
   ```js
   const REMOTE_RESULTS_URL='https://raw.githubusercontent.com/USUARIO/REPO/main/results.json';
   ```
   con tu URL real, por ejemplo:
   ```
   https://raw.githubusercontent.com/tu-usuario/wc2026-data/main/results.json
   ```
4. Cada vez que quieras publicar resultados nuevos: edita `results.json` y
   haz `git commit` + `push`. La app lo detecta:
   - Al abrir la app.
   - Cada 5 minutos (auto-refresh, `AUTO_REFRESH_MS`).
   - Cuando la app vuelve a primer plano (visibilitychange).
   - Al tocar el botón ↻ en el header.

## Formato de `results.json`

```json
{
  "updated_at": "2026-06-17T18:30:00-05:00",
  "source": "Wikipedia / FIFA.com",
  "results": {
    "23": {"g1": 1, "g2": 2},
    "47": {"g1": 0, "g2": 0}
  }
}
```

- Las claves son el campo `id` de cada partido en `MATCHES` (ej. `23` =
  Ghana vs Panamá).
- Para partidos de eliminación directa usa los ids string: `"r16-1"`, `"qf-2"`, etc.
- `g1`/`g2` = goles del equipo 1 / equipo 2 según el orden definido en `MATCHES`.

## Cómo poblar `results.json` durante el torneo

Usa el skill `fifa-world-cup-2026-results` (ya instalado) para que Claude
busque los resultados reales (Wikipedia/FIFA/ESPN) y te devuelva el JSON
actualizado en el formato anterior. Tú solo copias/pegas y haces push.

## Comportamiento offline / fallback

- Si el `fetch()` falla (sin internet), la app usa la última copia cacheada
  en `localStorage` (`wc2026_remote_cache`).
- Los resultados ingresados manualmente por el usuario (`scores` en
  `localStorage`) se conservan; los remotos solo **sobrescriben** si difieren,
  priorizando la fuente oficial.
- Nunca se requiere API key, backend propio, ni dependencias de pago.

## WebView — notas específicas

- **Android (`WebView`)**: asegúrate de que `setDomStorageEnabled(true)` esté
  activo (para `localStorage`) y que el WebView tenga permiso `INTERNET`.
- **iOS (`WKWebView`)**: funciona out-of-the-box; `fetch()` y `localStorage`
  están soportados nativamente.
- No se usa `XMLHttpRequest` legado ni librerías externas — solo `fetch`
  nativo, compatible con WebViews modernos (Android 5+/iOS 11+).
