# SAPUI5 Design & Controls - Reglas de Diseño

> **APLICABILIDAD**: Todas las aplicaciones SAPUI5 FreeStyle
> **PRIORIDAD**: ⭐ CRÍTICO - Cumplimiento obligatorio al 100%
> **FUENTE**: Fiori Design Guidelines y SAP UI5 Standards

## 1. Uso Estricto de Controles Estándar

### ❌ PROHIBIDO ABSOLUTAMENTE
```css
/* ❌ NUNCA modificar controles UI5 con CSS personalizado */
.sapMBtn {
    background-color: red !important;
    border-radius: 20px !important;
}

.sapMList .sapMLIB {
    padding: 20px !important;
    background: linear-gradient(...) !important;
}
```

```javascript
// ❌ NUNCA manipular DOM interno de controles
this.byId("myButton").getDomRef().style.backgroundColor = "red";
this.byId("myButton").$().addClass("custom-style");
```

### ✅ OBLIGATORIO: Solo Controles Estándar
```javascript
// ✅ CORRECTO: Usar propiedades nativas del control
const oButton = new Button({
    text: "{i18n>buttonText}",
    type: ButtonType.Emphasized,  // Usar tipos predefinidos
    icon: "sap-icon://accept",
    width: "200px",              // Propiedades permitidas
    enabled: true
});

// ✅ CORRECTO: Usar clases CSS solo para contenedores de aplicación
const oPanel = new Panel({
    headerText: "{i18n>panelHeader}",
    content: [oButton],
    class: "myAppContainer"      // Solo para contenedores propios
});
```

### CSS Permitido Solo para Contenedores de Aplicación
```css
/* ✅ CORRECTO: CSS solo para contenedores de aplicación */
.myAppContainer {
    margin: 1rem;
    padding: 1rem;
}

.myCustomLayout {
    display: flex;
    justify-content: space-between;
}

/* ✅ CORRECTO: Espaciado entre controles */
.sapUiMediumMargin {
    /* Usar clases de espaciado predefinidas de UI5 */
}
```

## 2. Fiori Design Guidelines Compliance

### Patrones de Diseño Obligatorios
```xml
<!-- ✅ CORRECTO: Usar patrones Fiori estándar -->
<Page title="{i18n>pageTitle}" 
      showNavButton="true"
      navButtonPress="onNavBack">
    
    <!-- Header con acciones estándar -->
    <customHeader>
        <Bar>
            <contentLeft>
                <Button icon="sap-icon://nav-back" press="onNavBack"/>
            </contentLeft>
            <contentMiddle>
                <Title text="{i18n>pageTitle}"/>
            </contentMiddle>
            <contentRight>
                <Button icon="sap-icon://action" press="onAction"/>
            </contentRight>
        </Bar>
    </customHeader>
    
    <!-- Contenido con controles estándar -->
    <content>
        <List items="{/items}">
            <StandardListItem 
                title="{name}"
                description="{description}"
                icon="{icon}"
                press="onItemPress"/>
        </List>
    </content>
</Page>
```

### Iconografía Estándar
```javascript
// ✅ OBLIGATORIO: Solo iconos SAP predefinidos
const aStandardIcons = [
    "sap-icon://accept",
    "sap-icon://decline",
    "sap-icon://edit",
    "sap-icon://delete",
    "sap-icon://add",
    "sap-icon://save",
    "sap-icon://cancel"
];

// ❌ PROHIBIDO: Iconos personalizados o externos
const sCustomIcon = "data:image/svg+xml;base64,..."; // ❌ INCORRECTO
```

## 3. Theming y Temas Oficiales

### ✅ OBLIGATORIO: Solo Temas SAP Oficiales
```html
<!-- ✅ CORRECTO: Temas oficiales SAP -->
<script id="sap-ui-bootstrap"
    data-sap-ui-theme="sap_horizon">        <!-- Recomendado -->
</script>

<!-- Otros temas oficiales permitidos -->
<!-- data-sap-ui-theme="sap_fiori_3" -->
<!-- data-sap-ui-theme="sap_fiori_3_dark" -->
<!-- data-sap-ui-theme="sap_horizon_dark" -->
<!-- data-sap-ui-theme="sap_horizon_hcb" -->  <!-- Alto contraste -->
<!-- data-sap-ui-theme="sap_horizon_hcw" -->  <!-- Alto contraste -->
```

