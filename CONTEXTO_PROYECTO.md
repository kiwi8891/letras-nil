---
name: Letras de Nil — Project Overview
description: App de lectoescritura para Nil (6 años). Lee la frase en el iPad, la escribe en papel, papá valida, gana monedas y compra Pokémon por cadenas de evolución.
type: project
---

## Proyecto: LETRAS DE NIL

**Fichero único:** `index.html` (~970 líneas, HTML + CSS + JS, sin build system).
Más `manifest.json` y tres PNG de icono.

**Why:** Nil (casi 6 años) lee y escribe **solo en mayúsculas y despacio**. En el cole ya
mezclan mayúscula y minúscula. Objetivo: **fluidez en los dos alfabetos**.

**Principio de diseño innegociable:** el lápiz está en el **papel**, no en la pantalla.
El iPad es consigna, cronómetro y recompensa. La app **nunca** intenta reconocer trazo.

**Bucle único:**
```
La app muestra una frase → Nil la lee en voz alta → la escribe en papel → papá pulsa ✓/✗ → monedas
```

**GitHub:** pendiente (`kiwi8891/letras-nil`, GitHub Pages)

---

## Decisiones del usuario (no cambiar sin preguntar)

| Tema | Decisión |
|---|---|
| Dispositivo | iPad, Safari. Escritura en papel. |
| Audio / TTS | **No.** Solo visual. |
| Validación | Papá pulsa ✓/✗ mirando el papel. |
| Contenido | **Siempre frases**, nunca palabras sueltas. |
| Progresión | **Sin fases ni niveles bloqueados.** Dificultad elegible. |
| Quién elige dificultad | **Papá**, en el panel. Nil solo pulsa JUGAR. |
| Recompensa | Monedas → tienda → Pokémon por **cadenas de evolución**. |
| Gamificación extra | **Solo racha de días** (x1,5 desde el 3er día). Nada más. |
| Temas de las frases | Familia / casa / cole y fútbol / deportes. |
| Progreso | `localStorage` + añadir a pantalla de inicio + export JSON. |

---

## Dificultad: dos selectores independientes

Los fija papá en el panel. 9 combinaciones.

| Longitud | Frases | Monedas por frase |
|---|---|---|
| `corta` | 3-4 palabras (45 frases) | 5 |
| `media` | 5-8 palabras (40 frases) | 8 |
| `larga` | 8-11 palabras (35 frases) | 12 |

| Tipo de letra | Qué se ve en pantalla |
|---|---|
| `may` | `EL BALÓN ES ROJO.` |
| `dos` | `EL BALÓN ES ROJO.` arriba en gris + `El balón es rojo.` abajo en negro |
| `min` | `El balón es rojo.` |

El modo `dos` es el puente pedagógico: ve las dos formas a la vez y escribe la de abajo.

**Las frases se guardan ya bien escritas** (tildes, mayúscula inicial, punto final).
`may` se deriva con `toUpperCase()`, que en JS conserva las tildes. Si se añaden frases
nuevas, escribirlas con ortografía correcta o Nil copiará la falta.

---

## Economía

```
frase correcta a la primera  →  5 / 8 / 12 monedas según longitud
frase acertada en el repaso  →  la mitad
sesión sin ningún fallo      →  +20
racha de 3 días o más        →  x1,5 sobre el total
```
Sesión perfecta ≈ 60 (corta) / 84 (media) / 116 (larga) monedas.

**Precios** (`priceOf`): básico con evolución 40 · primera evolución 120 ·
segunda evolución 350 · sin evolución 150 · aves legendarias 900 · Mewtwo 1200 · Mew 1500.

Una sesión compra un básico. Un Charizard son ~4 sesiones. Mewtwo, unas tres semanas.

**Regla de la tienda:** no puedes comprar una evolución sin tener la anterior
(`canBuy` comprueba `PREV[id]`). Es lo que hace que ahorre en vez de gastar a bote pronto.

