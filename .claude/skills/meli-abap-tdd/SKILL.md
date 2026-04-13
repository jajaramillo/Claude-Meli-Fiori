---
name: meli-abap-tdd
description: Genera automáticamente el Documento de Diseño Técnico (TDD) en Español para desarrollos SAP ABAP del cliente Meli. Úsalo cuando el usuario proporcione un número de OT, un ID de ticket Jira/Solman, código ABAP, o diga "genera el TDD", "documentar el desarrollo", "generar documentación técnica" o "llenar el template".
---

# Meli ABAP TDD Writer

Genera el TDD completo (15 secciones) en Español. El modo principal es **automático por OT**: dado un número de OT, usa el MCP `abap-adt` para buscar los objetos, leer el código y extraer los cambios del encabezado. El modo secundario es **manual**: el usuario pega el código y datos directamente.

---

## MODO AUTOMÁTICO (input = número de OT o Jira/Solman)

### Paso 1 — Obtener la OT

Si el usuario da un **número de OT** (formato `DSXKXXXXXXXX`, ej: `DS4K9B569G`):
- Intentar `GetTransport(transport)`. Si falla por permisos, usar `RunQuery` como fallback (ver abajo).
- Extraer: lista de objetos, descripción de la OT, tipo (Workbench/Customizing), autor, ID Solman

Si el usuario da un **ID Solman** (ej: `5000044228`) o **Jira** (ej: `SDFPSINIC-23815`):
- Buscar la OT principal en la tabla `E07T` via `RunQuery`:
  ```sql
  SELECT e070~trkorr, e070~trfunction, e070~trstatus, e070~as4user, e07t~as4text
  FROM e070 INNER JOIN e07t ON e070~trkorr = e07t~trkorr
  WHERE e07t~as4text LIKE '%[SOLMAN_O_JIRA]%'
  AND e070~trfunction <> 'T'
  ```
- Filtrar solo órdenes principales (padre): `trfunction IN ('W', 'K')`. Excluir tareas (`S`, `R`) y copias (`T`).
- Si hay varias OTs padre, mostrarlas al usuario y pedir que elija.

**Fallback cuando GetTransport está bloqueado — obtener objetos via E071:**
```sql
SELECT pgmid, object, obj_name FROM e071
WHERE trkorr IN ('[OT_PADRE]', '[TAREAS_HIJAS]')
ORDER BY object, obj_name ASCENDING
```
Para obtener las tareas hijas: `SELECT trkorr FROM e070 WHERE strkorr = '[OT_PADRE]'`

Deduplicar resultados de E071. Ignorar objetos de tipo `RELE` (son registros de liberación, no objetos ABAP).

### Paso 2 — Leer el código de cada objeto

Para cada objeto único de la OT, llamar `GetSource(object_type, name)`.

Mapeo de tipos E071 a tipos de GetSource:
- `PROG` / `REPS` → `PROG` o `INCL` | `CLAS` / `CLSD` / `CPUB` / `METH` → `CLAS` | `INTF` → `INTF`
- `FUGR` → `FUGR` | `FUNC` → `FUNC` | `REPS` (include de FUGR, empieza con `L`) → `INCL`
- `DDLS` → `DDLS` (CDS view) | `TABL` → usar `GetTable`
- Para `METH`: usar `GetSource(CLAS, clase, include=implementations, method=nombre_metodo)` — más eficiente que leer la clase completa
- Para `CLAS` con muchos métodos: leer solo los métodos listados en E071 con `METH`, no toda la clase
- Si `GetSource` retorna un archivo demasiado grande: usar `GrepObjects` con el patrón del encabezado (`Transport\s*:\s*[OT]`) para encontrar el `Change ID`, y luego usar `GrepObjects` con patrón `AD_Vxxx|MOD_Vxxx` para encontrar las líneas del delta. **NO conformarse solo con leer el encabezado — siempre buscar el código cambiado.**

Para CDS views (`DDLS`): también llamar `GetCDSDependencies` para obtener las tablas base (Sección 11).

### Paso 3 — Parsear el encabezado Globant

Cada objeto tiene un encabezado estándar. Buscar y extraer los campos de estas secciones:

**Sección de creación** (siempre presente):
```
* Program/Class/Function Module : [nombre]
* Created by     : [autor creación]
* Creation date  : [fecha creación]
* Description    : [descripción del objeto]
* Transport      : [OT de creación]
```

**Secciones de modificación** (puede haber varias, una por cambio):
```
* Modified by    : [autor modificación]
* Requirement N° : [número requerimiento]
* Change ID      : [ID cambio]
* Date           : [fecha modificación]
* Description    : [descripción del cambio]
* Transport      : [OT de la modificación]  ← buscar la OT solicitada aquí
```

