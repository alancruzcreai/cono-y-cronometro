# Cono y Cronómetro

Guía de vertidos para café filtrado. Un paso a la vez, a pantalla completa, y
en vez de una cuenta atrás te dice **el peso que debe marcar la báscula en este
segundo**: mientras viertes el número sube interpolado; en la pausa se queda
quieto. Solo igualas.

Una página, sin dependencias ni build. Abre `index.html` y ya.

![Los tres pasos](docs/pantallas.png)

## Qué hace

- **Una tarjeta por paso.** Título, qué hacer, el número que importa y el botón.
  Nada más en pantalla mientras tienes el hervidor en la mano.
- **El número se adapta al momento**: mientras viertes es el peso objetivo en
  gramos; en las pausas y el drenado es el tiempo que falta.
- **Barra de progreso por pasos** arriba, y `‹ ›` para adelantar o rehacer uno.
- **La pantalla no se apaga** mientras corre el cronómetro (Screen Wake Lock).
  Si el navegador no lo permite, la app lo dice en vez de fingir.
- **Avisos de audio** en cada cambio de paso y cuenta de 3 antes de arrancar.
- **Ajuste de dosis**: cambias los gramos de café y se recalcula toda el agua.
- **Registro del drenado real**, que es el dato que diagnostica la molienda: si
  se te va más de 30 s por encima de lo previsto, mueles demasiado fino.
- **Cinco recetas de referencia**: una taza (V60), Hoffmann, 4:6 de Tetsu
  Kasuya, Kalita Wave y Chemex, cada una con su porqué y sus correcciones.
- **Compartir**: manda la receta como texto legible más un enlace que la
  reconstruye entera en el cronómetro del que lo recibe.
- **PDF**: una ficha de una página para guardar o pegar en la pared.

## Compartir

El botón de la pantalla de receta arma un texto legible por sí solo —dosis,
agua, ratio, temperatura, molienda y la tabla de vertidos con sus tiempos— y le
pega al final un enlace que **reconstruye esa receta exacta** en el cronómetro
de quien lo abra.

La receta va comprimida dentro del `#hash`, así que el enlace no depende de
ningún servidor: no hay base de datos, ni identificadores, ni nada que caduque.
Una receta de cinco vertidos con su explicación cabe en unos 760 caracteres.

```
receta → objeto compacto → JSON → deflate-raw → base64url → #r=…
```

`deflate-raw` sale de `CompressionStream`, que es nativo; donde no exista, el
JSON viaja sin comprimir y el enlace solo queda más largo. Se comparte con
`navigator.share` y, si el navegador no lo trae, cae al portapapeles y, si
tampoco, a un cuadro de texto ya seleccionado. El botón nunca se queda sin
hacer nada.

## PDF

El botón arma una ficha de una página —nombre, parámetros, tabla de vertidos
con sus notas, el porqué y qué mover si no sale bien— y abre el diálogo de
impresión, donde «Guardar como PDF» es una de las salidas.

No lleva ninguna librería de PDF. La ficha es HTML con su propio `@media
print`, así que la dibuja el motor del navegador: sale con la tipografía real,
en vectores, y el texto sigue siendo texto seleccionable. Una librería habría
pesado 350 KB para producir algo peor, y además los descargas por script están
bloqueados dentro de un visor incrustado, con lo que el botón no haría nada
allí.

## Temas

Claro y oscuro, incluida la pantalla de vertido, que es la que más rato miras.
Todos los colores salen de tokens redefinidos en los tres estados que existen
de verdad: `:root` (claro), `@media (prefers-color-scheme: dark)` protegido con
`:root:not([data-theme="light"])`, y `:root[data-theme="dark"]` para que un
cambio manual gane en los dos sentidos.

El botón de la barra cicla **automático → claro → oscuro** y recuerda la
elección. «Automático» no es un tercer tema: es no estampar nada y dejar que
mande el sistema, así que si cambias el teléfono a oscuro de noche la app lo
sigue sin tocar nada. La elección se aplica en un script del `<head>`, antes de
pintar, para que no parpadee el tema equivocado al cargar.

## Tipografía

Fuentes del sistema, cero descargas: `ui-serif` para los títulos (New York en
Apple, Georgia en el resto) y `system-ui` para todo lo demás. Además de verse
como debe en iOS, evita el parpadeo de una fuente web en una pantalla que
miras tres minutos seguidos.

## Dos modos

La misma página se adapta a dónde corre:

| | GitHub Pages / local | En un host con lectura de imagen |
|---|---|---|
| Cronómetro y recetas | ✅ | ✅ |
| Leer la etiqueta con una foto | — | ✅ |
| Memoria de bolsas y tazas | — | ✅ |

Lo detecta al cargar: si no hay quien interprete la foto, oculta la cámara y
arranca directo en las recetas de referencia. **No hay claves de API en ninguna
parte y no las habrá**: una página estática no es sitio para una credencial.

## Estructura

```
index.html    todo: estilos, marcado y motor. Fuente de verdad.
```

## Cómo está construido

**El reloj manda.** Una receta es una lista de vertidos con `inicio` (segundo en
que empiezas a verter), `duracion` y `hasta` (peso **acumulado** en la báscula
al terminar ese vertido). De ahí se derivan las esperas y el drenado, y de ahí
sale todo lo demás: qué tarjeta se ve, cuánto lleva llena cada barra de
progreso y qué número va en grande. Nadie escribe esos estados a mano.

```js
cardAt(t)   // qué tarjeta toca en el segundo t
pesoEn(t)   // interpola dentro del vertido, se congela fuera
```

Avanzar de tarjeta no es un evento: es una consecuencia de que el reloj pasó un
límite. Por eso `‹ ›` simplemente mueven el reloj y todo lo demás se acomoda.

### Trampas encontradas

1. El cronómetro deriva el tiempo de `performance.now()`, no de contar frames:
   si el navegador suspende el `requestAnimationFrame` al ocultar la pestaña, al
   volver el reloj sigue correcto.
2. Un hijo de ancho fijo dentro de un contenedor `align-items:center` que a su
   vez está en un bloque con `text-align:center` **no queda centrado**: el
   `text-align` no alcanza a un elemento de bloque. El botón de play y la barra
   se iban a la izquierda hasta convertir la tarjeta en un flex column.
3. `clientHeight` de algo dentro de un panel con `hidden` da **0**, así que
   cualquier medida hay que tomarla después de mostrarlo.

## Licencia

MIT.
