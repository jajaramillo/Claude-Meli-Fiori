# SAPUI5 Accessibility & i18n - Accesibilidad e Internacionalización

> **APLICABILIDAD**: Todas las aplicaciones SAPUI5 FreeStyle
> **PRIORIDAD**: ⭐ CRÍTICO - Cumplimiento obligatorio al 100%
> **FUENTE**: SAP Accessibility Guidelines y i18n Best Practices

## 1. Regla Fundamental: Todos los Textos en i18n

### ❌ PROHIBIDO ABSOLUTAMENTE: Textos Hardcodeados
```xml
<!-- ❌ NUNCA usar textos directos en XML -->
<Button text="Guardar"/>                    <!-- ❌ INCORRECTO -->
<Title text="Dashboard Principal"/>         <!-- ❌ INCORRECTO -->
<Label text="Nombre del Cliente"/>          <!-- ❌ INCORRECTO -->
<MessageToast.show("Operación exitosa");   <!-- ❌ INCORRECTO -->
```

### ✅ OBLIGATORIO: Solo Binding i18n
```xml
<!-- ✅ CORRECTO: Todos los textos desde archivos i18n -->
<Button text="{i18n>buttonSave}"/>
<Title text="{i18n>dashboardTitle}"/>
<Label text="{i18n>customerNameLabel}"/>
```

```javascript
// ✅ CORRECTO: Mensajes desde i18n
MessageToast.show(this.getResourceBundle().getText("operationSuccess"));
```

## 2. Accesibilidad (ARIA y Screenreader Support)

### ✅ OBLIGATORIO: ARIA Labels con i18n
```xml
<!-- ✅ CORRECTO: Labels descriptivos desde i18n -->
<Button text="{i18n>buttonSave}"
        tooltip="{i18n>buttonSaveTooltip}"
        ariaLabelledBy="saveButtonLabel"
        ariaDescribedBy="saveButtonDescription"/>

<Label id="saveButtonLabel" 
       text="{i18n>saveButtonAriaLabel}"
       visible="false"/>

<Text id="saveButtonDescription"
      text="{i18n>saveButtonAriaDescription}"
      visible="false"/>
```

### Roles ARIA Obligatorios
```xml
<!-- ✅ OBLIGATORIO: Roles semánticos apropiados -->
<Panel accessibleRole="Region"
       ariaLabelledBy="panelHeader">
    <headerToolbar>
        <Toolbar>
            <Title id="panelHeader" text="{i18n>customerInfoTitle}"/>
        </Toolbar>
    </headerToolbar>
    <content>
        <!-- Contenido del panel -->
    </content>
</Panel>

<!-- ✅ OBLIGATORIO: Landmarks para navegación -->
<Page accessibleRole="Main"
      ariaLabelledBy="pageTitle">
    <customHeader>
        <Bar accessibleRole="Banner">
            <contentMiddle>
                <Title id="pageTitle" text="{i18n>dashboardTitle}"/>
            </contentMiddle>
        </Bar>
    </customHeader>
</Page>
```

### Estados y Propiedades ARIA
```xml
<!-- ✅ OBLIGATORIO: Estados dinámicos -->
<Input value="{/customerName}"
       required="true"
       ariaRequired="true"
       ariaInvalid="{= ${/customerNameError} ? 'true' : 'false'}"
       ariaDescribedBy="customerNameError"/>

<Text id="customerNameError"
      text="{/customerNameErrorMessage}"
      visible="{= !!${/customerNameError}}"
      class="sapUiFormFieldError"/>
```

## 2. Navegación por Teclado

### ✅ OBLIGATORIO: Soporte Completo de Teclado
```javascript
// ✅ CORRECTO: Manejo de eventos de teclado
sap.ui.define([
    "sap/ui/core/mvc/Controller",
    "sap/ui/events/KeyCodes"
], function(Controller, KeyCodes) {
    "use strict";
    
    return Controller.extend("com.myapp.controller.Main", {
        
        onKeyDown: function(oEvent) {
            const iKeyCode = oEvent.which || oEvent.keyCode;
            
            // Enter para activar
            if (iKeyCode === KeyCodes.ENTER) {
                this.onItemPress(oEvent);
                oEvent.preventDefault();
            }
            
            // Escape para cancelar
            if (iKeyCode === KeyCodes.ESCAPE) {
                this.onCancel();
                oEvent.preventDefault();
            }
        }
    });
});
```