### ❌ PROHIBIDO: Temas Personalizados
```css
/* ❌ NUNCA crear temas personalizados */
:root {
    --sapBrandColor: #custom-color;
    --sapHighlightColor: #another-color;
}
```

## 4. Responsive Design

### Breakpoints Estándar UI5
```javascript
// ✅ CORRECTO: Usar breakpoints predefinidos de UI5
sap.ui.define([
    "sap/ui/Device"
], function(Device) {
    "use strict";
    
    // Detectar dispositivo
    if (Device.system.phone) {
        // Lógica para móvil
    } else if (Device.system.tablet) {
        // Lógica para tablet
    } else {
        // Lógica para desktop
    }
});
```

### Layout Responsivo con Controles UI5
```xml
<!-- ✅ CORRECTO: Usar controles responsivos nativos -->
<ResponsiveGridLayout 
    defaultSpan="XL3 L4 M6 S12"
    defaultIndent="XL1 L1 M1 S0">
    
    <Button text="{i18n>button1}"/>
    <Button text="{i18n>button2}"/>
    <Button text="{i18n>button3}"/>
    
</ResponsiveGridLayout>

<!-- ✅ CORRECTO: Usar FlexBox nativo -->
<FlexBox direction="Row" 
         justifyContent="SpaceBetween"
         alignItems="Center">
    <Button text="{i18n>buttonLeft}"/>
    <Button text="{i18n>buttonCenter}"/>
    <Button text="{i18n>buttonRight}"/>
</FlexBox>
```

## 5. Accesibilidad en Controles

### ARIA Labels Obligatorios
```xml
<!-- ✅ OBLIGATORIO: Labels para accesibilidad -->
<Button text="{i18n>buttonSave}"
        tooltip="{i18n>buttonSaveTooltip}"
        ariaLabelledBy="saveButtonLabel"/>

<Label id="saveButtonLabel" 
       text="{i18n>saveButtonAriaLabel}"
       visible="false"/>

<!-- ✅ OBLIGATORIO: Roles ARIA cuando sea necesario -->
<Panel accessibleRole="Region"
       ariaLabelledBy="panelHeader">
    <headerToolbar>
        <Toolbar>
            <Title id="panelHeader" text="{i18n>userInfoTitle}"/>
        </Toolbar>
    </headerToolbar>
</Panel>
```

## 6. Validación de Controles

### Controles Permitidos por Categoría

#### Navegación
```javascript
// ✅ PERMITIDOS
- sap.m.Button
- sap.m.MenuButton  
- sap.m.SegmentedButton
- sap.tnt.NavigationList
- sap.f.ShellBar
```

#### Entrada de Datos
```javascript
// ✅ PERMITIDOS
- sap.m.Input
- sap.m.TextArea
- sap.m.ComboBox
- sap.m.MultiComboBox
- sap.m.DatePicker
- sap.m.TimePicker
- sap.m.CheckBox
- sap.m.RadioButton
- sap.m.Switch
- sap.m.Slider
- sap.m.StepInput
```

#### Visualización de Datos
```javascript
// ✅ PERMITIDOS
- sap.m.List
- sap.m.Table
- sap.ui.table.Table
- sap.m.Tree
- sap.suite.ui.commons.ChartContainer
- sap.viz.ui5.controls.VizFrame
```

### ❌ CONTROLES PROHIBIDOS
```javascript
// ❌ NUNCA usar controles deprecated
- sap.ui.commons.*     // Biblioteca obsoleta
- sap.ui.ux3.*        // Biblioteca obsoleta
- sap.ca.*            // Biblioteca obsoleta

// ❌ NUNCA crear controles HTML nativos
const oDiv = document.createElement("div");     // ❌ INCORRECTO
const oInput = document.createElement("input"); // ❌ INCORRECTO
```

## 7. Patrones de Interacción