**Regla clave:** Buscar en TODAS las secciones `* Transport :` el número de OT que el usuario proporcionó. La sección que contenga esa OT indica el cambio de esta versión. Extraer:
- `Modified by` → nombre del desarrollador (Sección 1)
- `Description` → descripción general del cambio en el encabezado
- `Change ID` → el identificador de versión (ej: `V004`) — **guardar esto, se necesita en el Paso 4**
- Si la OT coincide con `* Transport :` de la sección de **creación** → el objeto es nuevo en esta OT

### Paso 4 — Identificar SOLO las líneas de ESTA OT en el código

**CRÍTICO: Solo documentar el delta de esta OT. No describir cambios de OTs anteriores.**

Los comentarios inline del código identifican a qué versión pertenece cada línea:
- `"MOD_Vxxx` → línea **modificada** en la versión xxx
- `"AD_Vxxx` → línea **agregada** en la versión xxx
- `"DEL_Vxxx` → línea **eliminada** en la versión xxx

**Procedimiento por método:**
1. Obtener el `Change ID` del encabezado para esta OT en este método (ej: `V004`)
2. Buscar en el código **solo** las líneas con tag `"MOD_V004`, `"AD_V004` o `"DEL_V004`
3. Documentar **únicamente** esas líneas/bloques en la Sección 5
4. Ignorar líneas con tags de versiones anteriores (`"MOD_V003`, `"AD_V001`, etc.) — esos pertenecen a OTs anteriores

**Ejemplo correcto:** Si el método tiene `Change ID: V004` para esta OT, y el código tiene:
```
WHEN iv_fiscal_year > iv_year_inv_entry "MOD_V004   ← documentar esta línea
THEN iv_fiscal_year                                  ← documentar esta línea
ELSE iv_year_inv_entry ).               "MOD_V003   ← NO documentar, es OT anterior
```
Solo se documenta la rama `WHEN iv_fiscal_year > iv_year_inv_entry`.

**Si el objeto es nuevo en esta OT** (la OT coincide con `* Transport :` en la sección de creación, no en modificación): documentar la lógica completa del objeto/método.

### Paso 5 — Buscar tags de modificación inline BEGIN/END

Dentro del código, buscar bloques del formato:
```
****BEGIN [Autor] - [Descripción proyecto] S [Solman] - [Fecha]***
[código agregado o comentado con explicación]
****END [Autor] - [Descripción proyecto] S [Solman] - [Fecha]***
```

Si el número Solman del tag coincide con el de la OT → ese bloque contiene los cambios específicos. Usar el código del bloque (sin pegarlo crudo) para enriquecer la Sección 5.

### Paso 5 — Manejar encabezado ausente

Si un objeto **no tiene encabezado Globant**:
- Avisar al desarrollador: "El objeto `[NOMBRE]` no tiene encabezado estándar Globant. ¿Es un objeto nuevo o una modificación?"
  - Si es **nuevo**: documentar todo desde el código fuente completo.
  - Si es **modificación**: pedirle que agregue el encabezado antes de generar el TDD, o que describa el cambio en el chat.

### Paso 6 — Permitir contexto adicional

Si el desarrollador quiere dar más contexto, puede:
- Pegar la especificación técnica directamente en el chat → incorporarla en Secciones 2, 3 y 5
- Subir un archivo `.md` con la especificación → leerlo con Read y usarlo como input adicional

---

## MODO MANUAL (fallback)

Si no hay OT o el usuario pega código directamente:
1. Solicitar: número de OT, descripción del ticket (Jira/Solman), y código fuente
2. Procesar el código manualmente con el workflow de análisis estándar

---

## Workflow de análisis del código

Una vez obtenido el código (por MCP o manual), analizar **solo las líneas del delta de esta OT** (tags MOD_Vxxx / AD_Vxxx):
- **SELECTs/JOINs:** qué tablas lee y con qué filtros → anotar para Sección 11
- **UPDATE/INSERT/DELETE/MODIFY:** qué tablas modifica → anotar para Sección 11
- **Llamadas externas:** BAPIs, RFCs, Function Modules, clases llamadas → Sección 11
- **Tablas de configuración consultadas** (MARV, T001, etc.) → pueden generar Pre Requisitos (Sección 7)
- **BAdIs / User Exits / Enhancement Spots:** si existen → Sección 12
- **Objetos referenciados** que NO están en la OT → Sección 11
- **Lógica de negocio:** qué problema resuelve el delta, en lenguaje de negocio

