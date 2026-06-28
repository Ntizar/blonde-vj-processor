# 🎭 BLONDE VOULAIN — VJ Processor para Concierto

## 📋 Plan Completo del Sistema

**Autor:** David Antizar  
**Fecha:** 2028-06-28  
**Versión:** 1.0 — Plan de Diseño  
**Objetivo:** Sistema de visuales en vivo para artistas con webcam, detección de cuerpo y efectos cyberpunk/espacial reactivos al movimiento y al audio.

---

## 1. 🏗️ Arquitectura General

### 1.1 Stack Tecnológico

```
┌─────────────────────────────────────────────────┐
│                   BROWSER (Chrome/Edge)          │
│                                                  │
│  ┌──────────┐   ┌──────────────┐   ┌──────────┐│
│  │  INPUT    │──▶│  DETECCIÓN   │──▶│  EFECTOS  ││
│  │          │   │              │   │          ││
│  │ • Webcam │   │ MediaPipe    │   │ Canvas   ││
│  │ • Vídeo  │   │ Pose (33 LM) │   │ 2D/WebGL ││
│  │ • URL    │   │ Hands (21 LM)│   │ p5.js    ││
│  └──────────┘   └──────────────┘   └──────────┘│
│                        │                         │
│                   ┌────▼────┐                   │
│                   │  AUDIO  │                   │
│                   │ Web Audio│                   │
│                   │ API FFT  │                   │
│                   └─────────┘                   │
│                                                  │
│  ┌──────────────────────────────────────────┐   │
│  │              UI / CONTROLES              │   │
│  │  Barra lateral · Selector efectos · BPM  │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

### 1.2 Fichero Único Autocontenido

**Un solo `index.html`** con todo el código inline:
- HTML + CSS + JavaScript
- Librerías vía CDN (MediaPipe, p5.js)
- Sin servidor, sin build, sin npm
- Se puede abrir directamente en el navegador
- Se puede desplegar en GitHub Pages, Vercel, Netlify

### 1.3 Requisitos del Sistema

| Componente | Mínimo | Recomendado |
|------------|--------|-------------|
| Navegador | Chrome 90+ / Edge 90+ | Chrome 113+ (WebGPU) |
| CPU | 2 cores | 4+ cores |
| RAM | 4 GB | 8+ GB |
| GPU | Integrada | Dedicada (para WebGL) |
| Webcam | 480p | 720p o 1080p |
| Micrófono | Opcional | Cualquier micrófono |
| Conexión | Offline (una vez cargado) | CDN para primera carga |

---

## 2. 👁️ Sistema de Detección

### 2.1 MediaPipe Pose

**Por qué MediaPipe y no YOLO:**
- YOLO: bounding box (cuadro) → solo sabe "hay una persona aquí"
- **MediaPipe: 33 puntos de referencia (landmarks)** → sabe dónde están manos, codos, hombros, cabeza, caderas, rodillas, pies
- Para efectos que siguen el movimiento del cuerpo, necesitamos landmarks, no cuadros
- Funciona a 30+ FPS en el navegador
- Modelo ligero (~12 MB)

**Landmarks del cuerpo (33 puntos):**
```
0: nariz            11: hombro izq
1: ojo izq int      12: codo izq
2: ojo izq ext      13: muñeca izq
3: ojo der int      14: meñique izq
4: ojo der ext      15: anular izq
5: oreja izq        16: medio izq
6: oreja der        17: indice izq
7: boca izq         18: pulgar izq
8: boca der         19: pulgar der
9: labio sup        20: indice der
10: labio inf       21: medio der
                    22: anular der
11-22: hombros,     23: meñique der
       codos,       24: cadera izq
       muñecas,     25: rodilla izq
       dedos        26: tobillo izq
                    27: talón izq
                    28: cadera der
                    29: rodilla der
                    30: tobillo der
                    31: talón der
                    32: talón der
```

### 2.2 MediaPipe Hands (opcional, para efectos de manos)

- 21 landmarks por mano
- Detecta gestos: puño, mano abierta, dedo índice
- Útil para: "lanzar" efectos con las manos, control gestual

### 2.3 Pipeline de Detección

```
Webcam/Vídeo (640x480 @ 30fps)
        │
        ▼