### Navegación Estándar
```javascript
// ✅ CORRECTO: Navegación con Router
onNavigateToDetail: function(oEvent) {
    const oContext = oEvent.getSource().getBindingContext();
    const sObjectId = oContext.getProperty("ID");
    
    this.getRouter().navTo("detail", {
        objectId: sObjectId
    });
}

// ✅ CORRECTO: Navegación hacia atrás
onNavBack: function() {
    const oHistory = History.getInstance();
    const sPreviousHash = oHistory.getPreviousHash();
    
    if (sPreviousHash !== undefined) {
        window.history.go(-1);
    } else {
        this.getRouter().navTo("main", {}, true);
    }
}
```

### Feedback al Usuario
```javascript
// ✅ CORRECTO: Usar controles estándar para feedback
MessageToast.show("Operación completada exitosamente");

MessageBox.confirm("¿Está seguro de eliminar este elemento?", {
    onClose: function(oAction) {
        if (oAction === MessageBox.Action.OK) {
            // Proceder con eliminación
        }
    }
});

// ✅ CORRECTO: Indicadores de carga
BusyIndicator.show(0);
// ... operación asíncrona
BusyIndicator.hide();
```

---

## 8. Layouts y Contenedores (VBox, HBox, FlexBox)

### Reglas de Uso de VBox/HBox

#### ✅ Cuándo SÍ usar VBox/HBox

**Para Organizar Filtros en Formularios:**
```xml
<!-- ✅ PERFECTO: Organizar campos de formulario -->
<VBox class="sapUiSmallMargin">
    <Label text="{i18n>name}"/>
    <Input value="{/name}"/>
    
    <Label text="{i18n>email}"/>
    <Input value="{/email}"/>
</VBox>
```

**Para Toolbars y Botones:**
```xml
<!-- ✅ PERFECTO: Botones horizontales -->
<HBox>
    <Button text="{i18n>save}" press="onSave"/>
    <Button text="{i18n>cancel}" press="onCancel"/>
</HBox>
```

**Para Cards/Tiles en Dashboards:**
```xml
<!-- ✅ PERFECTO: Dashboard cards -->
<VBox class="sapUiSmallMarginBottom">
    <Title text="{i18n>statistics}"/>
    <HBox wrap="Wrap">
        <GenericTile header="{i18n>total}"/>
        <GenericTile header="{i18n>pending}"/>
        <GenericTile header="{i18n>validated}"/>
    </HBox>
</VBox>
```

#### ❌ Cuándo NO usar VBox/HBox

**Como Contenedor Principal de Page:**
```xml
<!-- ❌ INCORRECTO: VBox como contenedor principal -->
<Page>
    <content>
        <VBox>  <!-- ❌ Causa problemas de altura -->
            <Panel>Filter Bar</Panel>
            <Table/>  <!-- Se corta a la mitad -->
        </VBox>
    </content>
</Page>

<!-- ✅ CORRECTO: Contenido directo en Page -->
<Page>
    <content>
        <Panel>Filter Bar</Panel>
        <Table/>  <!-- Ocupa todo el espacio disponible -->
    </content>
</Page>
```

**Para Layouts de Tabla + Filtros:**
```xml
<!-- ❌ INCORRECTO: VBox contenedor para SmartFilterBar + SmartTable -->
<Page>
    <content>
        <VBox>  <!-- ❌ Problema de altura -->
            <smartFilterBar:SmartFilterBar/>
            <smartTable:SmartTable/>  <!-- Se corta -->
        </VBox>
    </content>
</Page>

<!-- ✅ CORRECTO: Sin contenedor VBox -->
<Page>
    <content>
        <smartFilterBar:SmartFilterBar id="smartFilterBar"/>
        <smartTable:SmartTable 
            smartFilterId="smartFilterBar"
            enableAutoBinding="true"/>
    </content>
</Page>
```

### Alternativas Modernas a VBox/HBox

#### FlexBox (Más Flexible)
```xml
<!-- ✅ MEJOR que VBox/HBox para layouts complejos -->
<FlexBox 
    direction="Column"
    justifyContent="SpaceBetween"
    alignItems="Stretch"
    fitContainer="true">
    
    <FlexBox direction="Row" wrap="Wrap">
        <!-- Filtros -->
    </FlexBox>
    
    <Table/>  <!-- Se expande automáticamente -->
</FlexBox>
```

