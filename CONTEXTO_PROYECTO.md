---
name: Letras de Nil — Project Overview
description: App de lectoescritura para Nil (6 años). Lee la frase en el iPad, la escribe en papel, papá valida, gana monedas y compra Pokémon por cadenas de evolución.
type: project
---

## Proyecto: LETRAS DE NIL

**Fichero único:** `index.html` (~1720 líneas, HTML + CSS + JS, sin build system).
Más `manifest.json` y tres PNG de icono.

**Why:** Nil (casi 6 años) lee y escribe **solo en mayúsculas y despacio**. En el cole ya
mezclan mayúscula y minúscula. Objetivo: **fluidez en los dos alfabetos**.

**Principio de diseño innegociable:** el lápiz está en el **papel**, no en la pantalla.
El iPad es consigna, cronómetro y recompensa. La app **nunca** intenta reconocer trazo.

**Tres tareas, un solo bucle de recompensa:**
```
LEER      frase en pantalla → Nil la lee en voz alta      → papá pulsa ✓/✗ → monedas
ESCRIBIR  frase en pantalla → Nil la copia en el papel    → papá pulsa ✓/✗ → monedas
MATES     operación en pantalla → Nil la resuelve en el cuaderno → toca el resultado (4 opciones) → monedas
```
También en mates el lápiz sigue en el papel: la pantalla solo recoge la respuesta, la cuenta
se hace en el cuaderno. Es la única tarea que Nil hace sin papá delante.

**Live:** https://kiwi8891.github.io/letras-nil/
**GitHub:** `kiwi8891/letras-nil` (público, Pages desde `main`)
**Reglas cortas para editar:** `CLAUDE.md` del proyecto.

---

## Decisiones del usuario (no cambiar sin preguntar)

| Tema | Decisión |
|---|---|
| Dispositivo | iPad, Safari. Escritura en papel. |
| Audio / TTS | **No.** Solo visual. |
| Validación | Papá pulsa ✓/✗ mirando el papel. |
| Contenido | **Siempre frases**, nunca palabras sueltas. |
| Progresión | **Sin fases ni niveles bloqueados.** Dificultad elegible. |
| Quién elige dificultad | **Papá**, en el panel, y **por tarea**. Nil solo elige qué tarea hacer. |
| Mates | 4 opciones en pantalla (decisión de Ger, 2026-09-14). Sumas y restas **siempre llevando**, hasta 5 cifras. Tablas del 1 al 10. |
| Jerarquía de premio | A igual nivel: **escribir > leer > mates**. Las letras son el objetivo, las mates el complemento. |
| Recompensa | Monedas → tienda → Pokémon por **cadenas de evolución**. |
| Gamificación extra | **Solo racha de días** (x1,5 desde el 3er día). Nada más. |
| Temas de las frases | Familia / casa / cole y fútbol / deportes. |
| Progreso | `localStorage` + añadir a pantalla de inicio + export JSON. |

---

## Dificultad: por tarea, la fija papá

Cada tarea lleva sus propios selectores en el panel. Nil solo elige tarea.

**Leer y escribir:** longitud × tipo de letra × frases por sesión (5/8/12).

| Longitud | Frases |
|---|---|
| `corta` | 3-4 palabras (45 frases) |
| `media` | 5-8 palabras (40 frases) |
| `larga` | 8-11 palabras (35 frases) |

| Tipo de letra | Qué se ve en pantalla |
|---|---|
| `may` | `EL BALÓN ES ROJO.` |
| `dos` | `EL BALÓN ES ROJO.` arriba en gris + `El balón es rojo.` abajo en negro |
| `min` | `El balón es rojo.` |

El modo `dos` es el puente pedagógico: ve las dos formas a la vez y escribe la de abajo.

**Las frases se guardan ya bien escritas** (tildes, mayúscula inicial, punto final).
`may` se deriva con `toUpperCase()`, que en JS conserva las tildes. Si se añaden frases
nuevas, escribirlas con ortografía correcta o Nil copiará la falta.

**Mates:** nivel × operaciones por sesión (6/10/15) × qué tablas entran.

| Nivel | Sumas y restas | Multiplicaciones |
|---|---|---|
| `facil` | 2 cifras, con llevada | tablas seleccionadas × 1-10 |
| `medio` | 3-4 cifras, con llevada | 2 cifras × 1 cifra |
| `dificil` | 5 cifras, con llevada | 3 cifras × 1 cifra |

`makeMath()` **regenera hasta que hay llevada** (`hasCarry` / `hasBorrow`) en los tres
niveles: sumar sin llevar Nil ya lo domina. Las restas nunca dan resultado negativo.

**Los distractores no son números al azar.** `distractors()` genera los fallos típicos de
llevada (±1, ±10, ±100, ±1000…) y **prioriza los que tienen las mismas cifras que el
resultado**: un `107` frente a un `7` se descarta de un vistazo y la pregunta deja de medir
nada. En multiplicaciones son además errores de tabla (`a×(b±1)`, `res±a`).

