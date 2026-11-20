# Ejercicio: la tienda se mueve con GSAP

**Maquetación Web con Hojas de Estilo · Tema 2 · GSAP básico**

La web de **La Aguja y el Tornillo** ya está maquetada. Tu trabajo es animarla con **GSAP**, todo en `js/main.js`:

- **Tarea 1**: tu primer tween (el tornillo del logo gira).
- **Tarea 2**: animar varios elementos escalonados con `stagger`.
- **Tarea 3**: una intro de portada con una **timeline**.
- **Tarea 4**: controlar esa timeline con botones.
- **Tarea 5**: un menú a pantalla completa que se abre y se cierra con una timeline.

El HTML y el CSS ya están hechos: no hace falta tocarlos, salvo para enlazar GSAP.

---

## Paso 0 · Enlazar GSAP

En `index.html`, en el hueco marcado al final del `<body>`, **antes** de `js/main.js`:

```html
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/gsap.min.js"></script>
```

Comprueba en la consola (`F12`) que escribir `gsap` no da error.

---

## Lo básico de GSAP

```js
gsap.to(".caja",     { x: 200, duration: 1 });           // de donde está, A estos valores
gsap.from(".caja",   { autoAlpha: 0, y: 40 });            // DESDE estos valores, a donde está
gsap.fromTo(".caja", { x: 0 }, { x: 200, duration: 1 });  // de aquí a allí, los dos explícitos
```

Las propiedades que más vas a usar:

| Propiedad | Qué hace |
|---|---|
| `x`, `y` | mover en píxeles (o `xPercent`, `yPercent` en % del propio elemento) |
| `rotation`, `scale` | girar (grados) y escalar |
| `autoAlpha` | opacidad + `visibility`: con 0 el elemento además deja de recibir clics |
| `duration`, `delay` | segundos (por defecto, `duration: 0.5`) |
| `ease` | la curva: `"power3.out"`, `"back.out(1.7)"`, `"none"`... |
| `repeat`, `yoyo` | repetir (`-1` = infinito) e ir y volver |
| `stagger` | segundos entre un elemento y el siguiente cuando animas varios |