#### Grid (Para Layouts Responsivos)
```xml
<!-- ✅ MEJOR para layouts responsivos -->
<Grid 
    defaultSpan="XL3 L4 M6 S12"
    hSpacing="1"
    vSpacing="1">
    
    <GenericTile header="{i18n>total}"/>
    <GenericTile header="{i18n>pending}"/>
    <GenericTile header="{i18n>validated}"/>
    <GenericTile header="{i18n>rejected}"/>
</Grid>
```

#### DynamicPage (Para List Reports)
```xml
<!-- ✅ RECOMENDADO para patrón List Report -->
<f:DynamicPage>
    <f:header>
        <f:DynamicPageHeader>
            <smartFilterBar:SmartFilterBar id="smartFilterBar"/>
        </f:DynamicPageHeader>
    </f:header>
    <f:content>
        <smartTable:SmartTable 
            smartFilterId="smartFilterBar"
            enableAutoBinding="true"/>
    </f:content>
</f:DynamicPage>
```

### Patrones Comunes por Escenario

#### Patrón 1: List Report (Filter + Table)
```xml
<!-- ✅ PATRÓN RECOMENDADO -->
<Page>
    <content>
        <!-- Filter Bar -->
        <Panel expandable="true" expanded="true">
            <VBox class="sapUiSmallMargin">  <!-- ✅ OK aquí -->
                <HBox wrap="Wrap" alignItems="End">
                    <VBox class="sapUiSmallMarginEnd">
                        <Label text="{i18n>country}"/>
                        <ComboBox items="{/countries}"/>
                    </VBox>
                    <VBox class="sapUiSmallMarginEnd">
                        <Label text="{i18n>status}"/>
                        <ComboBox items="{/statuses}"/>
                    </VBox>
                    <HBox>
                        <Button text="{i18n>clear}"/>
                        <Button text="{i18n>search}" type="Emphasized"/>
                    </HBox>
                </HBox>
            </VBox>
        </Panel>
        
        <!-- Table (SIN VBox contenedor) -->
        <Panel>
            <Table 
                items="{/items}"
                growing="true"
                growingThreshold="20"
                sticky="ColumnHeaders"/>
        </Panel>
    </content>
</Page>
```

#### Patrón 2: Object Page (Header + Content)
```xml
<!-- ✅ PATRÓN RECOMENDADO -->
<f:DynamicPage>
    <f:header>
        <f:DynamicPageHeader>
            <VBox>  <!-- ✅ OK en header -->
                <Title text="{invoice/number}"/>
                <HBox>
                    <Label text="{i18n>status}:"/>
                    <ObjectStatus text="{invoice/status}"/>
                </HBox>
            </VBox>
        </f:DynamicPageHeader>
    </f:header>
    <f:content>
        <!-- Content directo, sin VBox -->
        <IconTabBar>
            <items>
                <IconTabFilter text="{i18n>details}">
                    <VBox>  <!-- ✅ OK dentro de tab -->
                        <SimpleForm>
                            <!-- Form fields -->
                        </SimpleForm>
                    </VBox>
                </IconTabFilter>
            </items>
        </IconTabBar>
    </f:content>
</f:DynamicPage>
```

#### Patrón 3: Dashboard (Cards + Table)
```xml
<!-- ✅ PATRÓN RECOMENDADO -->
<Page>
    <content>
        <!-- Statistics Cards -->
        <VBox class="sapUiSmallMarginBottom">  <!-- ✅ OK para cards -->
            <Title text="{i18n>statistics}" level="H2"/>
            <HBox wrap="Wrap" renderType="Bare">
                <GenericTile header="{i18n>total}">
                    <layoutData>
                        <FlexItemData minWidth="12rem" maxWidth="20rem" growFactor="1"/>
                    </layoutData>
                </GenericTile>
                <GenericTile header="{i18n>pending}">
                    <layoutData>
                        <FlexItemData minWidth="12rem" maxWidth="20rem" growFactor="1"/>
                    </layoutData>
                </GenericTile>
            </HBox>
        </VBox>
        
        <!-- Table (sin VBox contenedor) -->
        <Panel>
            <Table items="{/items}"/>
        </Panel>
    </content>
</Page>
```

### Tabla de Decisión Rápida