### Tab Order y Focus Management
```xml
<!-- ✅ OBLIGATORIO: Orden de tabulación lógico -->
<VBox>
    <Input id="firstName" 
           value="{/firstName}"
           fieldGroupIds="personalInfo"/>
    
    <Input id="lastName" 
           value="{/lastName}"
           fieldGroupIds="personalInfo"/>
    
    <Input id="email" 
           value="{/email}"
           fieldGroupIds="contactInfo"/>
    
    <!-- Botones al final del formulario -->
    <HBox>
        <Button text="{i18n>buttonSave}" press="onSave"/>
        <Button text="{i18n>buttonCancel}" press="onCancel"/>
    </HBox>
</VBox>
```

## 3. Soporte RTL (Right-to-Left)

### ✅ OBLIGATORIO: Configuración RTL
```javascript
// ✅ CORRECTO: Detectar y configurar RTL
sap.ui.define([
    "sap/ui/core/Configuration"
], function(Configuration) {
    "use strict";
    
    // Verificar configuración RTL
    const bRTL = Configuration.getRTL();
    
    if (bRTL) {
        // Ajustes específicos para RTL
        this.getView().addStyleClass("sapUiRTL");
    }
});
```

### CSS RTL-Aware
```css
/* ✅ CORRECTO: Estilos que respetan RTL */
.myContainer {
    /* Usar logical properties */
    margin-inline-start: 1rem;
    margin-inline-end: 2rem;
    padding-inline: 1rem;
}

/* ✅ CORRECTO: Evitar left/right específicos */
.myButton {
    /* En lugar de margin-left: 1rem; */
    margin-inline-start: 1rem;
}

/* ✅ CORRECTO: Usar clases UI5 RTL-aware */
.sapUiMarginBegin {
    /* UI5 maneja automáticamente RTL */
}
```

### Iconos y Direccionalidad
```xml
<!-- ✅ CORRECTO: Iconos que se adaptan a RTL -->
<Button icon="sap-icon://navigation-right-arrow"
        text="{i18n>buttonNext}"
        iconFirst="false"/>

<!-- UI5 automáticamente invierte la dirección en RTL -->
```

## 4. Temas de Alto Contraste

### ✅ OBLIGATORIO: Soporte de Alto Contraste
```javascript
// ✅ CORRECTO: Detectar tema de alto contraste
sap.ui.define([
    "sap/ui/core/theming/Parameters"
], function(Parameters) {
    "use strict";
    
    // Verificar si es tema de alto contraste
    const sTheme = Parameters.get("sapUiTheme");
    const bHighContrast = sTheme.includes("hcb") || sTheme.includes("hcw");
    
    if (bHighContrast) {
        // Ajustes específicos para alto contraste
        this.getView().addStyleClass("sapUiHighContrast");
    }
});
```

### CSS para Alto Contraste
```css
/* ✅ CORRECTO: Estilos específicos para alto contraste */
.sapUiHighContrast .myCustomElement {
    border: 2px solid;
    background-color: transparent;
}

/* ✅ CORRECTO: Usar variables de tema UI5 */
.myElement {
    color: var(--sapTextColor);
    background-color: var(--sapBackgroundColor);
    border-color: var(--sapBorderColor);
}
```

## 5. Internacionalización (i18n)

### ✅ OBLIGATORIO: Uso de Archivos i18n
```javascript
// ✅ CORRECTO: Configuración de modelo i18n
// En Component.js
sap.ui.define([
    "sap/ui/core/UIComponent",
    "sap/ui/model/resource/ResourceModel"
], function(UIComponent, ResourceModel) {
    "use strict";
    
    return UIComponent.extend("com.myapp.Component", {
        
        init: function() {
            UIComponent.prototype.init.apply(this, arguments);
            
            // Configurar modelo i18n
            const oI18nModel = new ResourceModel({
                bundleName: "com.myapp.i18n.i18n",
                supportedLocales: ["en", "es", "de", "fr"],
                fallbackLocale: "en"
            });
            
            this.setModel(oI18nModel, "i18n");
        }
    });
});
```

