# Programas de movilidades internacionales

Sitio web estático para consultar programas de becas y movilidad internacional de la Facultad Regional La Plata. La interfaz permite buscar programas, filtrarlos por país, grupo y mes de apertura, y consultar sus requisitos, beneficios y vías de contacto.

## Tecnologías

- HTML, CSS y JavaScript nativos.
- SVG para el mapa mundial interactivo.
- JSON como fuente de datos.
- Google Fonts para las tipografías `Fraunces` e `Inter`.
- Vercel para el despliegue.

No utiliza framework, gestor de paquetes ni proceso de compilación.

## Estructura

```text
.
|-- index.html     # Interfaz, estilos, lógica y geometría del mapa
|-- becas.json     # Catálogo y metadatos de los programas
|-- vercel.json    # Configuración mínima de Vercel
`-- README.md
```

`index.html` es grande porque contiene dentro de la constante `WORLD` toda la geometría GeoJSON utilizada para dibujar el mapa. Conviene evitar reformatear o editar manualmente ese bloque salvo que se vaya a reemplazar la cartografía.

## Ejecución local

El sitio carga `becas.json` mediante `fetch`, por lo que debe abrirse desde un servidor HTTP. Abrir `index.html` directamente con `file://` puede impedir la carga del catálogo.

Con Python:

```powershell
py -m http.server 8000
```

Luego visitar:

```text
http://localhost:8000
```

También se puede utilizar cualquier servidor estático, por ejemplo la extensión Live Server de VS Code.

## Funcionamiento

Al iniciar, `init()` realiza los siguientes pasos:

1. Carga `becas.json`.
2. Descarta los registros con `"activa": false`.
3. Cuenta cuántos programas corresponden a cada país.
4. Genera el mapa SVG y colorea los países según ese conteo.
5. Crea los filtros de grupo y mes.
6. Renderiza las tarjetas agrupadas.

El estado actual de la interfaz vive en un objeto JavaScript:

```js
{
  pais: null,
  grupo: null,
  mes: "",
  q: ""
}
```

Cada interacción modifica ese estado y llama a `render()`. La función `matches()` aplica conjuntamente todos los filtros activos.

### Mapa

La geometría GeoJSON se proyecta de forma equirectangular sobre un SVG de `1000 x 500`. Los países se relacionan con las becas mediante códigos ISO 3166-1 alpha-2, por ejemplo:

```json
"paises": ["AR", "DE"]
```

Solo los países asociados a programas activos son interactivos. Al seleccionar uno se filtra la lista; al volver a pulsarlo o pulsar el océano se elimina el filtro.

### Tarjetas

Las tarjetas se generan como HTML dentro de `cardHTML()`. Al abrir una tarjeta, JavaScript calcula la altura real de su contenido para animar el acordeón.

Los textos provenientes del JSON pasan por `esc()` antes de insertarse en la página.

### Diseño responsive

Los estilos están incluidos en `index.html`.

- Escritorio: mapa y listado aparecen en dos columnas.
- Hasta `880px`: la página pasa a una única columna y utiliza scroll vertical.
- Hasta `520px`: detalles y enlaces pasan a una columna.
- Hasta `380px`: se compactan el encabezado y el mapa.

También se contemplan `100dvh`, áreas seguras del dispositivo y `prefers-reduced-motion`.

## Formato de `becas.json`

El archivo posee cinco propiedades principales:

```json
{
  "becas": [],
  "nombres": {},
  "continente": {},
  "orden": [],
  "meses": []
}
```

### `becas`

Contiene los programas. Ejemplo reducido:

```json
{
  "activa": true,
  "num": 53,
  "title": "Nombre original del programa",
  "title_display": "Nombre mostrado",
  "País": "Alemania",
  "Descripción": "Descripción breve del programa.",
  "Requisitos": "Requisitos principales.",
  "Beneficios": "Beneficios principales.",
  "Público": "Destinatarios.",
  "Áreas": "Áreas académicas.",
  "paises": ["DE"],
  "niveles": ["Maestría"],
  "grupo": "Posgrado",
  "apertura": "Agosto-Octubre",
  "aperturaMeses": ["agosto", "septiembre", "octubre"],
  "cierreDisplay": "Diciembre",
  "cierreMeses": ["diciembre"],
  "notaFecha": "Las fechas pueden variar.",
  "urls": ["https://example.org"],
  "mails": ["contacto@example.org"]
}
```