| Escenario | Usar VBox/HBox | Alternativa Recomendada |
|-----------|----------------|-------------------------|
| **Contenedor principal de Page** | ❌ NO | Contenido directo |
| **Organizar filtros** | ✅ SÍ | FlexBox |
| **Botones en toolbar** | ✅ SÍ | HBox |
| **Cards/Tiles** | ✅ SÍ | Grid, FlexBox |
| **Formularios** | ✅ SÍ | Form, SimpleForm |
| **Table + FilterBar** | ❌ NO | DynamicPage |
| **Header de página** | ✅ SÍ | VBox |
| **Tabs content** | ✅ SÍ | VBox |

### Problemas Comunes y Soluciones

#### Problema: Tabla se corta a la mitad
```xml
<!-- ❌ CAUSA: VBox como contenedor principal -->
<Page>
    <content>
        <VBox>
            <Panel>Filters</Panel>
            <Table/>  <!-- Se corta -->
        </VBox>
    </content>
</Page>

<!-- ✅ SOLUCIÓN: Quitar VBox contenedor -->
<Page>
    <content>
        <Panel>Filters</Panel>
        <Table/>  <!-- Ocupa todo el espacio -->
    </content>
</Page>
```

#### Problema: Scroll extra innecesario
```xml
<!-- ❌ CAUSA: VBox con height fijo -->
<VBox height="100%">
    <Table/>  <!-- Genera scroll extra -->
</VBox>

<!-- ✅ SOLUCIÓN: Usar FlexBox con fitContainer -->
<FlexBox fitContainer="true" direction="Column">
    <Table/>  <!-- Sin scroll extra -->
</FlexBox>
```

---

## 9. Smart Controls en FreeStyle (SmartFilterBar, SmartTable, SmartField)

> **NOTA IMPORTANTE**: Esta sección cubre el uso de Smart Controls **individuales** en aplicaciones SAPUI5 FreeStyle. NO cubre Fiori Elements Templates (List Report, Object Page, etc.) que son una solución diferente.

### ✅ Smart Controls Cubiertos

- `sap.ui.comp.smartfilterbar.SmartFilterBar`
- `sap.ui.comp.smarttable.SmartTable`
- `sap.ui.comp.smartfield.SmartField`
- `sap.ui.comp.smartform.SmartForm`
- `sap.ui.comp.smartvariants.SmartVariantManagement`

### SmartFilterBar en FreeStyle

#### Configuración Básica Obligatoria
```xml
<!-- ✅ CORRECTO: Configuración mínima requerida -->
<smartFilterBar:SmartFilterBar
    id="smartFilterBar"
    entitySet="Products"
    persistencyKey="myAppProductFilters"
    search="onSearch">
</smartFilterBar:SmartFilterBar>
```

**Propiedades Obligatorias:**
- `entitySet`: Nombre del entity set de OData
- `persistencyKey`: Clave única para guardar filtros del usuario
- `search`: Evento cuando el usuario presiona "Go"

#### Vinculación con SmartTable
```xml
<!-- ✅ CORRECTO: SmartFilterBar vinculado con SmartTable -->
<Page>
    <content>
        <!-- FilterBar -->
        <smartFilterBar:SmartFilterBar
            id="smartFilterBar"
            entitySet="Products"
            persistencyKey="productFilters">
        </smartFilterBar:SmartFilterBar>
        
        <!-- Table vinculada -->
        <smartTable:SmartTable
            id="smartTable"
            entitySet="Products"
            smartFilterId="smartFilterBar"
            enableAutoBinding="true">
        </smartTable:SmartTable>
    </content>
</Page>
```

#### Custom Filters Programáticos
```javascript
// Controller.js
onInit: function() {
    const oSmartFilterBar = this.byId("smartFilterBar");
    
    // ✅ CORRECTO: Agregar filtro custom
    oSmartFilterBar.addCustomFilter({
        name: "customStatus",
        label: "Estado Custom",
        type: "sap.m.ComboBox"
    });
}
```

#### Problemas Comunes y Soluciones

**Problema 1: Filtros no se aplican automáticamente**
```javascript
// ❌ INCORRECTO: Falta enableAutoBinding
<smartTable:SmartTable
    entitySet="Products"
    smartFilterId="smartFilterBar">
</smartTable:SmartTable>

// ✅ CORRECTO: Agregar enableAutoBinding
<smartTable:SmartTable
    entitySet="Products"
    smartFilterId="smartFilterBar"
    enableAutoBinding="true">
</smartTable:SmartTable>
```

