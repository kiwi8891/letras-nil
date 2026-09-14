# LETRAS DE NIL — instrucciones del proyecto

App de lectoescritura para Nil (6 años). Contexto completo, estructuras de datos y
bitácora de uso en **`CONTEXTO_PROYECTO.md`**: leerlo antes de tocar nada.

**Live:** https://kiwi8891.github.io/letras-nil/ · repo `kiwi8891/letras-nil` (Pages desde `main`)

## Reglas innegociables

1. **El lápiz está en el papel, no en la pantalla.** La app nunca intenta reconocer trazo,
   ni pide escribir con el dedo, ni añade teclado. El iPad es consigna y recompensa.
2. **Sin audio ni TTS.** Decisión explícita de Ger. Nada de Web Speech API.
3. **Siempre frases, nunca palabras sueltas.**
4. **Sin fases ni niveles bloqueados.** La dificultad son dos selectores independientes
   (longitud × tipo de letra) que **fija papá** en su panel. Nil solo pulsa JUGAR.
5. **La recompensa es una tienda, no un gacha.** Monedas → comprar el Pokémon que él elija,
   por cadenas de evolución: no se puede comprar una evolución sin tener la anterior, y cada
   etapa cuesta mucho más. Eso es lo que le hace ahorrar. No convertirlo en cajas sorpresa.
6. **Gamificación: solo la racha de días.** Ger descartó equipo de 6, récord de velocidad y
   otros añadidos. No reintroducirlos sin que los pida.
7. **Fichero único `index.html`.** No modularizar, no meter build system, no añadir framework.
8. **Fuente Andika para las frases.** Es criterio pedagógico (`a` de un solo piso, como la
   escribe a mano), no estético. No sustituir por Inter/Arial/system-ui.

## Al añadir frases al banco

Escribirlas **ya bien escritas**: tilde, mayúscula inicial y punto final. El modo
MAYÚSCULAS se deriva con `toUpperCase()`. Si se cuela una frase sin tilde, Nil copia la falta.

Respetar las longitudes: `corta` 3-4 palabras · `media` 5-8 · `larga` 8-11.
Temas: familia, casa, cole y fútbol/deportes.

## Al probar

`file://` está bloqueado en Playwright: servir con `python3 -m http.server 8777` y abrir
`http://localhost:8777/index.html`.

Si cambia la forma del estado guardado, **subir la versión de la key** (`letras_nil_v2` →
`v3`) o escribir migración en `load()`. Nil pierde monedas y Pokémon si se rompe.
