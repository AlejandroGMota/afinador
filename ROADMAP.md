# Afinador · Roadmap

Última actualización: 2026-11-14

Documento vivo con el estado del proyecto, sus decisiones técnicas y todo lo que podría crecer. Las ideas van ordenadas por fase; dentro de cada fase, por impacto estimado.

---

## 1. Qué es

Afinador web de guitarra, gratis, sin anuncios, sin registro y sin backend. Todo el análisis de audio ocurre en el navegador del usuario; el sonido nunca sale del dispositivo.

**Principios (no negociables):**
1. **Gratis y sin anuncios.** Sin banners, sin popups, sin "versión pro".
2. **Privado.** Cero audio enviado a servidores. Si algún día hay analítica, sin cookies y sin datos personales.
3. **Ligero.** Abre rápido en un Android de gama baja con 3G. Meta: < 100 KB transferidos sin fuentes.
4. **Plano.** Un sitio estático; nada que mantener en el servidor.
5. **Español primero.** Pensado para México y Latinoamérica (cifrado latino Do-Re-Mi junto al americano C-D-E).

---

## 2. Estado actual (v0.1)

| Área | Estado |
|---|---|
| Detección de tono (pitchy / McLeod) | ✅ Verificado con señales sintéticas a 44.1 y 48 kHz, error < 0.15 cents |
| Micrófono real | ⚠️ Sin probar |
| Guitarra real | ⚠️ Sin probar |
| iOS Safari | ⚠️ Sin probar |
| Android Chrome / Edge | ⚠️ Sin probar |
| Deploy en Oracle (nginx + HTTPS) | ✅ `afinador.160-34-222-215.sslip.io` |
| Dominio `afinador.alejandrogmota.com` | ⏳ Esperando registro DNS en Namecheap |
| SEO on-page (meta, OG, JSON-LD, FAQ, sitemap) | ✅ |
| Google Search Console | ⏳ Pendiente (requiere dominio) |
| Repo público | ✅ github.com/AlejandroGMota/afinador |

### Arquitectura

```
Micrófono → getUserMedia (sin AEC/NS/AGC)
          → AudioContext → AnalyserNode (fftSize 4096)
          → getFloatTimeDomainData
          → pitchy.findPitch  → [frecuencia, claridad]
          → filtros: claridad ≥ 0.9, RMS ≥ 0.01, 60–420 Hz
          → mediana de 5 lecturas
          → cents = 1200·log2(f / objetivo)
          → UI: aguja ±50 cents, verde a ±5 cents, cuerda marcada tras 600 ms
```

### Archivos

| Archivo | Para qué |
|---|---|
| `index.html` | Toda la app: HTML, CSS y JS en un archivo |
| `robots.txt`, `sitemap.xml` | Indexación |
| `deploy/nginx-afinador.conf` | Server block de nginx en la VM |
| `README.md` | Cómo probar y subir |
| `ROADMAP.md` | Este documento |

### Constantes que hay que calibrar con micrófono real

| Constante | Valor | Efecto si sube | Efecto si baja |
|---|---|---|---|
| `MIN_RMS` | 0.01 | Ignora cuerdas suaves | Reacciona al ruido del cuarto |
| `MIN_CLARITY` | 0.9 | Más estable, tarda en reaccionar | Más rápido, más saltos |
| `IN_TUNE_CENTS` | 5 | Más permisivo | Más exigente |
| `HOLD_MS` | 600 | Tarda más en marcar la cuerda | Marca con lecturas fugaces |
| `BUFFER_SIZE` | 4096 | Mejor en graves, más latencia | Menos latencia, peor en E2 |

### Diseño

Paleta tomada de la Yamaha EG112 roja: cuerpo `#C42E27`, pickguard `#F4EEDD`, pastilla `#E9DFC3`, palo de rosa `#2B1A14`, afinado `#1E8A52`, desafinado `#B3261F`. Tipografías Space Grotesk e Inter. El panel es el pickguard; el selector de cuerdas es una pastilla single-coil con 6 polos. Canvas: https://claude.ai/artifact/67XM7APRUZ8g8XicqsqTrJ

---

## 3. Fase 0 — Terminar el lanzamiento (esta semana)

- [ ] Crear el registro A `afinador → 160.34.222.215` en Namecheap y verificar con `dig`.
- [ ] Ampliar el certificado al dominio con `certbot --expand`.
- [ ] Redirigir `sslip.io` → dominio con 301 (como en finanzas).
- [ ] Probar con guitarra real en Android (Edge y Chrome), empezando por la 6.ª cuerda.
- [ ] Probar en iOS Safari (pedir un iPhone prestado o BrowserStack gratis).
- [ ] Calibrar `MIN_RMS` y `MIN_CLARITY`.
- [ ] Dar de alta en Google Search Console y Bing Webmaster Tools; enviar el sitemap.
- [ ] Imagen `og-image.png` de 1200×630 para que la vista previa en WhatsApp/Facebook tenga imagen.
- [ ] Caché en nginx: `Cache-Control` largo para fuentes, corto para `index.html`; `gzip`/`brotli`.