**Problema 2: Persistencia no funciona**
```xml
<!-- ❌ INCORRECTO: Sin persistencyKey -->
<smartFilterBar:SmartFilterBar
    entitySet="Products">
</smartFilterBar:SmartFilterBar>

<!-- ✅ CORRECTO: Con persistencyKey único -->
<smartFilterBar:SmartFilterBar
    entitySet="Products"
    persistencyKey="myApp_ProductFilters_v1">
</smartFilterBar:SmartFilterBar>
```

---

### SmartTable en FreeStyle

#### Configuración Básica Obligatoria
```xml
<!-- ✅ CORRECTO: Configuración mínima requerida -->
<smartTable:SmartTable
    id="smartTable"
    entitySet="Products"
    smartFilterId="smartFilterBar"
    tableType="ResponsiveTable"
    useExportToExcel="true"
    enableAutoBinding="true"
    showRowCount="true"
    header="Products">
</smartTable:SmartTable>
```

**Propiedades Obligatorias:**
- `entitySet`: Nombre del entity set de OData
- `smartFilterId`: ID del SmartFilterBar vinculado
- `tableType`: "ResponsiveTable" (móvil) o "GridTable" (desktop)
- `enableAutoBinding`: true para carga automática

#### Layout Correcto (CRÍTICO)

**❌ INCORRECTO: VBox como contenedor**
```xml
<!-- ❌ MAL: Causa que la tabla se corte a la mitad -->
<Page>
    <content>
        <VBox>  <!-- ❌ Problema aquí -->
            <smartFilterBar:SmartFilterBar id="smartFilterBar"/>
            <smartTable:SmartTable 
                smartFilterId="smartFilterBar"/>  <!-- Se corta -->
        </VBox>
    </content>
</Page>
```

**✅ CORRECTO: Sin VBox contenedor**
```xml
<!-- ✅ BIEN: Tabla ocupa todo el espacio disponible -->
<Page>
    <content>
        <smartFilterBar:SmartFilterBar 
            id="smartFilterBar"
            entitySet="Products"/>
        
        <smartTable:SmartTable
            id="smartTable"
            entitySet="Products"
            smartFilterId="smartFilterBar"
            enableAutoBinding="true"
            tableType="ResponsiveTable"/>
    </content>
</Page>
```

**✅ ALTERNATIVA: Usar DynamicPage (Recomendado Fiori)**
```xml
<!-- ✅ MEJOR: Patrón Fiori con DynamicPage -->
<f:DynamicPage>
    <f:header>
        <f:DynamicPageHeader pinnable="true">
            <smartFilterBar:SmartFilterBar
                id="smartFilterBar"
                entitySet="Products"/>
        </f:DynamicPageHeader>
    </f:header>
    <f:content>
        <smartTable:SmartTable
            id="smartTable"
            entitySet="Products"
            smartFilterId="smartFilterBar"
            enableAutoBinding="true"/>
    </f:content>
</f:DynamicPage>
```

#### Export a Excel
```xml
<!-- ✅ CORRECTO: Habilitar export -->
<smartTable:SmartTable
    entitySet="Products"
    useExportToExcel="true"
    exportType="GW">  <!-- GW = Gateway, UI5Client = Client-side -->
    
    <!-- Personalizar columnas exportadas -->
    <smartTable:customData>
        <core:CustomData 
            key="p13nDialogSettings"
            value='{"export": {"fileName": "Products.xlsx"}}'/>
    </smartTable:customData>
</smartTable:SmartTable>
```

#### Variant Management
```xml
<!-- ✅ CORRECTO: Habilitar gestión de variantes -->
<smartTable:SmartTable
    entitySet="Products"
    useVariantManagement="true"
    useTablePersonalisation="true"
    persistencyKey="myApp_ProductTable_v1">
</smartTable:SmartTable>
```

#### Personalización de Columnas Programática
```javascript
// Controller.js
onBeforeRebindTable: function(oEvent) {
    const mBindingParams = oEvent.getParameter("bindingParams");
    
    // ✅ CORRECTO: Agregar filtros adicionales
    mBindingParams.filters.push(
        new Filter("Status", FilterOperator.EQ, "Active")
    );
    
    // ✅ CORRECTO: Agregar sorters
    mBindingParams.sorter.push(
        new Sorter("ProductName", false)
    );
    
    // ✅ CORRECTO: Seleccionar campos específicos
    mBindingParams.parameters.select = "ProductID,ProductName,Price";
}
```