MediaPipe Pose Solution
        │
        ▼
33 landmarks (x, y, z, visibility)
        │
        ├──▶ Posición del cuerpo (centro, contorno)
        ├──▶ Velocidad de movimiento (delta entre frames)
        ├──▶ Ángulos de articulaciones (codos, rodillas)
        ├──▶ Tamaño relativo (escala del cuerpo en cámara)
        └──▶ Manos (landmarks 19-23) → gestos
                │
                ▼
        Parámetros normalizados [0-1]
                │
                ▼
        Motor de Efectos
```

### 2.4 Datos Extraídos del Cuerpo

Cada frame produce estos datos normalizados:

| Dato | Cálculo | Uso en efectos |
|------|---------|----------------|
| **centroX, centroY** | Media de todos los landmarks | Anclaje de efectos centrales |
| **silueta** | Landmarks 11→13→15→17→19→21→... forman contorno | Contorno neón, sombras |
| **manos[izq, der]** | Landmarks 19-23 (izq), 20-23 (der) | Rayos de manos, partículas |
| **velocidad** | Distancia entre frame_actual y frame_anterior | Intensidad de efectos |
| **bpm_detect** | Frecuencia de movimiento repetitivo | Sincronización visual |
| **altura** | Distancia cabeza→pies | Escala de efectos |
| **apertura** | Distancia entre manos | Apertura de portal, ondas |
| **girar** | Rotación del torso detectada | Rotación de efectos |

---

## 3. 🎨 Catálogo de Efectos

### 3.1 Categorías

Los efectos se organizan en **6 categorías**, cada una con múltiples efectos:

```
CIBERESPACIO
├── 01. Grid Distorsion
├── 02. Matrix Rain
├── 03. Circuit Board
└── 04. Data Stream

LUZ NEÓN
├── 05. Neon Tracer
├── 06. Light Streaks
├── 07. Glow Pulse
└── 08. Neon Outline

PARTÍCULAS
├── 09. Constellation
├── 10. Particle Burst
├── 11. Star Field
└── 12. Dust Storm

ONDA / ENERGÍA
├── 13. Shockwave
├── 14. Energy Field
├── 15. Sound Waves
└── 16. Pulse Ring

ESPACIO / VR
├── 17. Portal Vortex
├── 18. Warp Speed
├── 19. Galaxy Spiral
└── 20. Wormhole