---

## Estructuras de datos

| Nombre | Qué es |
|---|---|
| `BANK` | `{corta:[], media:[], larga:[]}` — 120 frases ya bien escritas |
| `POKE` | 151 nombres, índice = id-1 |
| `PREV` | `{id: id_anterior}` — de quién evoluciona cada uno |
| `CHAINS` | cadenas construidas automáticamente desde `PREV` al cargar |
| `LEGEND` | `[144,145,146,150,151]` |
| `COIN_PER` | monedas por frase según longitud |

Eevee es el único caso ramificado: `CHAINS` da `[133,134,135,136]` y la tienda pinta `·`
en vez de `→` entre hermanos (`PREV[id] === chain[i-1]`).

Sprites: `https://raw.githubusercontent.com/PokeAPI/sprites/.../official-artwork/{id}.png`.
URL determinista, sin llamadas a la API. Requiere internet la primera vez que se ve cada uno.

---

## Pantallas

`s-home` · `s-play` · `s-result` · `s-shop` · `s-dex` · `s-dad` + `#overlay` (compra).
Cambio con `go(id)`: quita `.active` de todas y la pone en una.

`s-play` es la pantalla que importa: texto enorme centrado, y abajo dos botones grandes
y **muy separados** (`gap:26px`) para que papá no falle al pulsar y Nil no le dé sin querer.
`✗ OTRA VEZ` no resta monedas: reencola la frase al final para repetirla una vez.

---

## Funciones clave

| Función | Qué hace |
|---|---|
| `startSession()` | Monta la cola: hasta 1/3 son frases ya falladas antes, resto al azar |
| `renderItem()` | Pinta la frase según `settings.case` |
| `judge(ok)` | Suma monedas o reencola; registra tiempo y fallo |
| `endSession()` | Bonus, racha, multiplicador, historial, guarda |
| `touchStreak()` / `liveStreak()` | Racha: sube una vez al día; se rompe si salta un día |
| `canBuy(id)` | Comprueba prerrequisito de evolución **y** monedas |
| `buy(id)` | Descuenta, añade a la colección, overlay + fanfare |
| `priceOf(id)` / `stageOf(id)` | Precio por etapa de evolución |
| `playVictoryFanfare()` | Web Audio API, sin dependencias (portado de JUEGO_TABLAS) |

---

## Persistencia

`localStorage` key **`letras_nil_v2`**:
`{v, coins, dex[], streak{count,last}, settings{len,case,items,solo}, history[], fails{}, times{}}`

Safari en iOS borra `localStorage` de webs normales tras ~7 días sin visitas. Una web
**añadida a la pantalla de inicio** (`display:standalone` en el manifest) queda exenta.
Por eso el manifest no es opcional: es lo que hace viable "poder retomar" sin backend.
Respaldo adicional: exportar/importar JSON desde el panel de papá.

---

## Diseño

- Paleta cálida: `--bg:#FDF6E9` crema · `--ink:#2A2119` marrón · `--red:#E03B2F` Poké Ball ·
  `--green:#2E9E5B` · `--gold:#E8A317`. Ni blanco ni negro puros.
- **Fuente Andika** para las frases: diseñada para alfabetización, con la `a` **de un solo piso**
  (la que el niño escribe a mano) y la `l` distinguible de la `I`. **Decisión pedagógica,
  no estética**: no sustituir por una fuente con `a` de doble piso. Fallback: Lexend.
- Fredoka para títulos y UI.

---

## Notas para editar

- Todo en un scope global de JS, sin módulos ni clases. Estilo `var` + `function`, como JUEGO_TABLAS.
- Si se añaden frases, respetar tildes, mayúscula inicial y punto final.
- Al probar con Playwright en local: servir con `python3 -m http.server` (el protocolo `file:` está bloqueado).
- Precedente y patrones: `15_PROYECTOS/JUEGO_TABLAS/index.html` (mismo niño, misma arquitectura).
