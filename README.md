# Scroll Curtain — Pinned Scroll Transitions with GSAP + Astro

Demo técnico de un patrón de landing page tipo "cinematic scroll" (el mismo estilo usado en sitios de gacha games como Azur Promilia o Genshin Impact): secciones que se fijan en pantalla mientras el scroll controla una transición de cortina/neblina, más un selector de personajes tipo carrusel circular, inspirado en el selector de skins del cliente de League of Legends.

> Proyecto de práctica / fan demo. No afiliado a Riot Games. Todos los assets de League of Legends pertenecen a Riot Games; se usan aquí solo con fines demostrativos y no comerciales.

**[Ver demo en vivo →](  )**

---

## Qué hace

- **Hero de pantalla completa** con imagen optimizada (`astro:assets`) y overlay para legibilidad de texto.
- **Secciones "pineadas"**: el scroll no mueve la página normalmente — fija la sección en pantalla y usa la posición del scroll para controlar una animación de cortina que revela el contenido de abajo.
- **Selector de personajes tipo carrusel**: ventana deslizante de 3 elementos visibles, con posicionamiento calculado mediante distancia circular (soporta N elementos, dando la vuelta en ambas direcciones sin duplicar datos).
- **Panel de detalle animado**: cambia de contenido (imagen de fondo, personaje recortado, texto) con transiciones de entrada/salida independientes por elemento.
- **Arquitectura basada en componentes reutilizables** con props y slots, sin depender de un framework de UI (React/Vue) — todo el estado corre en JS vanilla sobre Astro.

## Stack

- **[Astro](https://astro.build)** — generación estática, cero JS por defecto, islas de interactividad solo donde hacen falta.
- **[GSAP](https://gsap.com) + ScrollTrigger** — control de animación atada al scroll (`pin`, `scrub`) y timelines coordinados.
- **TypeScript** — tipado en los scripts de cliente.

## Por qué Astro en vez de React/Vue

El proyecto es ~90% contenido presentacional (scroll storytelling) y ~10% interactividad puntual (el carrusel). Ese perfil encaja mejor con el modelo de islas de Astro que con una SPA completa: HTML real desde el server, cero overhead de framework en el cliente, y GSAP corriendo directo sobre el DOM sin pelear contra el ciclo de vida de componentes (`useEffect`, refs, etc.).

## Técnicas destacadas

### 1. Pin + scrub como mecanismo de transición
Cada sección usa un único `ScrollTrigger` con `pin: true` y un timeline `scrub`-eado. El scroll del usuario no dispara la animación — la **controla directamente**, como el scrubber de un reproductor de video. El "corte" entre escenas ocurre en el punto de máxima cobertura del telón, donde se cambia el contenido de forma instantánea (`gsap.set`) mientras está oculto, evitando cualquier salto visible.

### 2. Distancia circular para el carrusel
En vez de posicionar los botones del carrusel a mano, cada uno calcula su offset como `(índice - índiceActivo)`, corregido para tomar siempre el camino más corto alrededor del array (como las horas de un reloj). Esto permite cualquier cantidad de campeones sin lógica especial en los extremos.

### 3. Ventana deslizante (windowing)
Solo se mantienen 3 botones visibles en todo momento (anterior / activo / siguiente), independientemente de cuántos campeones tenga el dataset. Los que entran/salen de la ventana lo hacen con fade + scale, y se les desactiva `pointer-events` mientras están ocultos para que no intercepten clicks fantasma.

### 4. Componentización con slots
`PinnedSection.astro` no sabe nada sobre su contenido — solo maneja pin + telón, y expone un `<slot />` genérico. `ChampionSelector.astro` vive anidado adentro sin acoplarse a la lógica de scroll. Los ids internos se generan dinámicamente (`` `curtainTop-${sectionID}` ``) para permitir múltiples instancias en la misma página sin colisiones.

## Estructura del proyecto

```
src/
├── components/
│   ├── PinnedSection.astro      # pin + telón, agnóstico del contenido
│   └── ChampionSelector.astro   # carrusel + panel de detalle
├── data/
│   └── champions.js             # dataset de campeones
├── pages/
│   └── index.astro
public/
└── champions/                   # thumbnails, banners, recortes de personaje
```

## Correr el proyecto localmente

```bash
npm install
npm run dev
```

Abrí `http://localhost:4321`.

## Aprendizajes / decisiones que valieron la pena documentar

- `querySelector` (a diferencia de `getElementById`) no garantiza unicidad — un componente duplicado en la página puede hacer que una animación le pegue al elemento equivocado sin ningún error en consola.
- `gsap.set()` no anima nada, ni respeta `duration` — es fácil confundirlo con `gsap.to()` al copiar/pegar código entre fases de un timeline.
- Comparar solo el signo de una distancia no alcanza para detectar un "wrap" real en un carrusel circular — hace falta comparar la magnitud del salto entre el estado anterior y el nuevo.

## Créditos

Arte y nombres de campeones © Riot Games. Técnica de scroll inspirada en el sitio oficial de [Azur Promilia](https://azurpromilia.jimi-global.com).