GLITCH / RETRO
├── 21. Glitch Body
├── 22. VHS Static
├── 23. Synthwave Sunset
└── 24. Pixel Sort
```

### 3.2 Detalle de Efectos (los 24 principales)

---

#### **01. Grid Distorsion** 🟦
- **Categoría:** Ciberespacio
- **Visual:** Rejilla cyberpunk (líneas perspectiva) que se deforma donde está el cuerpo
- **Data input:** Landmarks del contorno del cuerpo
- **Comportamiento:** La grid se curva/bulta alrededor del artista
- **Color:** Azul neón (#00f0ff) sobre fondo oscuro
- **Audio-reactive:** Las ondulaciones siguen el beat del bajo

#### **02. Matrix Rain** 🟢
- **Categoría:** Ciberespacio
- **Visual:** Columnas de caracteres cayendo (estilo Matrix) que se "esquivan" al cuerpo
- **Data input:** Silueta del cuerpo como colisión
- **Comportamiento:** Los caracteres caen y se desvían al chocar con el artista
- **Color:** Verde Matrix (#00ff41) 
- **Audio-reactive:** Velocidad de caída = frecuencia del beat

#### **03. Circuit Board** 🔵
- **Categoría:** Ciberespacio
- **Visual:** Líneas de circuito que crecen desde las manos del artista
- **Data input:** Posición de manos (landmarks 19-23)
- **Comportamiento:** Las líneas crecen en tiempo real desde las muñecas
- **Color:** Dorado/cobre sobre azul oscuro
- **Audio-reactive:** Complejidad del circuito = volumen del audio

#### **04. Data Stream** 🟡
- **Categoría:** Ciberespacio
- **Visual:** Flujo de datos (números/binario) que circula alrededor del cuerpo
- **Data input:** Contorno del cuerpo como "tubería"
- **Comportamiento:** Los datos fluyen por el contorno como sangre digital
- **Color:** Cian brillante
- **Audio-reactive:** Velocidad del flujo = tempo

---

#### **05. Neon Tracer** 💜
- **Categoría:** Luz Neón
- **Visual:** Líneas neón que trazan el contorno del cuerpo con estela
- **Data input:** Landmarks 11→13→15→17→19→21→... (contorno)
- **Comportamiento:** Cada punto del contorno deja una estela de luz al moverse
- **Color:** Magenta (#ff00ff) / Cian (#00f0ff) / Amarillo (#ffff00)
- **Audio-reactive:** Brillo = volumen, grosor = frecuencia

#### **06. Light Streaks** ⚡
- **Categoría:** Luz Neón
- **Visual:** Rayos de luz que salen de las muñecas/hombros
- **Data input:** Landmarks de muñecas (15, 16) y hombros (11, 12)
- **Comportamiento:** Los rayos siguen la dirección del movimiento
- **Color:** Blanco azulado con estela
- **Audio-reactive:** Longitud del rayo = energía del beat

#### **07. Glow Pulse** 🟡
- **Categoría:** Luz Neón
- **Visual:** Halo de luz que pulsa alrededor de toda la silueta
- **Data input:** Todos los landmarks del cuerpo
- **Comportamiento:** El halo se expande/contruye rítmicamente
- **Color:** Gradiente neón (cian→magenta→azul)
- **Audio-reactive:** Directamente al beat (4/4)

#### **08. Neon Outline** 🔴
- **Categoría:** Luz Neón
- **Visual:** Contorno del cuerpo dibujado con neón brillante
- **Data input:** Landmarks del contorno conectados
- **Comportamiento:** El contorno "respira" con un brillo pulsante
- **Color:** Rojo neón (#ff0040)
- **Audio-reactive:** Grosor del contorno = volumen

---

#### **09. Constellation** ⭐
- **Categoría:** Partículas
- **Visual:** Estrellas que se conectan con líneas entre los landmarks del cuerpo
- **Data input:** 33 landmarks como nodos de constelación
- **Comportamiento:** Las estrellas brillan y las líneas pulsan
- **Color:** Blanco con brillo azulado
- **Audio-reactive:** Número de estrellas visibles = frecuencia

#### **10. Particle Burst** 💥
- **Categoría:** Partículas
- **Visual:** Explosión de partículas desde cada mano
- **Data input:** Posición de manos (landmarks 19-23)
- **Comportamiento:** Al cerrar/abrir mano, explosión de partículas
- **Color:** Multicolor (HSL cycling)
- **Audio-reactive:** Número de partículas = volumen

#### **11. Star Field** 🌌
- **Categoría:** Partículas
- **Visual:** Campo de estrellas que se mueven hacia el espectador
- **Data input:** Centro del cuerpo como punto de fuga
- **Comportamiento:** Las estrellas "vuelan" desde el artista hacia la cámara
- **Color:** Blanco/variado
- **Audio-reactive:** Velocidad = tempo

#### **12. Dust Storm** 🌪️
- **Categoría:** Partículas
- **Visual:** Tormenta de partículas que gira alrededor del cuerpo
- **Data input:** Contorno del cuerpo como centro de rotación
- **Comportamiento:** Las partículas orbitan el cuerpo
- **Color:** Arena/dorado
- **Audio-reactive:** Intensidad = volumen

---

#### **13. Shockwave** 💫
- **Categoría:** Onda / Energía
- **Visual:** Ondas de choque concéntricas desde el centro del cuerpo
- **Data input:** Centro del cuerpo + velocidad de movimiento
- **Comportamiento:** Al moverse rápido, se generan ondas expansivas
- **Color:** Cian/blanco
- **Audio-reactive:** Frecuencia de ondas = beat

#### **14. Energy Field** ⚡
- **Categoría:** Onda / Energía
- **Visual:** Campo de energía que rodea todo el cuerpo con rayos
- **Data input:** Silueta completa
- **Comportamiento:** El campo se intensifica con el movimiento
- **Color:** Azul eléctrico (#4060ff)
- **Audio-reactive:** Intensidad = volumen

#### **15. Sound Waves** 🔊
- **Categoría:** Onda / Energía
- **Visual:** Ondas sonoras visibles que salen de la boca/cabeza
- **Data input:** Landmarks de cabeza (0-10) + FFT del audio
- **Comportamiento:** Las ondas se expanden y su forma refleja el espectro
- **Color:** Gradiente del espectro (rojo=bass, azul=treble)
- **Audio-reactive:** Forma directa del FFT

#### **16. Pulse Ring** 🟣
- **Categoría:** Onda / Energía
- **Visual:** Anillos de pulso que se expanden desde las muñecas
- **Data input:** Manos (landmarks 19-23)
- **Comportamiento:** Al hacer gesto de "push", anillos se expanden
- **Color:** Magenta
- **Audio-reactive:** Grosor del anillo = frecuencia

---

#### **17. Portal Vortex** 🌀
- **Categoría:** Espacio / VR
- **Visual:** Vórtice/galaxia detrás del artista
- **Data input:** Centro del cuerpo como origen del vórtice
- **Comportamiento:** El vórtice gira y respira con el movimiento
- **Color:** Púrpura/azul profundo
- **Audio-reactive:** Velocidad de rotación = tempo

#### **18. Warp Speed** 🚀
- **Categoría:** Espacio / VR
- **Visual:** Estrellas que se estiran (efecto warp de Star Trek)
- **Data input:** Centro del cuerpo como punto de fuga
- **Comportamiento:** Las estrellas se estiran hacia los bordes
- **Color:** Blanco/azul claro
- **Audio-reactive:** Velocidad de warp = volumen

#### **19. Galaxy Spiral** 🌀
- **Categoría:** Espacio / VR
- **Visual:** Espiral galáctica que gira detrás del artista
- **Data input:** Centro del cuerpo + manos como puntos de atracción
- **Comportamiento:** Las estrellas de la galaxia siguen las manos
- **Color:** Azul/dorado/púrpura
- **Audio-reactive:** Rotación = tempo, brillo = volumen

#### **20. Wormhole** 🕳️
- **Categoría:** Espacio / VR
- **Visual:** Agujero de gusano que se abre detrás del artista
- **Data input:** Centro del cuerpo + apertura de manos
- **Comportamiento:** Al abrir las manos, el agujero se expande
- **Color:** Negro con bordes de luz
- **Audio-reactive:** Tamaño = volumen

---

#### **21. Glitch Body** 🔴
- **Categoría:** Glitch / Retro
- **Visual:** Fragmentos del cuerpo que se desplazan horizontalmente
- **Data input:** Imagen de la cámara + landmarks
- **Comportamiento:** La imagen se fragmenta y desplaza en franjas
- **Color:** RGB split (rojo/azul desplazados)
- **Audio-reactive:** Intensidad del glitch = frecuencias agudas

#### **22. VHS Static** 📺
- **Categoría:** Glitch / Retro
- **Visual:** Estática de VHS sobre la imagen
- **Data input:** Imagen de la cámara
- **Comportamiento:** Ruido estático que aparece/disaparece con el movimiento
- **Color:** Blanco/gris
- **Audio-reactive:** Cantidad de estática = volumen

#### **23. Synthwave Sunset** 🌅
- **Categoría:** Glitch / Retro
- **Visual:** Horizonte synthwave con grid retro y sol
- **Data input:** Centro del cuerpo como posición del sol
- **Comportamiento:** El sol se mueve con la cabeza del artista
- **Color:** Rosa/naranja/púrpura
- **Audio-reactive:** Brillo del sol = volumen

#### **24. Pixel Sort** 🎨
- **Categoría:** Glitch / Retro
- **Visual:** Pixel sorting de la imagen (efecto datamosh)
- **Data input:** Imagen de la cámara + velocidad del cuerpo
- **Comportamiento:** Los píxeles se ordenan por luminancia en dirección del movimiento
- **Color:** Colores originales distorsionados
- **Audio-reactive:** Umbral de sorting = frecuencia

---

## 4. 🖥️ Interfaz de Usuario

### 4.1 Layout Principal

```
┌──────────────────────────────────────────────────────┐
│ ⚡ BLONDE VOULAIN — VJ Processor          [⚡REC] [⚙️]│
├────────┬─────────────────────────────────────┬───────┤
│        │                                     │       │
│ PANEL  │          VISUAL OUTPUT              │ PANEL │
│ IZQ    │          (Canvas principal)         │ DER   │
│        │                                     │       │
│ Efecto │    [Aquí se renderizan los          │ Info  │
│ activo │     visuales sobre la webcam/       │ BPM   │
│        │     vídeo detectado]                │ FPS   │
│ Barra  │                                     │ Detec │
│ de     │                                     │       │
│ audio  │                                     │ Colores│
│ FFT    │                                     │       │
│        │                                     │       │
├────────┴─────────────────────────────────────┴───────┤
│  [▶️ Webcam] [📁 Vídeo] [🔗 URL]  │  01 02 03 04 ... │
│  🎤 Micrófono: [ON/OFF]          │  [Efectos grid]   │
└──────────────────────────────────────────────────────┘
```

### 4.2 Panel Izquierdo — Efecto Seleccionado
- Nombre del efecto activo
- Barra de visualización de audio (FFT) en tiempo real
- Indicador de nivel de audio (bass/mid/treble)
- Thumbnail del efecto

### 4.3 Panel Central — Visual Output
- Canvas principal a pantalla completa (opción fullscreen con F11)
- Superposición de la cámara (transparencia configurable)
- FPS counter (esquina superior derecha)
- Indicador de detección (puntos de landmarks visibles)

### 4.4 Panel Derecho — Info y Controles
- BPM detectado
- FPS actual
- Estado de detección (pose detectada / no detectada)
- Paleta de colores del efecto
- Controles de intensidad

### 4.5 Barra Inferior — Input y Selector
- Botones: Webcam / Vídeo / URL
- Toggle de micrófono
- Grid de efectos con thumbnails/emojis
- Scroll horizontal para navegar efectos
- Búsqueda rápida de efectos

### 4.6 Modo Presentación (F11)
- Panel izquierdo y derecho se ocultan
- Solo queda el canvas + barra inferior minimalista
- Los efectos se cambian con teclado (1-9, ←→)

### 4.7 Controles de Teclado

| Tecla | Acción |
|-------|--------|
| `F11` | Modo presentación (pantalla completa) |
| `Esc` | Salir de presentación |
| `1-9` | Seleccionar efecto rápido |
| `← →` | Navegar efectos |
| `Space` | Pausar/reanudar |
| `M` | Activar/desactivar micrófono |
| `C` | Mostrar/ocultar contorno del cuerpo |
| `G` | Mostrar/ocultar grid de referencia |
| `R` | Grabar clip (10 segundos) |
| `S` | Captura de pantalla |
| `+ -` | Aumentar/disminuir intensidad |
| `Tab` | Ciclar categorías |

---

## 5. 🎵 Sistema de Audio Reactivo

### 5.1 Web Audio API

```javascript
// Pipeline de audio
Micrófono → AnalyserNode → FFT (2048 bins) → 
    ├── Sub-bass (20-60 Hz) → bass
    ├── Bass (60-250 Hz) → bass  
    ├── Low-mid (250-500 Hz) → lowMid
    ├── Mid (500-2000 Hz) → mid
    ├── Upper-mid (2000-4000 Hz) → upperMid
    ├── Presence (4000-6000 Hz) → presence
    └── Brilliance (6000-20000 Hz) → brilliance
