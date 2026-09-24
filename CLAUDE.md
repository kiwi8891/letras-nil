# LETRAS DE NIL — instrucciones del proyecto

App de lectoescritura para Nil (6 años). Contexto completo, estructuras de datos y
bitácora de uso en **`CONTEXTO_PROYECTO.md`**: leerlo antes de tocar nada.

**Live:** https://kiwi8891.github.io/letras-nil/ · repo `kiwi8891/letras-nil` (Pages desde `main`)

## Reglas innegociables

1. **El lápiz está en el papel, no en la pantalla.** La app nunca intenta reconocer trazo,
   ni pide escribir con el dedo, ni añade teclado. El iPad es consigna y recompensa.
   En mates la cuenta también se hace en el cuaderno: la pantalla solo recoge la respuesta.
2. **Sin audio ni TTS.** Decisión explícita de Ger. Nada de Web Speech API.
3. **Siempre frases, nunca palabras sueltas.**
4. **Sin fases ni niveles bloqueados.** La dificultad la **fija papá** en su panel, y va
   **por tarea**: leer, escribir y mates tienen selectores propios. Nil solo elige tarea.
5. **La recompensa es una tienda, no un gacha.** Monedas → comprar el Pokémon que él elija,
   por cadenas de evolución: no se puede comprar una evolución sin tener la anterior, y cada
   etapa cuesta mucho más. Eso es lo que le hace ahorrar. No convertirlo en cajas sorpresa.
6. **Gamificación: solo la racha de días.** Ger descartó equipo de 6, récord de velocidad y
   otros añadidos. No reintroducirlos sin que los pida.
7. **Fichero único `index.html`.** No modularizar, no meter build system, no añadir framework.
8. **Fuente `Comic Neue` para las frases y las tablas.** La interfaz va en Fredoka.
   Tres criterios, por orden: `a` **de un solo piso** (la que escribe a mano), **`I` con
   travesaños** para que no se confunda con la `l` ni con el `1`, y **peso 700 real**.
   Ya descartadas por Ger: **Andika** y **Edu AU VIC WA NT Pre** (esta sale inclinada y con
   las letras pegadas). Descartadas por criterio: Quicksand y Nunito (`l` e `I` idénticas),
   Delius (solo pesa 400, el negrita saldría sintético), Schoolbell (demasiado irregular).
   **Al cambiar de fuente, comprobar dos cosas antes de nada:**
   a) **Glifos del español uno a uno en navegador.** `Edu NSW ACT Foundation` y las
      `Edu … Beginner` declaran cubrir Latin-1 y **no traen tildes ni ñ**; Safari cambia de
      fuente a mitad de palabra sin avisar. Método: `measureText` del glifo con la fuente y
      sin ella; si coinciden, falta. **Control obligatorio con un carácter que falte seguro**
      (`Ж`), o el test da falsos negativos. Comparar píxeles de canvas NO sirve.
   b) **Mirarla renderizada**, no fiarse del nombre ni de la descripción. Montar una página
      con frases reales del banco y hacer screenshot.
9. **Solo dos niveles de letra: fácil MAYÚSCULA, difícil minúscula.** El viejo modo puente
   "las dos" está eliminado (2026-09-16). `CASE_MULT` y `CASE_LABEL` conservan la clave `dos`
   solo para no romper historiales antiguos, y `normalizeCase()` reescribe a `min` cualquier
   ajuste guardado con ella. No reintroducir el modo sin que Ger lo pida.
10. **Tres tareas: leer, escribir y mates.** Las mates se responden en pantalla con 4
   opciones (decisión explícita de Ger). Sumas y restas **siempre llevando**, hasta 5 cifras,
   nunca resultado negativo. Tablas de multiplicar del 1 al 10.
11. **Jerarquía de premio innegociable:** a igual nivel, **escribir > leer > mates**, aun
   comparando el mejor modificador de mates contra el peor de las letras. Las letras son el
   objetivo del curso. Si tocas `COIN`, `CASE_MULT` u `OP_MULT`, vuelve a comprobarlo.
12. **Leer y escribir en minúscula se premian** (`CASE_MULT`: may x1, min x1,4), y
   **restar llevando y multiplicar** más que sumar (`OP_MULT`: x1, x1,25, x1,4).
13. **Tope diario de mates.** Si en un día acumula 12 operaciones más que frases, las mates
   pagan la mitad; desde 28, un cuarto. Se recupera leyendo o escribiendo. No quitarlo: es
   lo que impide que se salte la lectura a base de sumas.

14. **La colección son cartas oficiales del JCC en inglés** (2026-09-24), del set
   **Scarlet & Violet "151"** (`sv3pt5`), el único que tiene los 151 con estilo uniforme.
   En ese set **nº de carta = nº de Pokédex**: `cardUrl(id, big)` →
   `images.pokemontcg.io/sv3pt5/{id}.png` (245 px) o `{id}_hires.png` (734 px). Los 3, 6 y 9
   son las cartas `ex` (es lo que trae el set con ese número). No mezclar sets.
   Toda la estética es de carta: mesa azul del reverso, bloques con marco amarillo, tareas con
   su energía (leer Agua, escribir Fuego, mates Planta). La frase y la operación van sobre una
   carta grande de cara crema: **fondo claro y quieto para leer**, sin brillos detrás.
   Zoom propio en el overlay (el viewport lleva `user-scalable=no`): tocar = x2,5 donde toca,
   pellizcar hasta x4, arrastrar mueve con zoom o inclina con brillo holo sin zoom.

## Al añadir frases al banco

Escribirlas **ya bien escritas**: tilde, mayúscula inicial y punto final. El modo
MAYÚSCULAS se deriva con `toUpperCase()`. Si se cuela una frase sin tilde, Nil copia la falta.

Respetar las longitudes: `corta` 3-4 palabras · `media` 5-8 · `larga` 8-11.
Temas: familia, casa, cole y fútbol/deportes.

## Al probar

`file://` está bloqueado en Playwright: servir con `python3 -m http.server 8777` y abrir
`http://localhost:8777/index.html`.

Si cambia la forma del estado guardado, **subir la versión de la key** (`letras_nil_v3` →
`v4`) **y** escribir la migración en `migrate()`. Nil pierde monedas y Pokémon si se rompe.
`migrate()` corre en dos sitios: al cargar (si solo existe la key vieja) y al **importar un
JSON** exportado por una versión anterior. Los dos caminos tienen que pasar por ella.

La caché de PokeAPI vive aparte, en `letras_nil_pokecache`. Es regenerable y no entra en el
export: borrarla no pierde nada.

## Al cambiar la economía

La tabla del panel de papá (`tarifTable`) es la única fuente de verdad que ve Ger para
decidir el nivel del día: monedas por acierto, total de una sesión clavada y el saldo al que
llegaría, cruzado con lo que ese saldo le abre en la tienda. Si añades una tarea o un
modificador, tiene que aparecer ahí.