#### Problemas Comunes y Soluciones

**Problema 1: Tabla se corta a la mitad**
```xml
<!-- ❌ CAUSA: VBox contenedor -->
<VBox>
    <smartTable:SmartTable/>  <!-- Se corta -->
</VBox>

<!-- ✅ SOLUCIÓN: Quitar VBox -->
<Page>
    <content>
        <smartTable:SmartTable/>  <!-- Ocupa todo el espacio -->
    </content>
</Page>
```

**Problema 2: Datos no se cargan**
```xml
<!-- ❌ INCORRECTO: Falta enableAutoBinding -->
<smartTable:SmartTable
    entitySet="Products"
    smartFilterId="smartFilterBar">
</smartTable:SmartTable>

<!-- ✅ CORRECTO: Agregar enableAutoBinding -->
<smartTable:SmartTable
    entitySet="Products"
    smartFilterId="smartFilterBar"
    enableAutoBinding="true">
</smartTable:SmartTable>
```

**Problema 3: Export no funciona**
```xml
<!-- ❌ INCORRECTO: Falta configuración -->
<smartTable:SmartTable entitySet="Products">
</smartTable:SmartTable>

<!-- ✅ CORRECTO: Habilitar export -->
<smartTable:SmartTable
    entitySet="Products"
    useExportToExcel="true"
    exportType="GW">
</smartTable:SmartTable>
```

---

### SmartField en FreeStyle

#### Uso en Formularios
```xml
<!-- ✅ CORRECTO: SmartField en formulario -->
<VBox>
    <Label text="Product Name"/>
    <smartField:SmartField
        value="{ProductName}"
        editable="true">
    </smartField:SmartField>
    
    <Label text="Price"/>
    <smartField:SmartField
        value="{Price}"
        editable="true">
    </smartField:SmartField>
</VBox>
```

#### Tipos Automáticos desde OData
```xml
<!-- ✅ CORRECTO: SmartField detecta tipo automáticamente -->
<smartField:SmartField value="{OrderDate}"/>
<!-- Renderiza DatePicker automáticamente -->

<smartField:SmartField value="{Quantity}"/>
<!-- Renderiza Input numérico automáticamente -->

<smartField:SmartField value="{Status}"/>
<!-- Renderiza ComboBox si hay value list en metadata -->
```

#### Value Help Integration
```xml
<!-- ✅ CORRECTO: Value help automático desde annotations -->
<smartField:SmartField
    value="{CustomerID}"
    showValueHelp="true">
</smartField:SmartField>
```

#### Custom Configuration
```xml
<!-- ✅ CORRECTO: Configuración personalizada -->
<smartField:SmartField
    value="{Price}"
    editable="true"
    mandatory="true"
    placeholder="Enter price"
    textAlign="End">
    
    <!-- Custom control cuando sea necesario -->
    <smartField:configuration>
        <smartField:Configuration
            displayBehaviour="descriptionAndId"
            preventInitialDataFetchInValueHelpDialog="false">
        </smartField:Configuration>
    </smartField:configuration>
</smartField:SmartField>
```

---

### SmartForm en FreeStyle

#### Layout Automático
```xml
<!-- ✅ CORRECTO: SmartForm con grupos -->
<smartForm:SmartForm
    id="productForm"
    editable="true"
    title="Product Details">
    
    <!-- Grupo General -->
    <smartForm:Group label="General Information">
        <smartForm:GroupElement label="Product Name">
            <smartField:SmartField value="{ProductName}"/>
        </smartForm:GroupElement>
        
        <smartForm:GroupElement label="Description">
            <smartField:SmartField value="{Description}"/>
        </smartForm:GroupElement>
    </smartForm:Group>
    
    <!-- Grupo Pricing -->
    <smartForm:Group label="Pricing">
        <smartForm:GroupElement label="Price">
            <smartField:SmartField value="{Price}"/>
        </smartForm:GroupElement>
        
        <smartForm:GroupElement label="Currency">
            <smartField:SmartField value="{Currency}"/>
        </smartForm:GroupElement>
    </smartForm:Group>
</smartForm:SmartForm>
```