```

### 5.2 Valores Normalizados

Cada banda de frecuencia se normaliza a [0-1]:
- `bass` → controla: ondas de choque, pulso del grid, explosiones de partículas
- `mid` → controla: movimiento de partículas, rotación de efectos
- `treble` → controla: glitch, pixel sort, brillo de estrellas
- `volume` (RMS) → controla: intensidad general, escala de efectos
- `beat` (detección de beat) → controla: disparos de efectos puntuales

### 5.3 Detección de Beat

Algoritmo de detección de beat simplificado:
1. Calcular energía del frame actual (RMS)
2. Comparar con energía promedio de los últimos N frames
3. Si energía_actual > promedio * umbral → BEAT detectado
4. Estimar BPM a partir de la distancia entre beats

### 5.4 Modos de Reactividad

| Modo | Descripción |
|------|-------------|
| **Audio** | Solo reacciona al sonido (microfono) |
| **Movimiento** | Solo reacciona al movimiento del cuerpo |
| **Híbrido** | Combina audio + movimiento |
| **Manual** | Parámetros fijos, sin reactividad |
| **Beat** | Solo en beats detectados (disparos puntuales) |

---

## 6. 🔧 Arquitectura del Código

### 6.1 Estructura del Fichero

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <!-- Meta tags, título -->
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/pose@0.4/pose.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils@0.3/camera_utils.js"></script>
    <style>/* CSS inline */</style>
</head>
<body>
    <!-- UI: paneles, canvas, controles -->
    <script>
    // ============================================
    // MÓDULO 1: Configuración global
    // ============================================
    const CONFIG = { ... };
    
    // ============================================
    // MÓDULO 2: Detección (MediaPipe)
    // ============================================
    class BodyDetector { ... }
    
    // ============================================
    // MÓDULO 3: Audio (Web Audio API)
    // ============================================
    class AudioAnalyzer { ... }
    
    // ============================================
    // MÓDULO 4: Motor de efectos
    // ============================================
    class EffectEngine { ... }
    
    // ============================================
    // MÓDULO 5: Efectos individuales
    // ============================================
    class NeonTracer extends Effect { ... }
    class GridDistortion extends Effect { ... }
    // ... 24+ clases de efectos
    
    // ============================================
    // MÓDULO 6: UI y controles
    // ============================================
    class UI { ... }
    
    // ============================================
    // MÓDULO 7: Orquestador principal
    // ============================================
    class VJProcessor { ... }
    
    // Inicialización
    const app = new VJProcessor();
    </script>
</body>
</html>
```