Prueba las curvas en el [visualizador de easing](https://gsap.com/docs/v3/Eases/).

---

## Tarea 1 · El tornillo gira

El icono del logo (`.tornillo`) da vueltas sin parar a velocidad constante.

- `gsap.to()` con `rotation: 360`, `duration: 6`, `ease: "none"` y `repeat: -1`.
- Prueba a quitar `ease: "none"`: ¿qué pasa al final de cada vuelta?

## Tarea 2 · Colocar el escaparate

Al pulsar **Colocar el escaparate** (`#colocar`), las seis `.tarjeta` entran una detrás de otra: desde abajo (`y: 60`), transparentes y un poco giradas, hasta su sitio.

- Usa `gsap.fromTo()` con `stagger: 0.1` y `ease: "back.out(1.4)"`.
- **¿Por qué `fromTo` y no `from`?** Pulsa el botón dos veces seguidas con `from()` y mira qué pasa: `from()` anima hasta donde está el elemento *en ese momento*, y si está a medias, se queda a medias.
- **Bonus**: un giro distinto para cada tarjeta con `rotation: () => gsap.utils.random(-6, 6)`.

## Tarea 3 · La intro de la portada

Al cargar la página, la portada aparece por partes. Crea una timeline:

```js
const intro = gsap.timeline({
    defaults: { duration: 0.8, ease: "power3.out" }
});

intro
    .from(".cabecera", { yPercent: -100 })
    .from(".portada-antetitulo", { autoAlpha: 0, y: 20 }, "-=0.4")
    // ...sigue tú
```

Pasos (en este orden): `.cabecera` baja desde arriba, aparece `.portada-antetitulo`, las tres `.linea-texto` del título suben desde abajo (`yPercent: 110`, con `stagger`), aparecen `.portada-texto` y los botones `.portada-botones a`. Al mismo tiempo, la foto `.portada-foto` se descubre de abajo arriba y la imagen hace un pequeño zoom hacia atrás:

```js
.fromTo(".portada-foto",
    { clipPath: "inset(100% 0% 0% 0%)" },
    { clipPath: "inset(0% 0% 0% 0%)", duration: 1.2, ease: "power4.inOut" },
    0.3)
.from(".portada-foto img", { scale: 1.3, duration: 1.6 }, "<")
```

El tercer parámetro (la **posición**) decide cuándo empieza cada paso:

| Posición | Empieza... |
|---|---|
| *(nada)* | cuando acaba el paso anterior |
| `"-=0.4"` | 0,4 s antes de que acabe el anterior (se solapan) |
| `"+=0.5"` | 0,5 s después de que acabe el anterior (una pausa) |
| `"<"` | a la vez que empieza el anterior |
| `"<0.1"` | 0,1 s después de que empiece el anterior |
| `0.3` | en el segundo 0,3 de la timeline |

Las líneas del título suben "desde dentro" porque cada `.linea` tiene `overflow: clip` en el CSS: el texto está escondido debajo y aparece al subir.

## Tarea 4 · El mando a distancia

Haz que los botones de la sección **El mando a distancia** controlen la timeline `intro`:

- Cada `.boton-mando` tiene su acción en `data-accion`: `play`, `pause`, `reverse`, `restart`, `lento` y `normal`.
- `lento` → `intro.timeScale(0.25)` y `normal` → `intro.timeScale(1)`.
- El resto se puede hacer en una línea: `intro[accion]()` llama al método con ese nombre.
- **La barra** `#progreso`: al moverla (evento `input`), pausa la intro y llévala a ese punto con `intro.progress(valor)`.
- **Bonus**: que la barra se mueva sola mientras la intro avanza. Añade `onUpdate` a la timeline y, dentro, actualiza el valor de la barra y el `<output id="valor-progreso">`.

## Tarea 5 · El menú

El botón **Menú** (`#abrir-menu`) abre `#menu` a pantalla completa y **Cerrar** (`#cerrar-menu`) lo cierra.

1. Crea una timeline **en pausa**: `gsap.timeline({ paused: true })`.
2. Pasos: mostrar el menú (`.set("#menu", { autoAlpha: 1 })`), descubrirlo de arriba abajo con `clipPath` (de `"inset(0% 0% 100% 0%)"` a `"inset(0% 0% 0% 0%)"`), que los `.menu-enlace` suban con `stagger` y que aparezca `.menu-pie`.
3. Abrir: `menuTl.play()`. Cerrar: `menuTl.reverse()`. La misma animación sirve para las dos cosas.
4. Actualiza `aria-expanded` del botón y cierra también al pulsar un enlace o la tecla `Escape`.

---

## Extras

**Extra 1 · Laboratorio de easing.** Al pulsar **Lanzar**, la `.ficha` recorre la `.pista` con el ease elegido en `#ease`; **Volver** la devuelve al principio. Pista para la distancia: `pista.clientWidth - 32 - ficha.offsetWidth` (32 = relleno de la pista). Usa `overwrite: true` para que un tween nuevo cancele el anterior.

**Extra 2 · Evitar el parpadeo.** Al cargar, se ve la portada un instante antes de que empiece la intro. El CSS ya oculta `.cabecera` y `.portada` si `<html>` tiene la clase `.js`. Añade en el `<head>` (hueco marcado):

```html
<script>document.documentElement.classList.add('js');</script>
```

y, en `main.js`, justo antes de crear la intro: `gsap.set(".cabecera, .portada", { autoAlpha: 1 });`. Si no hay JavaScript, la web se ve normal.

**Extra 3 · Movimiento reducido.** Si el sistema pide reducir el movimiento, sáltate la intro y el tornillo:

```js
const reducirMovimiento = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
if (reducirMovimiento) intro.progress(1);   // directos al final
```

---

## Documentación

- [GSAP: primeros pasos](https://gsap.com/resources/get-started/) · [tweens](https://gsap.com/docs/v3/GSAP/Tween/) · [timelines](https://gsap.com/docs/v3/GSAP/Timeline/) · [parámetro de posición](https://gsap.com/resources/position-parameter/)
- [Visualizador de easing](https://gsap.com/docs/v3/Eases/)
- [stagger](https://gsap.com/resources/getting-started/Staggers/)