#### Responsive Behavior
```xml
<!-- ✅ CORRECTO: Layout responsivo automático -->
<smartForm:SmartForm
    editable="true"
    useHorizontalLayout="false">  <!-- Vertical en móvil -->
    
    <smartForm:layout>
        <smartForm:ColumnLayout
            columnsXL="2"
            columnsL="2"
            columnsM="1">
        </smartForm:ColumnLayout>
    </smartForm:layout>
    
    <!-- Grupos y elementos -->
</smartForm:SmartForm>
```

#### Edit vs Display Mode
```javascript
// Controller.js
onEdit: function() {
    const oSmartForm = this.byId("productForm");
    oSmartForm.setEditable(true);
},

onDisplay: function() {
    const oSmartForm = this.byId("productForm");
    oSmartForm.setEditable(false);
}
```

---

### Integración con OData

#### Metadata Requerido
```xml
<!-- OData Metadata necesario para Smart Controls -->
<EntityType Name="Product">
    <Property Name="ProductID" Type="Edm.String" Nullable="false"/>
    <Property Name="ProductName" Type="Edm.String" 
              sap:label="Product Name"
              sap:filterable="true"
              sap:sortable="true"/>
    <Property Name="Price" Type="Edm.Decimal" 
              sap:label="Price"
              sap:unit="Currency"
              sap:filterable="true"/>
    <Property Name="Currency" Type="Edm.String" 
              sap:semantics="currency-code"/>
    <Property Name="Status" Type="Edm.String"
              sap:label="Status"
              sap:value-list="fixed-values"/>
</EntityType>
```

#### Annotations Importantes
```xml
<!-- Annotations para SmartFilterBar -->
<Annotations Target="MyService.Product">
    <Annotation Term="UI.SelectionFields">
        <Collection>
            <PropertyPath>ProductName</PropertyPath>
            <PropertyPath>Status</PropertyPath>
        </Collection>
    </Annotation>
</Annotations>

<!-- Annotations para SmartTable -->
<Annotations Target="MyService.Product">
    <Annotation Term="UI.LineItem">
        <Collection>
            <Record Type="UI.DataField">
                <PropertyValue Property="Value" PropertyPath="ProductName"/>
            </Record>
            <Record Type="UI.DataField">
                <PropertyValue Property="Value" PropertyPath="Price"/>
            </Record>
        </Collection>
    </Annotation>
</Annotations>
```

---

### Configuración en manifest.json

#### DataSource para Smart Controls
```json
{
  "sap.app": {
    "dataSources": {
      "mainService": {
        "uri": "/sap/opu/odata/sap/MY_SERVICE/",
        "type": "OData",
        "settings": {
          "odataVersion": "2.0",
          "localUri": "localService/metadata.xml",
          "annotations": ["annotation0"]
        }
      },
      "annotation0": {
        "type": "ODataAnnotation",
        "uri": "annotations/annotation.xml",
        "settings": {
          "localUri": "annotations/annotation.xml"
        }
      }
    }
  },
  "sap.ui5": {
    "models": {
      "": {
        "dataSource": "mainService",
        "preload": true,
        "settings": {
          "defaultBindingMode": "TwoWay",
          "defaultCountMode": "Inline",
          "refreshAfterChange": false,
          "metadataUrlParams": {
            "sap-value-list": "none"
          }
        }
      }
    }
  }
}
```

---

## Herramientas de Validación

### Verificación de Compliance
```bash
# Verificar uso de controles estándar
grep -r "\.sap[A-Z]" src/ --include="*.js" --include="*.ts"

# Verificar CSS personalizado en controles UI5
grep -r "\.sap[A-Z].*{" src/ --include="*.css" --include="*.less"
```

### MCP Tools para Validación
- Usar `get_api_reference` para verificar controles disponibles
- Usar `run_ui5_linter` para detectar uso de controles deprecated

---

**IMPORTANTE**: El incumplimiento de estas reglas puede causar:
- Problemas de compatibilidad con actualizaciones de UI5
- Fallos en temas y modo oscuro
- Problemas de accesibilidad
- Inconsistencias visuales con Fiori Design Guidelines