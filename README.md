# Letras de Nil

**https://kiwi8891.github.io/letras-nil/**

App de lectoescritura y cálculo para Nil. Tres tareas: **leer** una frase en voz alta,
**escribirla** en el papel (papá valida las dos con ✓/✗) y **mates** (sumas, restas y
multiplicaciones, que resuelve en el cuaderno y contesta en pantalla entre 4 opciones).
Todo paga monedas, y con las monedas compra Pokémon en la tienda, por cadenas de evolución.

Escribir paga más que leer, y leer más que las mates. Si un día hace muchas más mates que
frases, las mates bajan de precio hasta que vuelva a leer o escribir.

Fichero único `index.html`, sin build system.

> **Importante:** añadir a la pantalla de inicio del iPad. Safari borra el
> `localStorage` de webs normales tras ~7 días sin visitas, pero no el de una
> app añadida a la pantalla de inicio. Sin eso se pierden las monedas.

Contexto completo, decisiones de diseño y bitácora de uso en `CONTEXTO_PROYECTO.md`.
Reglas cortas para editar el código en `CLAUDE.md`.

Cada Pokémon comprado tiene su ficha: foto grande, tipos, altura, peso, la cadena de
evolución y la descripción oficial de la Pokédex en español.

Los sprites se cargan desde [PokeAPI/sprites](https://github.com/PokeAPI/sprites) y las
fichas desde [PokeAPI](https://pokeapi.co), cacheadas en el navegador tras la primera vez.
Pokémon es marca de Nintendo / Game Freak; esto es un juego personal sin ánimo de lucro.
