---
name: fiori-MCB
description: Este skill permite a los agentes IA entender la arquitectura, las aplicaciones Fiori que componen MCB (Meli Central Buying). Incluye reglas de coordinación automática con el agente ABAP cuando se detectan dependencias de backend.
---

# MCB (Meli Central Buying) — Fiori Frontend Skill

Este skill permite a los agentes IA entender la arquitectura, las aplicaciones Fiori que componen **MCB (Meli Central Buying)**. El proyecto MCB digitaliza y gestiona el proceso de compras corporativas de MercadoLibre, desde la solicitud de pedido hasta la generación de OC/Contratos, incluyendo workflow de aprobaciones, SLA, notificaciones y lista de precios.

---

## Aplicaciones MCB

|   Nombre proyecto |   URL GITHUB  |   id app  |   Nombre  |   Descripción |
|---|---|---|---|---|
|   `fury_fps1pmelicentralbuying` |   https://github.com/melisource/fury_fps1pmelicentralbuying   |   meli.central.buying |   Nueva Solicitud de Compra   |   App para creación de formulario para gestión de ordenes de compra |
|   `fury_fps1pcentralbuyingcontractscre` |   https://github.com/melisource/fury_fps1pcentralbuyingcontractscre   |   meli.central.buying.contract.create |   Solicitud de Contrato   |   para creación de formulario para gestión de ordenes de compra |
|   `fury_fps1pcentralbuying`   |   https://github.com/melisource/fury_fps1pcentralbuying   |   meli.central.buying.po.edit |   Modificación orden de compra    | App para modificación de ordenes de compra creadas por medio de MCB   |
|   `fury_fps1pcentralbuyingcontractedit`   |   https://github.com/melisource/fury_fps1pcentralbuyingcontractedit   |   meli.central.buying.contract.edit   |   Modificación contrato   |   App para modificación de contratos creadas por medio de MCB |
|   `fury_fpsmcbmisformularios` |   https://github.com/melisource/fury_fpsmcbmisformularios |   meli.finops.po.central.buying.user.forms    |   Mis Solicitudes de Compra   |   App para gestión de los formularios por los solicitantes. En esta app se ven los form donde esta como solicitante, creador o colaborador    |
|   `fury_fpscentralbuyingform` |   https://github.com/melisource/fury_fpscentralbuyingform |   meli.finops.po.central.buying.forms |   Control de Formularios  |   App control de formularios, usada por los diferentes agentes de MCB para realizar aprobación de formularios |	
|   `fury_carga-masiva-forms`   |   https://github.com/melisource/fury_carga-masiva-forms   |   furyfps.formulariomasivo    |   Carga masiva de formularios |   App para carga masiva de formularios de Ordenes de Compra o de Contratos    |
|   `fury_pricelistabm` |   https://github.com/melisource/fury_pricelistabm  |   furypricelistabm   |   Gestión de Listas de Precios    |   App para gestión de listas de precios   |
|   `fury_mcbpricelist` |   https://github.com/melisource/fury_mcbpricelist |   pricelist   |   Creación Masiva de Lista de Precios |   App para creación masiva de lista de precios    |
|   `fury_fpsmcbdashboard`  |   https://github.com/melisource/fury_fpsmcbdashboard  |   meli.central.buying.dashboard   |   Ciclo de Vida Documentos de Compra  |   App reporte para visualizar ciclo de vida de documentos de un formulario    |
---

## Coordinación automática con el agente ABAP

Cuando estés analizando código Fiori MCB y detectes cualquiera de los siguientes indicadores, **debes escalar automáticamente al agente `abap-mcb-analyzer`** sin esperar que el usuario lo pida. Usa el `Agent tool` con `subagent_type: "abap-mcb-analyzer"`.

### Disparadores que requieren escalar al agente ABAP

| Lo que ves en Fiori | Escalar porque |
|---|---|
| Servicio OData en `manifest.json` (`ZFORM_UI_SRV_*`, `ZSRV_*`, `ZGW_*`) | El servicio vive en el backend ABAP |
| Error HTTP 4xx/5xx al llamar OData | Puede ser un bug en el Behavior Provider o DPC |
| Campo faltante o `null` en la respuesta OData | El CDS View o projection puede no exponer ese campo |
| Acción RAP inexistente o que falla (`generateSolpeds`, `submit`, etc.) | La acción está definida en el BDEF ABAP |
| Performance lenta en lectura de tabla (`ZTMM_FORMS`, `ZTMM_FORMITEMS`) | Puede requerir índice HANA o ajuste en AMDP |
| Filtros OData que no aplican correctamente | El `$filter` puede no estar soportado en el CDS |
| Entidad OData sin datos aunque existen en SAP | Problema en la CDS View o en el query provider |
| Workflow o SLA no avanza | La lógica está en `ZCL_E_FORM_WF_BR` / `ZCL_E_FORM_SLA` |
| Adjuntos DMS no se cargan | Bug en `ZCL_E_FORM_QI_ATTACHMENTS_DMS` |
| Notificaciones de mail no llegan | Problema en `ZCL_MAIL_SENDER` o Smart Templates |

### Cómo escalar al agente ABAP

Cuando detectes un disparador, lanza el agente ABAP con este patrón:

```
Escalar automáticamente — incluir en el prompt del agente ABAP:

"Contexto: Estoy analizando la app Fiori MCB [nombre_app] y encontré una dependencia
de backend que necesita análisis ABAP.

Servicio OData involucrado: [SRVB]
Entidad/acción problemática: [nombre]
Síntoma observado en Fiori: [descripción del problema]
Archivo Fiori donde se consume: [ruta del archivo]

Por favor analiza el objeto ABAP correspondiente y proporciona el plan de solución backend."
```

### Lo que NO requiere escalar al agente ABAP

- Problemas de CSS, layout, VBox/HBox
- Errores de formatter (`webapp/model/formatter.js`)
- Problemas de routing o navegación (`manifest.json > routing`)
- Textos hardcodeados (i18n)
- Linter errors de UI5
- Problemas de binding en XML View sin error de red

---

## Contexto del sistema MCB para escalar

Al escalar al agente ABAP, siempre incluir este contexto:

```
Sistema: SAP S/4HANA DS4, cliente 200
URL SAP: vhmcads4ci.rise.melisap.com:44300
Servicio principal OData V4: ZFORM_UI_SRV_SOLPED_O2
API externa: ZSRV_GSP_FORM_PO
Tabla core: ZTMM_FORMS
Clase orquestadora: ZCL_FORM_SOLPED_HANDLER
```