### 6.2 Clases Principales

#### `BodyDetector`
```javascript
class BodyDetector {
    constructor(videoElement) { ... }
    
    async init() {
        // Inicializar MediaPipe Pose
    }
    
    async detect(videoFrame) {
        // Retornar landmarks normalizados
        return {
            landmarks: [...],     // 33 puntos
            center: {x, y},       // Centro del cuerpo
            silhouette: [...],     // Contorno simplificado
            hands: {              // Manos
                left: {x, y, landmarks},
                right: {x, y, landmarks}
            },
            velocity: {x, y},     // Velocidad del movimiento
            aperture: number,     // Distancia entre manos
            height: number,       // Altura del cuerpo
            angles: {...},        // Ángulos de articulaciones
        };
    }
    
    getLandmarkPosition(landmarkIndex) { ... }
    getBodyContour() { ... }
    getHandPosition(side) { ... }
}
```

#### `AudioAnalyzer`
```javascript
class AudioAnalyzer {
    constructor() { ... }
    
    async init(useMicrophone = true) {
        // Configurar AudioContext + AnalyserNode
    }
    
    update() {
        // Llamar cada frame para actualizar datos de audio
        return {
            bass: number,        // 0-1
            mid: number,         // 0-1
            treble: number,      // 0-1
            volume: number,      // 0-1 (RMS)
            beat: boolean,       // true si hay beat
            bpm: number,         // BPM estimado
            spectrum: Float32Array, // Espectro completo
        };
    }
    
    detectBeat() { ... }
    estimateBPM() { ... }
}
```