## 4. Fase 1 — Robustez y velocidad (1–2 semanas)

- [ ] **Servir pitchy local** (`./pitchy.js`) para no depender de esm.sh: si esm.sh cae, el afinador deja de funcionar.
- [ ] **Fuentes locales** (subset latino de Space Grotesk e Inter en WOFF2) para quitar Google Fonts: más rápido y sin peticiones a terceros.
- [ ] **PWA:** `manifest.webmanifest` + service worker → instalable en el celular y funciona **sin internet** (clave para ensayos, tocadas y lugares sin señal).
- [ ] **Wake Lock API:** que la pantalla no se apague mientras afinas.
- [ ] **AudioWorklet** en lugar de `requestAnimationFrame` + AnalyserNode: análisis fuera del hilo principal, menos batería.
- [ ] Mensajes claros por cada error: sin permiso, sin micrófono, micrófono ocupado por otra app, navegador sin soporte.
- [ ] Instrucción visual para reactivar el permiso si se negó (cada navegador lo hace distinto).
- [ ] Indicador de nivel de entrada ("te escucho bajito, acércate").
- [ ] Pruebas automatizadas de la detección: Node + WAV sintéticos y grabaciones reales en `test/fixtures/`, corriendo en GitHub Actions.
- [ ] Lighthouse ≥ 95 en las cuatro categorías.
- [ ] Accesibilidad: `aria-live` para anunciar "baja / alta / afinada" a lectores de pantalla; probar con TalkBack.

## 5. Fase 2 — Más afinaciones e instrumentos (1–2 meses)

Cada uno con **su propia URL** (`/drop-d`, `/ukulele`, …): una página por búsqueda es lo que más visitas trae.

### Afinaciones de guitarra
- [ ] Drop D (D A D G B E)
- [ ] Medio tono abajo (Eb Ab Db Gb Bb Eb)
- [ ] Un tono abajo (D G C F A D)
- [ ] Drop C, Drop B
- [ ] Open G, Open D, Open E, DADGAD
- [ ] Guitarra de 7 y 8 cuerdas
- [ ] Afinación personalizada (el usuario elige nota por cuerda; se guarda en `localStorage`)

### Instrumentos (prioridad por búsquedas en México)
- [ ] **Bajo** de 4 y 5 cuerdas (E1 = 41.2 Hz → requiere buffer de 8192 y bajar `MIN_HZ` a ~30)
- [ ] **Ukulele** (G C E A)
- [ ] **Requinto** y **guitarra de doce cuerdas**
- [ ] **Jarana jarocha**, **vihuela** y **guitarrón** (mariachi): casi no hay afinadores online para ellos → poca competencia en Google
- [ ] **Bajo sexto** y **bajo quinto** (norteño): mismo argumento
- [ ] Violín, viola, violonchelo
- [ ] Mandolina, banjo
- [ ] **Modo cromático**: detecta cualquier nota (voz, instrumentos de viento, **piano**)

### Ajustes
- [ ] La de referencia configurable (440 Hz por defecto; 432, 442, 443 para orquestas)
- [ ] Cifrado latino / americano (Mi-La-Re vs E-A-D) como preferencia
- [ ] Capo virtual (transponer la afinación)
- [ ] Tolerancia "principiante" (±10 cents) vs "estudio" (±2 cents)

## 6. Fase 3 — Herramientas para el músico (2–4 meses)

- [ ] **Tono de referencia:** reproducir la nota de cada cuerda con Web Audio para afinar de oído.
- [ ] **Metrónomo** (`/metronomo`): BPM, compás, acento. Otra búsqueda con mucho volumen.
- [ ] **Diapasón / modo estroboscópico**: para quien quiere precisión de ±0.1 cents.
- [ ] **Afinación de octavas (entonación):** comparar cuerda al aire contra traste 12 para ajustar el puente.
- [ ] **Detector de acordes** (polifónico; ver Integraciones → ML).
- [ ] **Diccionario de acordes** con diagramas SVG.
- [ ] **Juego de oído:** adivina la nota / el intervalo.
- [ ] **Historial local:** qué tanto se desafina cada cuerda entre sesiones (señal de cuerdas viejas o clavijas flojas).
- [ ] Recordatorio "cambia tus cuerdas" según tiempo de uso (todo local).

## 7. Integraciones posibles

### 7.1 Hardware y entrada de audio
- **Interfaces de audio USB** (Focusrite, Behringer): elegir el dispositivo de entrada con `enumerateDevices`; afinar la eléctrica conectada, sin ruido de cuarto.
- **Adaptadores OTG/USB-C a jack** en Android: mismo caso, desde el celular.
- **Web MIDI API:** recibir notas de un **piano o teclado MIDI** para modo práctica (toca la nota que ves), y mandar MIDI a pedaleras que lo acepten.
- **Pedales Bluetooth de página** (actúan como teclado): cambiar de cuerda con el pie.
- **Web Bluetooth:** afinadores de clip o pedales con BLE abiertos (exploratorio).

