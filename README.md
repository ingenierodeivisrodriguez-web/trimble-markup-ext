# Creador de Etiquetas — Extensión para Trimble Connect

Extensión para el visor 3D de Trimble Connect (Workspace API). Al seleccionar una o varias
piezas en el visor, crea automáticamente un **text markup** sobre cada pieza con el valor de
una propiedad IFC elegida por el usuario.

Es una aplicación web estática de un solo archivo (`index.html`), sin dependencias que
instalar ni proceso de compilación.

## Uso

1. Abre el proyecto en Trimble Connect y activa la extensión desde el panel lateral.
2. Elige la propiedad IFC en el desplegable.
3. Selecciona una o varias piezas en el visor 3D.
4. Pulsa **Crear etiquetas** — o activa **Modo automático** para que se creen solas cada vez
   que cambie la selección.

## Propiedades disponibles

El nombre exacto de una propiedad varía según el modelo y el exportador IFC, así que cada
preset prueba varios nombres candidatos, sin distinguir mayúsculas de minúsculas:

| Preset      | Nombres candidatos                                                |
|-------------|-------------------------------------------------------------------|
| Nombre      | `Name`                                                            |
| Área        | `Area`, `Net Area`, `NetArea`, `Gross Area`, `GrossArea`          |
| Volumen     | `Volume`, `Net Volume`, `NetVolume`, `Gross Volume`, `GrossVolume`|
| ID          | (ninguno — usa directamente el `objectRuntimeId`)                 |
| Nivel       | `Level`, `Building Storey`, `BuildingStorey`                      |
| Material    | `Material`                                                        |
| Personalizada | El nombre exacto que escribas en el campo de texto               |

La opción **Personalizada** acepta cualquier nombre de propiedad, incluidos los que contienen
una barra, por ejemplo `Repère Assemblage/Elément béton`. También admite la forma
`ConjuntoDePropiedades/Propiedad` para acotar la búsqueda a un property set concreto.

Si no se encuentra ninguna coincidencia, la etiqueta usa como respaldo el texto `ID: <rid>` y
queda registrado en el log.

## Cómo funciona

1. Se conecta al visor con `TrimbleConnectWorkspace.connect(window.parent, callback, 15000)`.
2. Escucha el evento `viewer.modelObject.selection` y lee la selección inicial con
   `API.viewer.getSelection()`.
3. Por cada objeto, lee las propiedades con
   `API.viewer.getObjectProperties(modelId, [objectRuntimeId])` y recorre los property sets
   buscando los nombres candidatos.
4. Obtiene el bounding box con
   `API.viewer.getObjectBoundingBoxes(modelId, [objectRuntimeId])` y calcula su centro
   `(min + max) / 2` en x, y, z. La API de markup exige coordenadas, no solo texto.
5. Crea las etiquetas con `API.markup.addTextMarkup([...])`, donde cada elemento es
   `{ text, start, end, color }` y `start`/`end` son el mismo punto (`MarkupPick` con
   `positionX`, `positionY`, `positionZ`, `modelId`, `objectId`), ya que es una etiqueta
   puntual y no una línea.

Las etiquetas se envían en un solo lote. Si el lote es rechazado, se reintenta objeto por
objeto para identificar cuáles fallan, y el resumen final indica `X OK, Y fallo(s)`.

## Interfaz

- **Selector de propiedad** con presets y opción personalizada.
- **Contador de selección** en tiempo real.
- **Modo automático** con debounce de 500 ms, para no dispararse varias veces seguidas
  mientras cambia la selección.
- **Actualizar** vuelve a leer la selección actual del visor.
- **Limpiar** vacía el log.
- **Registro de actividad** con hora y estado ✓ / ✗ por cada acción. Ningún error es
  silencioso: los fallos de conexión, de lectura de propiedades, de bounding box y de la
  propia API de markup se muestran en pantalla.

## Instalación en Trimble Connect

Trimble Connect → **Extensiones** → **Añadir extensión personalizada**, y pega la URL del
`manifest.json` publicado.

## Archivos

- `index.html` — la aplicación completa (interfaz + lógica).
- `manifest.json` — descriptor de la extensión para Trimble Connect.