#### `EffectEngine`
```javascript
class EffectEngine {
    constructor(canvas) { ... }
    
    registerEffect(effect) { ... }
    setActiveEffect(name) { ... }
    
    render(bodyData, audioData, deltaTime) {
        // Limpiar canvas
        // Obtener datos del efecto activo
        // Renderizar efecto
        // Superponer indicadores (opcional)
    }
}
```

#### `Effect` (base class)
```javascript
class Effect {
    constructor(name, category, emoji) {
        this.name = name;
        this.category = category;
        this.emoji = emoji;
        this.params = {};     // Parámetros ajustables
        this.audioMode = 'hybrid'; // audio/movement/hybrid/manual
    }
    
    // Métodos que cada efecto debe implementar
    init(canvas) { ... }
    update(bodyData, audioData, dt) { ... }
    render(ctx, bodyData, audioData) { ... }
    getParams() { ... }       // Retornar parámetros ajustables
    setParam(key, value) { ... }
}
```

### 6.3 Rendering Strategy

Para cada frame:

```
1. Detección: bodyDetector.detect(videoFrame)
2. Audio: audioAnalyzer.update()
3. Delta time: dt = now - lastFrame
4. Actualizar efecto: activeEffect.update(bodyData, audioData, dt)
5. Renderizar: activeEffect.render(ctx, bodyData, audioData)
6. UI overlay: renderUI(ctx, audioData, fps)
7. RequestAnimationFrame → paso 1
```