### Estructura de Archivos i18n
```
webapp/
├── i18n/
│   ├── i18n.properties          # Inglés (fallback)
│   ├── i18n_es.properties       # Español
│   ├── i18n_de.properties       # Alemán
│   ├── i18n_fr.properties       # Francés
│   └── i18n_en.properties       # Inglés explícito
```

### ✅ OBLIGATORIO: Todos los Textos en i18n
```properties
# i18n.properties
pageTitle=Customer Management
buttonSave=Save
buttonCancel=Cancel
messageSuccess=Data saved successfully
messageError=An error occurred while saving
fieldRequired=This field is required
```

```xml
<!-- ✅ CORRECTO: Usar binding i18n para todos los textos -->
<Page title="{i18n>pageTitle}">
    <content>
        <Button text="{i18n>buttonSave}" press="onSave"/>
        <Button text="{i18n>buttonCancel}" press="onCancel"/>
    </content>
</Page>
```

### Formateo de Números y Fechas
```javascript
// ✅ CORRECTO: Usar formatters localizados
sap.ui.define([
    "sap/ui/core/format/NumberFormat",
    "sap/ui/core/format/DateFormat"
], function(NumberFormat, DateFormat) {
    "use strict";
    
    return {
        
        formatCurrency: function(sValue, sCurrency) {
            const oCurrencyFormat = NumberFormat.getCurrencyInstance({
                currencyCode: false
            });
            return oCurrencyFormat.format(sValue, sCurrency);
        },
        
        formatDate: function(oDate) {
            const oDateFormat = DateFormat.getDateInstance({
                style: "medium"
            });
            return oDateFormat.format(oDate);
        }
    };
});
```

### Pluralización y Parámetros
```properties
# i18n.properties
itemCount=You have {0} item(s)
welcomeMessage=Welcome, {0}!
```

```javascript
// ✅ CORRECTO: Usar parámetros en textos
const sMessage = this.getResourceBundle().getText("itemCount", [iCount]);
const sWelcome = this.getResourceBundle().getText("welcomeMessage", [sUserName]);
```

## 6. Validación de Accesibilidad

### ✅ OBLIGATORIO: Testing de Accesibilidad
```javascript
// ✅ CORRECTO: Verificar accesibilidad programáticamente
sap.ui.define([
    "sap/ui/core/AccessibilitySupport"
], function(AccessibilitySupport) {
    "use strict";
    
    // Verificar si el soporte de accesibilidad está habilitado
    if (AccessibilitySupport.getAccessibilityEnabled()) {
        // Aplicar configuraciones específicas de accesibilidad
        this.enableAccessibilityFeatures();
    }
});
```

### Herramientas de Validación
```bash
# Verificar textos hardcodeados
grep -r "text=\"[^{]" webapp/view/ --include="*.xml"

# Verificar labels faltantes
grep -r "<Input\|<Button\|<ComboBox" webapp/view/ --include="*.xml" | grep -v "ariaLabel"
```

---

## Checklist de Compliance

### ✅ Accesibilidad
- [ ] Todos los controles tienen ARIA labels apropiados
- [ ] Navegación por teclado funciona completamente
- [ ] Soporte para lectores de pantalla verificado
- [ ] Temas de alto contraste soportados
- [ ] Orden de tabulación es lógico

### ✅ RTL Support
- [ ] Layout se adapta correctamente a RTL
- [ ] Iconos direccionales se invierten apropiadamente
- [ ] Texto y alineación respetan la dirección

### ✅ Internacionalización
- [ ] Todos los textos están en archivos i18n
- [ ] Múltiples idiomas configurados
- [ ] Formateo de números y fechas localizado
- [ ] Parámetros y pluralización implementados

---

**CRÍTICO**: El incumplimiento de estas reglas puede resultar en:
- Violaciones de accesibilidad y compliance legal
- Exclusión de usuarios con discapacidades
- Problemas en mercados internacionales
- Fallos en auditorías de accesibilidad