**Regla Sección 7:** Si el código del delta depende de una tabla de configuración (ej: MARV para período contable, T001 para sociedad, etc.), documentar como pre-requisito que esa tabla debe estar actualizada en los sistemas destino. No asumir N/A automáticamente.

**Regla Sección 13 — Rollback específico:** Indicar el número de OT anterior (la que tenía la versión previa) y los Change IDs a los que se revierte. Esta info sale del encabezado del código (la sección de modificación inmediatamente anterior a la de esta OT).

---

## Reglas de generación

- **Mapeo Estricto:** Llenar TODAS las 15 secciones. Si falta info: "N/A" o "Pendiente de definir". Sin placeholders vacíos.
- **Tablas Obligatorias:** Secciones 1, 2, 4 y 6 SIEMPRE como tablas Markdown. La Sección 4 con columnas `Objeto | Tipo | Descripción`.
- **Lenguaje Técnico SAP:** UPDATE/INSERT/COMMIT WORK (no "guardar"), BAPI, infotipo, CDS View, BAdI, User Exit.
- **Todo en Español:** Traducir inputs en Portugués o Inglés.
- **Inventario Exhaustivo (Sección 4):** Listar TODOS los objetos fila por fila, sin agrupar. Usar descripción en español en columna Tipo (ej: "Clase" no "CLAS").
- **Lógica Natural (Sección 5):** Explicar el algoritmo con palabras. NUNCA pegar código crudo. Sub-sección por clase/include. **Agrupar métodos que comparten la misma lógica de cambio** en vez de repetir la misma descripción para cada uno. El título de sub-sección es solo el nombre del objeto/método — **NUNCA agregar "(Cambio Vxxx)" ni ningún identificador de versión en el título.** La versión ya está en el encabezado del código, no en el TDD.
- **Cero Alucinaciones:** Si falta contexto de negocio, escribir: "Requiere contexto de negocio no provisto".
- **Autor:** Usar el nombre extraído del encabezado (`Modified by` o `Created by`). Si no hay, usar "Consultor ABAP".
- **Extracción de Transporte (Sección 6):** Para una cadena como `DS4K9B569G S 5000044228: SDFPSINIC-23815 - Ctrl de Bloq Proveedor`:
  - **Solman:** Número tras `S ` → `5000044228`. Si no hay patrón `S XXXXXXXXX`, escribir `Pendiente de definir`
  - **OT:** Código inicial formato `DSXKXXXXXXXX` → `DS4K9B569G`
  - **Descripción:** Texto completo exacto desde el ticket → `SDFPSINIC-23815 - Ctrl de Bloq Proveedor`

---

## Formato del output

**ESTRUCTURA OBLIGATORIA:** El documento generado DEBE seguir exactamente la plantilla en [references/tdd-template.md](references/tdd-template.md). Las 15 secciones deben aparecer con los mismos títulos, en el mismo orden y con el mismo formato de tablas que la plantilla. No inventar secciones adicionales ni omitir ninguna.

Secciones que SIEMPRE son tablas Markdown:
- **Sección 1** — tabla con columnas: Autor del documento | Fecha | Versión | Resumen del cambio
- **Sección 2** — tabla con columnas: Objetivo | Beneficio
- **Sección 4** — tabla con columnas: Objeto | Tipo | Descripción
- **Sección 6** — tabla con columnas: Solman | OT | Tipo | Descripción

**NO imprimir el documento en el chat.** Guardar como archivo `.md` en el directorio actual con la herramienta Write.

**Nombre del archivo:** Derivar de la descripción de la OT o ticket:
- `SDFPSINIC-23815 - Ctrl de Bloq Proveedor` → `SDFPSINIC-23815_Ctrl_de_Bloq_Proveedor.md`
- Reemplazar espacios con `_`, eliminar caracteres especiales, extensión `.md`

**Responder SIEMPRE en este orden:**

1. Reporte de estado en el chat:
```
ESTADO DE LA DOCUMENTACIÓN:
✅ Secciones completadas: 15/15.
📄 Archivo generado: [nombre_del_archivo.md]
🔍 Objetos procesados: [N] objetos leídos via MCP
👤 Desarrollador detectado: [nombre extraído del encabezado]
❓ DATOS FALTANTES / SUPOSICIONES REALIZADAS:
   - [Listar qué faltó, qué objetos no tenían encabezado, qué fue asumido]
⚠️  ATENCIÓN: Revisa manualmente las secciones de pruebas, cutover y las suposiciones mencionadas.
```

2. Usar Write para crear el archivo `.md` con el documento completo.