### 6.4 Gestión de Colores

Cada efecto tiene una paleta propia:

```javascript
const PALETTES = {
    cyber: ['#00f0ff', '#ff00ff', '#ffff00', '#00ff41'],
    neon: ['#ff0040', '#00f0ff', '#ff00ff', '#ffff00'],
    space: ['#1a0a2e', '#4a1a6b', '#7b2fbe', '#c471f5'],
    synthwave: ['#ff6b9d', '#c44dff', '#4d79ff', '#ff4d4d'],
    matrix: ['#00ff41', '#00cc33', '#009926', '#006619'],
};
```

---

## 7. ⚡ Optimización de Rendimiento

### 7.1 Estrategias

| Estrategia | Descripción |
|------------|-------------|
| **Frame skipping** | Ejecutar MediaPipe cada 2 frames en hardware lento |
| **Resolución adaptativa** | Bajar resolución del canvas si FPS < 30 |
| **Canvas offscreen** | Usar offscreen canvas para efectos pesados |
| **requestAnimationFrame** | Sincronizado con refresh rate del monitor |
| **Object pooling** | Reusar objetos de partículas (evitar GC) |
| **WebGL para partículas** | Efectos pesados en WebGL, UI en 2D |
| **Lazy loading** | Solo cargar efectos cuando se seleccionan |

### 7.2 Targets de Rendimiento

| Métrica | Target |
|---------|--------|
| FPS mínimo | 30 fps |
| FPS objetivo | 60 fps |
| Latencia detección | < 50ms |
| Memoria máxima | 500 MB |
| Tiempo de carga inicial | < 5 segundos |

### 7.3 Adaptación al Hardware

```javascript
function detectHardware() {
    const gl = canvas.getContext('webgl2') || canvas.getContext('webgl');
    const debugInfo = gl.getExtension('WEBGL_debug_renderer_info');
    const gpu = gl.getParameter(debugInfo.UNMASKED_RENDERER_WEBGL);
    
    // Clasificar hardware
    if (gpu.includes('M1') || gpu.includes('RTX')) return 'high';
    if (gpu.includes('UHD') || gpu.includes('Intel')) return 'medium';
    return 'low';
}
```

---

## 8. 🎬 Modos de Entrada

### 8.1 Webcam (Live)
- Captura de cámara via `getUserMedia()`
- Resolución: 640x480 (default) o 1280x720
- Espejo horizontal (mirrored) por defecto
- Opción de no-espejo para grabs

### 8.2 Vídeo Subido
- Input file: `<input type="file" accept="video/*">`
- Reproducción en loop
- Barra de progreso
- Control de velocidad (0.5x - 2x)

### 8.3 URL Externa
- Input URL directa a vídeo (MP4/WebM)
- Soporte para YouTube (via embed o download)
- Streaming de vídeo remoto

