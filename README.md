# Creador de Etiquetas — Extensión para Trimble Connect

Extensión para el visor 3D de Trimble Connect (Workspace API). Al seleccionar una o varias
piezas en el visor, crea un **text markup** sobre cada pieza con el valor de una propiedad IFC
elegida por el usuario.

Es una aplicación web estática de un solo archivo (`index.html`), sin dependencias que
instalar ni proceso de compilación.

## Uso

1. Abre el proyecto en Trimble Connect y activa la extensión desde el panel lateral.
2. Selecciona una pieza en el visor 3D. La extensión lee sus propiedades y las carga en el
   desplegable.
3. Elige la propiedad que quieras usar como etiqueta.
4. Pulsa **Crear etiquetas** — o activa **Modo automático** para que se creen solas cada vez
   que cambie la selección.

## El desplegable se construye desde el modelo

No hay una lista fija de nombres de propiedad. Al seleccionar una pieza, la extensión llama a
`getObjectProperties` y rellena el desplegable con las propiedades que **esa pieza realmente
tiene**, agrupadas por property set, tal y como aparecen en el modelo. Así el nombre elegido
siempre coincide con el del IFC, sea cual sea el exportador o el idioma del modelo.

Además del listado del modelo, el desplegable ofrece siempre:

- **ID (objectRuntimeId)** — usa el identificador interno de la pieza, sin leer propiedades.
- **Personalizada…** — para escribir un nombre exacto a mano, útil si quieres aplicar a toda
  la selección una propiedad que la primera pieza no tiene.

Debajo del desplegable se muestra el property set y el **valor actual** de la propiedad
elegida para la pieza seleccionada, para confirmar de un vistazo que es la correcta.

## Multiselección

El desplegable se construye a partir de la primera pieza seleccionada. Al crear las etiquetas,
cada pieza se resuelve por separado:

1. Se busca la coincidencia exacta `PropertySet/Propiedad`.
2. Si no existe, se busca el mismo nombre de propiedad en cualquier otro property set.
3. Si tampoco existe, la etiqueta usa como respaldo `ID: <rid>`.

El estado final indica cuántas se crearon y cuántas no tenían esa propiedad, por ejemplo
`3 OK, 0 fallo(s) · 2 sin esa propiedad (se usó el ID)`.

## Cómo funciona

1. Se conecta al visor con `TrimbleConnectWorkspace.connect(window.parent, callback, 15000)`.
2. Escucha el evento `viewer.modelObject.selection` y lee la selección inicial con
   `API.viewer.getSelection()`.
3. Lee las propiedades con `API.viewer.getObjectProperties(modelId, [objectRuntimeId])` y
   recorre los property sets devueltos.
4. Obtiene el bounding box con
   `API.viewer.getObjectBoundingBoxes(modelId, [objectRuntimeId])` y calcula su centro
   `(min + max) / 2` en x, y, z. La API de markup exige coordenadas, no solo texto.
5. Crea las etiquetas con `API.markup.addTextMarkup([...])`, donde cada elemento es
   `{ text, start, end, color }` y `start`/`end` son el mismo punto (`MarkupPick` con
   `positionX`, `positionY`, `positionZ`, `modelId`, `objectId`), ya que es una etiqueta
   puntual y no una línea.

Las etiquetas se envían en un solo lote. Si el lote es rechazado, se reintenta objeto por
objeto para no perder las que sí son válidas.

## Interfaz

- **Selector de propiedad** poblado con las propiedades reales del modelo.
- **Contador de selección** en tiempo real.
- **Modo automático** con debounce de 500 ms, para no dispararse varias veces seguidas
  mientras cambia la selección.
- **Actualizar propiedades** vuelve a leer la selección y sus propiedades.
- **Línea de estado** al pie con el resultado de la última acción. Ningún error es silencioso:
  los fallos de conexión, de lectura de propiedades, de bounding box y de la propia API de
  markup se muestran ahí.

## Instalación en Trimble Connect

Trimble Connect → **Extensiones** → **Añadir extensión personalizada**, y pega la URL del
`manifest.json` publicado:

```
https://ingenierodeivisrodriguez-web.github.io/trimble-markup-ext/manifest.json
```

## Archivos

- `index.html` — la aplicación completa (interfaz + lógica).
- `manifest.json` — descriptor de la extensión para Trimble Connect.
