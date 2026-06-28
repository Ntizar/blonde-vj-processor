# ⚡ BLONDE VOULAIN — VJ Processor

**Sistema de visuales en vivo para artistas con webcam y detección de cuerpo.**

Hecho con ❤️ por David Antizar

---

## 🎭 ¿Qué es?

Un processor VJ que convierte el movimiento de un artista en visuales cyberpunk/espacial en tiempo real. Usa la webcam para detectar el cuerpo (33 puntos de referencia) y genera efectos que reaccionan al movimiento y al audio.

**Ideal para:** artistas, DJs, performers, instalaciones artísticas, conciertos.

## 🚀 Uso Rápido

1. Abre `index.html` en Chrome o Edge
2. Activa la webcam (o sube un vídeo)
3. Selecciona un efecto de la barra inferior
4. ¡Muévete! Los visuales reaccionan a tu cuerpo

**No necesita servidor** — solo doble-click en el archivo.

## 🎨 Efectos (10 incluidos)

| # | Efecto | Categoría | Emoji |
|---|--------|-----------|-------|
| 1 | Neon Tracer | Luz Neón | 💜 |
| 2 | Grid Distortion | Ciberespacio | 🟦 |
| 3 | Particle Burst | Partículas | 💥 |
| 4 | Constellation | Partículas | ⭐ |
| 5 | Shockwave | Onda / Energía | 💫 |
| 6 | Portal Vortex | Espacio / VR | 🌀 |
| 7 | Synthwave Sunset | Glitch / Retro | 🌅 |
| 8 | Energy Field | Onda / Energía | ⚡ |
| 9 | Warp Speed | Espacio / VR | 🚀 |
| 10 | Glitch Body | Glitch / Retro | 🔴 |

## ⌨️ Atajos de Teclado

| Tecla | Acción |
|-------|--------|
| `1-9` | Seleccionar efecto |
| `← →` | Navegar efectos |
| `F11` | Modo presentación |
| `Space` | Pausar / reanudar |
| `M` | Activar / desactivar micrófono |
| `C` | Mostrar landmarks del cuerpo |
| `F` | Alternar espejo webcam |
| `+ -` | Intensidad del efecto |
| `?` | Mostrar atajos |

## 🔧 Stack

- **MediaPipe Pose** — Detección de 33 landmarks del cuerpo en tiempo real
- **Web Audio API** — FFT del micrófono para reactividad al beat
- **Canvas 2D** — Renderizado de efectos a 60 FPS
- **100% en el navegador** — Sin servidor, sin npm, sin dependencias

## 🎵 Modos de Reactividad

- **Híbrido** — Combina audio + movimiento (por defecto)
- **Audio** — Solo reacciona al sonido
- **Movimiento** — Solo reacciona al cuerpo
- **Manual** — Parámetros fijos

## 📱 Requisitos

- Chrome 90+ o Edge 90+
- Webcam (para modo live)
- Micrófono (opcional, para audio-reactive)

## 📄 Licencia

MIT — Usa como quieras.

---

> *"No estás escribiendo código. Estás dirigiendo luz."*