Campos utilizados directamente por la interfaz:

| Campo | Uso |
| --- | --- |
| `activa` | Si es `false`, el programa no se carga. |
| `num` | Número mostrado en la tarjeta. |
| `title_display` / `title` | Título visible y contenido de búsqueda. |
| `País` | Texto descriptivo visible en la tarjeta. |
| `Descripción` | Descripción expandida. |
| `Requisitos`, `Beneficios`, `Público`, `Áreas` | Detalles del programa. |
| `paises` | Códigos ISO usados por el mapa y el filtro por país. |
| `grupo` | Categoría usada por los botones y la agrupación. |
| `apertura`, `aperturaMeses` | Texto visible y filtro por mes. |
| `cierreDisplay` | Fecha de cierre visible. |
| `notaFecha` | Aclaración opcional sobre las fechas. |
| `urls`, `mails` | Enlaces y contactos. Deben ser arrays, aunque estén vacíos. |

Los campos `niveles`, `Fechas`, `Sitio web`, `cierre` y `cierreMeses` se conservan como información estructurada, pero actualmente no participan de forma directa en el renderizado o filtrado.

### `nombres`

Relaciona códigos ISO con nombres en español:

```json
{
  "AR": "Argentina",
  "DE": "Alemania"
}
```

Todo código incluido en `becas[].paises` debería tener una entrada aquí. Este nombre aparece en el tooltip del mapa y en el contexto del filtro.

### `continente`

Relaciona códigos ISO con continentes. Actualmente se carga en JavaScript, pero no se utiliza para filtrar ni renderizar. Se mantiene disponible para futuras agrupaciones o filtros geográficos.

### `orden`

Define tanto los filtros de grupo como el orden de las secciones:

```json
[
  "Grado",
  "Posgrado",
  "Docentes/investigadores",
  "Graduados"
]
```

El valor de `becas[].grupo` debe coincidir exactamente con uno de estos textos. Un grupo ausente de `orden` no será renderizado.

### `meses`

Define las opciones del selector mensual. Los valores se escriben en minúsculas y deben coincidir con `aperturaMeses`.

## Agregar o modificar programas

1. Editar `becas.json` conservando la codificación UTF-8.
2. Agregar el registro dentro de `becas`.
3. Asignar un `num` único.
4. Usar códigos ISO válidos en `paises`.
5. Verificar que cada código exista en `nombres`.
6. Usar un `grupo` incluido en `orden`.
7. Usar meses incluidos en `meses`.
8. Mantener `urls`, `mails`, `paises` y `aperturaMeses` como arrays.
9. Levantar el servidor local y probar mapa, búsqueda, filtros y tarjeta.

Para ocultar temporalmente un programa sin eliminar sus datos:

```json
"activa": false
```

## Validación rápida

Comprobar que el JSON sea válido:

```powershell
node -p "(require('./becas.json').becas.length) + ' programas cargados'"
```

Antes de publicar, revisar:

- Que no haya errores en la consola del navegador.
- Que `becas.json` responda correctamente desde el servidor.
- Que los países nuevos sean seleccionables en el mapa.
- Que la búsqueda encuentre títulos, países, áreas y descripciones.
- Que los filtros combinados devuelvan resultados coherentes.
- Que la vista funcione en escritorio y en un ancho móvil cercano a `390px`.
- Que enlaces y correos abran el destino correcto.

## Despliegue

El repositorio está preparado para desplegarse como sitio estático en Vercel. `vercel.json` activa URLs limpias:

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "cleanUrls": true
}
```

No hay comando de build. Vercel debe publicar directamente la raíz del repositorio, incluyendo juntos `index.html` y `becas.json`.

## Consideraciones

- La aplicación depende de Google Fonts; si no hay conexión, el navegador utiliza tipografías del sistema.
- Los enlaces externos se abren con `target="_blank"` y `rel="noopener"`.
- No existen pruebas automatizadas ni validación formal del esquema JSON.
- Para cambios grandes, sería conveniente separar CSS, JavaScript y GeoJSON en archivos propios antes de ampliar funcionalidades.