### 8.4 Imagen Estática (opcional)
- Para pruebas o efectos de presentación
- Subir imagen → detectar cuerpo estático
- Efectos reaccionan a movimiento del mouse en este modo

---

## 9. 📱 Responsive Design

### 9.1 Breakpoints

| Tamaño | Layout |
|--------|--------|
| Desktop (>1024px) | 3 paneles (izq + centro + der) |
| Tablet (768-1024px) | 2 paneles (centro + panel colapsable) |
| Móvil (<768px) | Solo canvas + controles flotantes |

### 9.2 Touch Controls
- Swipe izq/der para cambiar efectos
- Tap para play/pause
- Pinch para zoom del canvas
- Long press para menú de efectos

---

## 10. 🚀 Despliegue

### 10.1 Opciones

| Método | Ventaja | Desventaja |
|--------|---------|------------|
| **GitHub Pages** | Gratis, automático | Solo HTTPS |
| **Vercel** | Gratis, rápido | Requiere cuenta |
| **Archivo local** | Sin internet | Solo en esa máquina |
| **NaN.builders** | Ya tienes infra | Más complejo |

### 10.2 Recomendación

**GitHub Pages** para la versión pública de Blonde Poulain:
- Repo: `Ntizar/blonde-vj-processor` (privado o público)
- URL: `ntizar.github.io/blonde-vj-processor/`
- Workflow: `pages.yml` con build_type: workflow

---

## 11. 📅 Roadmap de Implementación

### Fase 1: MVP (Sesión 1-2)
- [x] Plan completo
- [ ] Estructura HTML + CSS + JS
- [ ] MediaPipe Pose integrado
- [ ] Canvas de renderizado
- [ ] 3 efectos base: Neon Tracer, Grid Distortion, Particle Burst
- [ ] Input: Webcam + Vídeo
- [ ] Controles básicos

### Fase 2: Audio (Sesión 3)
- [ ] Web Audio API + FFT
- [ ] Detección de beat
- [ ] Reactividad de efectos al audio
- [ ] Modo híbrido (audio + movimiento)

### Fase 3: Catálogo Completo (Sesión 4-6)
- [ ] Efectos 04-12 (Ciberespacio + Partículas)
- [ ] Efectos 13-20 (Ondas + Espacio)
- [ ] Efectos 21-24 (Glitch)
- [ ] Paletas de colores por efecto
- [ ] Parámetros ajustables

### Fase 4: UI y UX (Sesión 7)
- [ ] Panel lateral completo
- [ ] Grid de efectos con thumbnails
- [ ] Modo presentación (F11)
- [ ] Controles de teclado
- [ ] Responsive design

### Fase 5: Polish (Sesión 8)
- [ ] Optimización de rendimiento
- [ ] Transiciones entre efectos
- [ ] Guardar/cargar configuraciones
- [ ] Exportar clips de 10s
- [ ] Test con vídeos reales de Blonde Poulain

---

## 12. 🎯 Criterios de Éxito

| Criterio | Target |
|----------|--------|
| Funciona en Chrome/Edge | ✅ |
| 30+ FPS con webcam | ✅ |
| Detección de cuerpo precisa | ✅ |
| 24+ efectos funcionales | ✅ |
| Reactividad al audio | ✅ |
| Un solo HTML autocontenido | ✅ |
| Se puede abrir con doble-click | ✅ |
| Funciona offline (una vez cargado) | ✅ |
| Estética cyberpunk coherente | ✅ |

---

## 13. 💡 Ideas Futuras (Post-v1)

- **Grabación de set completo** — exportar vídeo con visuales superpuestos
- **Modo multi-cámara** — varias fuentes de vídeo
- **MIDI mapping** — controlar efectos con controlador MIDI
- **Modo OBS** — capture window para OBS Studio
- **Templates guardables** — configuraciones predefinidas para diferentes canciones
- **Detección facial** — efectos que reaccionan a expresiones
- **Colaboración remota** — varios artistas conectados
- **IA generativa** — generar nuevos efectos con prompts

---

> *"No estás escribiendo código. Estás dirigiendo luz."*