---

## Economía

Monedas por acierto **a la primera**, antes de modificadores (`COIN`):

| | fácil / corta | medio / media | difícil / larga |
|---|---|---|---|
| **Leer** | 4 | 5 | 7 |
| **Escribir** | 5 | 8 | 12 |
| **Mates** | 2 | 3 | 4 |

Modificadores multiplicativos, redondeando:

```
tipo de letra (leer y escribir)   may x1   ·  dos x1,2  ·  min x1,4
operación (mates)                 sum x1   ·  res x1,25 ·  mul x1,4
acierto en el repaso              la mitad
sesión sin ningún fallo           +20
racha de 3 días o más             x1,5 sobre el total de la sesión
```

**Invariante que hay que mantener:** a igual nivel, `escribir > leer > mates` incluso
comparando el mejor modificador de mates contra el peor de las letras. Está verificado en
navegador; si se tocan las bases, volver a comprobarlo.

### Tope diario de las mates

Si en un día hace **12 operaciones más que frases**, las mates pasan a pagar la **mitad**;
a partir de **28**, un **cuarto**. Se recupera en cuanto vuelve a leer o escribir.
Evita que se salte la lectura a base de sumas, que es lo fácil y lo que menos le cuesta.

`state.day = {d, leer, escribir, mates}` cuenta los ítems del día y se resetea solo al
cambiar de fecha. `mathDamp()` devuelve 1 / 0,5 / 0,25 y `dampLabel()` el aviso que se
pinta en el botón de MATES y en el menú de mates. **La tarifa se congela al empezar la
sesión** (`M.rate`): si el tope salta a media sesión, no se le cambia el trato a mitad.

**Precios** (`priceOf`): básico con evolución 40 · primera evolución 120 ·
segunda evolución 350 · sin evolución 150 · aves legendarias 900 · Mewtwo 1200 · Mew 1500.

**Regla de la tienda:** no puedes comprar una evolución sin tener la anterior
(`canBuy` comprueba `PREV[id]`). Es lo que hace que ahorre en vez de gastar a bote pronto.

### Tabla de tarifas del panel de papá

`tarifTable()` pinta, para cada tarea y nivel: monedas por acierto, total de una sesión
clavada entera (todo a la primera + bonus + racha) y **el saldo al que llegaría**, con el
Pokémon más caro que ese saldo le abriría y hoy todavía no puede pagar (`unlockHint`).
Es la pantalla para decidir "hoy toca escribir largo porque le falta poco para Charizard".
Mates lleva una fila por operación porque la tarifa cambia con ella.

---

## Estructuras de datos

| Nombre | Qué es |
|---|---|
| `BANK` | `{corta:[], media:[], larga:[]}` — 120 frases ya bien escritas |
| `POKE` | 151 nombres, índice = id-1 |
| `PREV` | `{id: id_anterior}` — de quién evoluciona cada uno |
| `CHAINS` | cadenas construidas automáticamente desde `PREV` al cargar |
| `LEGEND` | `[144,145,146,150,151]` |
| `COIN` | base por tarea y nivel |
| `CASE_MULT` / `OP_MULT` | modificadores de tipo de letra y de operación |
| `MRANGE` | rango de cifras por nivel de mates |
| `TYPE_ES` | los 18 tipos Pokémon en español con su color |

Eevee es el único caso ramificado: `CHAINS` da `[133,134,135,136]` y la tienda pinta `·`
en vez de `→` entre hermanos (`PREV[id] === chain[i-1]`).

Sprites: `https://raw.githubusercontent.com/PokeAPI/sprites/.../official-artwork/{id}.png`.
URL determinista, sin llamadas a la API. Requiere internet la primera vez que se ve cada uno.

---

## Pantallas

`s-home` · `s-play` · `s-mathmenu` · `s-math` · `s-result` · `s-shop` · `s-dex` ·
`s-ficha` · `s-dad` + `#overlay` (compra y zoom de sprite).
Cambio con `go(id)`: quita `.active` de todas y la pone en una.

`s-home` ya no tiene un botón JUGAR: tiene tres botones de tarea que **dicen lo que paga
cada una** con el nivel puesto. Nil ve la diferencia entre leer y escribir sin preguntar.

`s-ficha` es la ficha de la Pokédex: sprite grande (y a pantalla completa al tocarlo),
nombre y descripción oficiales **en español**, tipos, altura, peso y la cadena de evolución
con los que aún no tiene en gris. Datos de **PokeAPI**, cacheados en `letras_nil_pokecache`
(clave aparte del estado, para no engordar la copia de seguridad). La primera vez hace falta
red; después no. Si falla, la ficha se queda con lo que ya sabe la app y lo dice.