### 7.2 Plataformas y distribución
- **PWA instalable** → icono en la pantalla de inicio sin pasar por tiendas.
- **Google Play vía TWA** (Trusted Web Activity con Bubblewrap): el mismo sitio como app de Android; cuota única de USD 25.
- **Microsoft Store** con PWABuilder (gratis).
- **Widget para OBS / streaming:** página `/overlay` transparente para músicos que hacen directos.
- **Embed para otros sitios** (`<iframe src=".../embed">`): blogs de guitarra lo ponen y cada embed es un enlace de vuelta → SEO.
- **Extensión de navegador** (Edge/Chrome): afinar sin abrir pestaña.
- **Smart TV / Chromecast**: aguja en pantalla grande para clases grupales.

### 7.3 Ecosistema propio
- **Blog (blog.alejandrogmota.com):** artículos que enlazan al afinador: "cómo afinar la guitarra sin afinador", "afinaciones alternativas", "por qué se desafina la guitarra", "cómo funciona un afinador (McLeod Pitch Method)" — este último encaja con el estilo técnico del canal.
- **YouTube «Mosaic»:** Short "construí un afinador en un solo archivo HTML" con enlace; video largo sobre procesamiento de señales en el navegador.
- **Celinki:** si algún día vende cuerdas, cejillas o accesorios de guitarra, un enlace discreto "¿cuerdas nuevas?" (sin anuncios de terceros; regla 1 se mantiene).
- **Portafolio (PortafolioDigital / dev-landing):** como proyecto destacado.
- **ScrumBoard:** seguimiento del roadmap como tablero.

### 7.4 Analítica que respete la privacidad
- **GoatCounter** o **Umami** autoalojado en la misma VM de Oracle (Docker, como el resto): sin cookies, sin banner de consentimiento.
- Alternativa sin JavaScript: contar visitas con los logs de nginx (`goaccess`).
- Métricas útiles: visitas, % que pulsa "Iniciar", % que niega el permiso, instrumento/afinación más usada, navegador.

### 7.5 Machine learning (en el navegador)
- **CREPE / SPICE con TensorFlow.js u ONNX Runtime Web:** detección de tono más robusta en cuartos ruidosos; comparar contra pitchy y usar el mejor.
- **Basic Pitch (Spotify, open source):** transcripción polifónica → detector de acordes.
- Todo local: el modelo se descarga una vez y se cachea con el service worker.

### 7.6 Comunidad y open source
- Aceptar PRs de afinaciones nuevas vía un `tunings.json`.
- Traducciones (inglés y portugués abren mercado: "guitar tuner online", "afinador de violão").
- Publicar en **Product Hunt**, **Hacker News (Show HN)**, **r/guitarra**, **r/Guitar**, grupos de Facebook de guitarristas en México.
- Listarlo en directorios de herramientas gratuitas (AlternativeTo, Free for Dev).

## 8. SEO: plan de contenido

| Página | Búsqueda objetivo |
|---|---|
| `/` | afinador de guitarra online, afinador de guitarra gratis |
| `/bajo` | afinador de bajo online |
| `/ukulele` | afinador de ukulele |
| `/drop-d` | afinación drop d |
| `/requinto`, `/jarana`, `/vihuela`, `/bajo-sexto` | afinador de requinto / jarana / vihuela / bajo sexto |
| `/cromatico` | afinador cromático online |
| `/metronomo` | metrónomo online |
| `/en/` | guitar tuner online |
| `/pt/` | afinador de violão online |

Cada página: título y descripción propios, `canonical`, `hreflang` entre idiomas, tabla de notas, FAQ en JSON-LD, entrada en `sitemap.xml`.

Cuando haya más de 3 páginas, conviene un generador estático pequeño (plantilla + `tunings.json` → HTML) para no copiar el mismo archivo a mano. Opciones: script en Node sin dependencias, o Astro.

## 9. Riesgos y limitaciones conocidas

- **iOS Safari:** históricamente el más problemático con Web Audio y permisos; puede requerir ajustes.
- **Graves:** E1 del bajo (41 Hz) necesita más muestras → más latencia.
- **Ruido ambiente:** en tocadas o ensayos con banda, el micrófono no basta → interfaz o modo de entrada de línea.
- **Dependencia de esm.sh y Google Fonts** hasta la Fase 1.
- **Un solo servidor:** si la VM de Oracle cae, el sitio cae. Mitigación barata: espejo en GitHub Pages o Cloudflare Pages (gratis) con el mismo HTML.
- **Sin pruebas automatizadas** del código del navegador todavía.

## 10. Ideas descartadas (y por qué)

- **Anuncios / AdSense:** rompe el principio 1 y empeora la velocidad.
- **Cuentas de usuario:** innecesarias; las preferencias caben en `localStorage`.
- **Backend propio:** nada lo requiere; mantenerlo estático es lo que lo hace gratis de operar.
