# Cono y Cronómetro

Cronómetro de vertidos para café filtrado. En vez de una cuenta atrás,
te dice **el peso que debe marcar la báscula en este segundo**: durante un
vertido el número sube interpolado, en la pausa se queda quieto. Solo igualas.

Una página, sin dependencias ni build. Abre `index.html` y ya.

![Los tres pasos](docs/pantallas.png)

## Qué hace

- **Diagrama de tiempos vertical.** Cada bloque mide lo que dura de verdad, así
  ves de un vistazo que el bloom son 12 s y la espera 33. Una marca baja por el
  eje conforme corre el tiempo.
- **Peso objetivo en grande**, con una regla de muescas en cada objetivo de vertido.
- **Estados con color y palabra**: vierte / espera / drena / listo, con avisos de
  audio en cada cambio y cuenta de 3 antes de arrancar.
- **La pantalla no se apaga** mientras corre el cronómetro (Screen Wake Lock).
  Si el navegador no lo permite, la app te lo dice en vez de fingir.
- **Ajuste de dosis**: cambias los gramos de café y se recalcula toda el agua.
- **Registro del drenado real**, que es el dato que diagnostica la molienda:
  si se te va más de 30 s por encima de lo previsto, mueles demasiado fino.
- **Cinco recetas de referencia**: una taza (V60), Hoffmann, 4:6 de Tetsu Kasuya,
  Kalita Wave y Chemex.

## Dos modos

Esta misma página corre en dos sitios y se adapta a lo que hay:

| | GitHub Pages / local | Dentro de Claude |
|---|---|---|
| Cronómetro y recetas | ✅ | ✅ |
| Leer la etiqueta con una foto | — | ✅ |
| Memoria de bolsas y tazas | — | ✅ |

La lectura de la etiqueta usa la capacidad `sample` del runtime de artifacts de
Claude (`window.claude`), y la memoria usa `db`. Fuera de ese runtime no existen,
así que la página lo detecta al cargar, oculta la cámara y arranca directo en las
recetas de referencia. No hay claves de API en ninguna parte y no las habrá:
una página estática no es sitio para una credencial.

## Estructura

```
index.html    todo: estilos, marcado y motor. Fuente de verdad.
```

La versión que corre dentro de Claude es este mismo archivo sin el envoltorio
`<!doctype html>…<head>` — el visor de artifacts pone el suyo. Si editas aquí,
hay que volver a publicarla allá.

## Cómo está construido

**Sistema de coordenadas del tiempo.** Una receta es una lista de vertidos con
`inicio` (segundo en que empiezas a verter), `duracion` y `hasta` (peso
**acumulado** en la báscula al terminar ese vertido). De ahí se derivan los
segmentos de espera y el drenado; nadie los escribe a mano.

**El peso en el segundo `t`** se interpola dentro del vertido en curso y se
congela en las pausas. Es la única función que importa de verdad:

```js
function pesoEn(t){ /* lerp dentro del vertido, plano fuera */ }
```

**Los bloques del diagrama** se posicionan en porcentaje sobre el tiempo total,
así que la marca de "ahora" siempre cae donde debe sin sincronizar nada.

### Trampas encontradas

1. `clientHeight` de algo dentro de un panel con `hidden` da **0**. Las líneas de
   detalle del diagrama se medían antes de mostrar el panel y desaparecían todas.
   Se mide después, en un `requestAnimationFrame`.
2. El cronómetro deriva el tiempo de `performance.now()`, no de contar frames:
   si el navegador suspende el `requestAnimationFrame` al ocultar la pestaña, al
   volver el reloj sigue correcto.
3. Una línea horizontal de "ahora" que cruza todo el ancho **se lee como texto
   tachado** sobre el vertido en curso. Quedó como una marca corta sobre el riel.

## Licencia

MIT.