`s-play` es la pantalla que importa: texto enorme centrado, y abajo dos botones grandes
y **muy separados** (`gap:26px`) para que papá no falle al pulsar y Nil no le dé sin querer.
`✗ OTRA VEZ` no resta monedas: reencola la frase al final para repetirla una vez.

---

## Funciones clave

| Función | Qué hace |
|---|---|
| `startSession(task)` | Monta la cola: hasta 1/3 son frases ya falladas antes, resto al azar |
| `renderItem()` | Pinta la frase según `settings.case` |
| `judge(ok)` | Suma monedas o reencola; registra tiempo y fallo |
| `endSession()` / `endMath()` | Cierran su sesión y delegan en `finishRun()` |
| `finishRun(r)` | Cierre común: bonus, racha, contador del día, historial, resultado |
| `makeMath(op,lvl)` / `distractors(q)` | Generan la operación (siempre llevando) y los fallos plausibles |
| `answer(btn,v)` | Corrige, pinta verde/rojo, reencola el fallo y avanza solo |
| `mathDamp()` | Tope diario de las mates: 1 / 0,5 / 0,25 |
| `showFicha(id)` | Ficha de la Pokédex; `fetchPoke` trae los datos de PokeAPI y los cachea |
| `tarifTable()` | Tabla de monedas por nivel y proyección de saldo |
| `touchStreak()` / `liveStreak()` | Racha: sube una vez al día; se rompe si salta un día |
| `canBuy(id)` | Comprueba prerrequisito de evolución **y** monedas |
| `buy(id)` | Descuenta, añade a la colección, overlay + fanfare |
| `priceOf(id)` / `stageOf(id)` | Precio por etapa de evolución |
| `playVictoryFanfare()` | Web Audio API, sin dependencias (portado de JUEGO_TABLAS) |

---

## Persistencia

`localStorage` key **`letras_nil_v3`**:
```
{v:3, coins, dex[], streak{count,last},
 settings:{ leer:{len,case,items}, escribir:{len,case,items},
            mates:{lvl,items,tablas[]}, solo },
 day:{d,leer,escribir,mates}, history[], fails{}, mfails{}, times{}}
```
`times` va por `"tarea:nivel"` (`escribir:corta`). `fails` son frases, `mfails` operaciones.

**Migración v2 → v3** (`migrate()`): la dificultad única de v2 se copia a leer y a escribir,
mates arranca en sus valores por defecto, `times` se reindexa a `escribir:*` y las sesiones
antiguas se marcan `task:"escribir"`. Corre tanto al cargar (si solo existe `letras_nil_v2`)
como al **importar un JSON exportado por la versión anterior**, que avisa en el `alert`.
Monedas, Pokémon y racha se conservan intactos. Verificado en navegador.

Caché aparte: **`letras_nil_pokecache`** con las fichas de PokeAPI. Es regenerable, no entra
en el export y se puede borrar sin perder nada.

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

---

## Estado

| Fecha | Qué |
|---|---|
| 2026-09-14 | v1 publicada. Verificado en el dominio real: Andika carga, sprites de PokeAPI cargan, `localStorage` disponible, 120 frases servidas. |
| 2026-09-14 | v2: tareas separadas (leer / escribir / mates), tarifa por tarea con modificadores, tope diario de mates, módulo de mates con 4 opciones, ficha de Pokédex con PokeAPI, tabla de tarifas y proyección de saldo en el panel de papá. Estado a `letras_nil_v3` con migración. |

**Pendiente:** probarlo con Nil delante y añadirlo a la pantalla de inicio de su iPad.
Si Ger importa el JSON de las primeras sesiones, la migración lo cuenta como ESCRIBIR.

---

## Bitácora de uso con Nil

Aquí se anota lo que Ger reporte después de cada sesión real: qué le cuesta, qué le aburre,
si la economía de monedas está bien calibrada, si las frases se le quedan cortas o largas.
**Es la fuente de verdad para decidir cambios** — por encima de cualquier intuición de diseño.

| Fecha | Observación | Qué se cambió |
|---|---|---|
| _(pendiente de la primera sesión con Nil)_ | | |

### Palancas de ajuste, por orden de preferencia

1. **Dificultad** (panel de papá): longitud y tipo de letra. Primera respuesta a casi todo.
2. **Nº de frases por sesión** (5 / 8 / 12): si se cansa o se queda con ganas.
3. **Banco de frases**: añadir más, o de otros temas que le tiren.
4. **Monedas por acierto** (`COIN`, `CASE_MULT`, `OP_MULT`): solo si la motivación se cae de
   verdad. Subir la dificultad ya paga más por sí solo. Si se tocan, volver a comprobar el
   invariante `escribir > leer > mates` a igual nivel.
4b. **Tope diario de mates** (`MATH_FREE` 12 / `MATH_HARD` 28): subirlo si castiga de más,
   bajarlo si sigue esquivando la lectura.
5. **Precios** (`priceOf`): lo último. Bajarlos devalúa lo que ya tiene comprado.
