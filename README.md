# Creador de Etiquetas — Extensión para Trimble Connect

Extensión para el visor 3D de Trimble Connect (Workspace API). Etiqueta las piezas del modelo
con los parámetros IFC que elijas, colocando la etiqueta sobre cada pieza al hacer clic en
ella.

Es una aplicación web estática de un solo archivo (`index.html`), sin dependencias que
instalar ni proceso de compilación.

## Uso

1. Abre el proyecto en Trimble Connect y activa la extensión desde el panel lateral.
2. Ajusta **tamaño y color de letra**. Se guarda y no hay que repetirlo.
3. Haz clic en una pieza: el panel lista los parámetros que esa pieza tiene de verdad.
4. **Marca los parámetros** que quieras en la etiqueta (GUID, NAME, Área, ID…). Puedes marcar
   varios: cada uno ocupa una línea. La vista previa muestra cómo quedará.
5. Con **Etiquetar al hacer clic** activado, cada pieza que selecciones se etiqueta sola. Si
   lo desactivas, usa el botón **Etiquetar selección**.

La selección de parámetros y la configuración del texto se guardan en el navegador, así que
la próxima sesión arranca como la dejaste.

## Los parámetros salen del modelo

No hay lista fija de nombres. Al seleccionar una pieza, la extensión llama a
`getObjectProperties` y muestra los parámetros que **esa pieza tiene**, agrupados por property
set y con su valor actual al lado. Así el nombre elegido siempre coincide con el del IFC, sea
cual sea el exportador o el idioma del modelo.

Además del listado, siempre están disponibles:

- **ID (objectRuntimeId)** — el identificador interno de la pieza.
- **Añadir parámetro por nombre exacto** — para aplicar a toda la selección un parámetro que
  la primera pieza no tiene. Acepta nombres con barra, como `Repère Assemblage/Elément béton`,
  y la forma `PropertySet/Propiedad`.

## Varias piezas a la vez

Cada pieza se resuelve por separado:

1. Coincidencia exacta `PropertySet/Parámetro`.
2. Si no existe, el mismo nombre de parámetro en cualquier otro property set.
3. Si tampoco, la etiqueta usa `ID: <rid>` y el estado lo indica.

## Cómo funciona

1. Se conecta al visor con `TrimbleConnectWorkspace.connect(window.parent, callback, 15000)`.
2. Escucha `viewer.modelObject.selection` (con debounce de 450 ms) y lee la selección inicial
   con `API.viewer.getSelection()`.
3. Lee los parámetros con `API.viewer.getObjectProperties(modelId, [objectRuntimeId])`.
4. Obtiene el bounding box con `API.viewer.getObjectBoundingBoxes(modelId, [objectRuntimeId])`
   y calcula su centro `(min + max) / 2`.
5. Crea la etiqueta con `API.markup.addTextMarkup([...])`. La forma del punto está confirmada
   leyendo un markup que el propio host había guardado:

```js
start: { type: 'point', positionX, positionY, positionZ }
end:   { type: 'point', positionX, positionY, positionZ }   // por encima de la pieza
```

El campo `type: 'point'` es obligatorio: sin él, el visor no resuelve el punto y coloca la
etiqueta en el origen. El host asigna su propio `id` numérico y descarta `modelId`/`objectId`
dentro del punto, así que no se envían.

### Verificación de lo creado

Tras crear las etiquetas, la extensión las relee con `getTextMarkups()` y comprueba su
posición real. Si el visor las guardó todas en el mismo punto, o no se puede leer la posición,
lo dice en el estado en vez de dar la operación por buena. El botón **Diagnóstico de markup**
vuelca las claves de `API.markup` y la posición de cada etiqueta guardada.

## Limitaciones conocidas

- **El tamaño de letra depende del visor.** Los text markups de Trimble Connect no guardan un
  tamaño propio: se envía en los campos `size` y `fontSize` por si el host los admite, y si no
  los guarda, el estado lo avisa explícitamente. El color sí se aplica.
- **Borrar etiquetas** usa `removeMarkups()` con los ids creados en la sesión. Si el visor no
  expone ese método, hay que borrarlas desde la barra de markup de Trimble.

## Instalación en Trimble Connect

Trimble Connect → **Extensiones** → **Añadir extensión personalizada**, y pega:

```
https://ingenierodeivisrodriguez-web.github.io/trimble-markup-ext/manifest.json
```

La cabecera del panel muestra la versión (`v9`). Si no coincide con la última, recarga con
Ctrl+F5: el navegador está sirviendo una copia cacheada.

## Archivos

- `index.html` — la aplicación completa (interfaz + lógica).
- `manifest.json` — descriptor de la extensión para Trimble Connect.
