# SAPUI5 Security & Performance - Reglas de Seguridad y Rendimiento

> **APLICABILIDAD**: Todas las aplicaciones SAPUI5 FreeStyle
> **PRIORIDAD**: ⭐ CRÍTICO - Cumplimiento obligatorio al 100%
> **FUENTE**: SAP Security Guidelines y Performance Best Practices

## 1. Content Security Policy (CSP)

### ✅ OBLIGATORIO: CSP Compliance
```html
<!-- ✅ CORRECTO: CSP headers recomendados -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-eval'; 
               style-src 'self' 'unsafe-inline'; 
               img-src 'self' data:;">
```

### ❌ PROHIBIDO: Violaciones de CSP
```html
<!-- ❌ NUNCA usar scripts inline -->
<script>
    sap.ui.getCore().attachInit(function() {
        // Viola CSP
    });
</script>

<!-- ❌ NUNCA usar estilos inline -->
<div style="background-color: red;">Content</div>

<!-- ❌ NUNCA usar eval() o similares -->
<script>
    eval("var x = 1;"); // ❌ PROHIBIDO
</script>
```

### ✅ CORRECTO: Alternativas CSP-compliant
```javascript
// ✅ CORRECTO: Usar ComponentSupport
sap.ui.define([
    "sap/ui/core/ComponentSupport"
], function(ComponentSupport) {
    "use strict";
    // Código de inicialización
});

// ✅ CORRECTO: Estilos en archivos CSS separados
// En archivo .css
.myCustomClass {
    background-color: red;
}
```

## 2. Prevención de XSS (Cross-Site Scripting)

### ✅ OBLIGATORIO: Output Encoding
```javascript
// ✅ CORRECTO: Usar controles UI5 que encodean automáticamente
const oText = new Text({
    text: sUserInput  // UI5 encodea automáticamente
});

// ✅ CORRECTO: Encoding manual cuando sea necesario
sap.ui.define([
    "sap/base/security/encodeHTML"
], function(encodeHTML) {
    "use strict";
    
    const sSafeHTML = encodeHTML(sUserInput);
});
```

### ❌ PROHIBIDO: Inserción directa de HTML
```javascript
// ❌ NUNCA insertar HTML sin encoding
this.byId("myDiv").getDomRef().innerHTML = sUserInput;

// ❌ NUNCA usar jQuery para insertar HTML no seguro
$("#myDiv").html(sUserInput);

// ❌ NUNCA usar document.write
document.write(sUserInput);
```

### Validación de Entrada de Datos
```javascript
// ✅ OBLIGATORIO: Validar entrada del usuario
sap.ui.define([
    "sap/base/security/sanitizeHTML"
], function(sanitizeHTML) {
    "use strict";
    
    onInputChange: function(oEvent) {
        const sValue = oEvent.getParameter("value");
        
        // Validar formato
        if (!/^[a-zA-Z0-9\s]+$/.test(sValue)) {
            this.showError("Formato inválido");
            return;
        }
        
        // Sanitizar si es necesario
        const sSafeValue = sanitizeHTML(sValue);
        this.getModel().setProperty("/userInput", sSafeValue);
    }
});
```

## 3. Manejo Seguro de Datos Sensibles

### ✅ OBLIGATORIO: Protección de Datos
```javascript
// ✅ CORRECTO: No exponer datos sensibles en logs
console.log("User logged in:", sUserId); // ✅ OK - ID público
console.log("Processing data");          // ✅ OK - información general

// ❌ NUNCA loggear datos sensibles
console.log("Password:", sPassword);     // ❌ PROHIBIDO
console.log("Token:", sAuthToken);       // ❌ PROHIBIDO
```

### Gestión de Tokens y Autenticación
```javascript
// ✅ CORRECTO: Gestión segura de tokens
sap.ui.define([
    "sap/ui/model/odata/v4/ODataModel"
], function(ODataModel) {
    "use strict";
    
    // Token se maneja automáticamente por UI5
    const oModel = new ODataModel({
        serviceUrl: "/odata/v4/service/",
        synchronizationMode: "None"
    });
    
    // ✅ CORRECTO: Headers de autenticación seguros
    oModel.attachRequestSent(function(oEvent) {
        // UI5 maneja automáticamente CSRF tokens
    });
});
```

## 4. Optimización de Rendimiento

### Lazy Loading Obligatorio
```javascript
// ✅ OBLIGATORIO: Cargar módulos bajo demanda
sap.ui.define([
    "sap/ui/core/mvc/Controller"
], function(Controller) {
    "use strict";
    
    return Controller.extend("com.myapp.controller.Main", {
        
        onOpenDialog: function() {
            // ✅ CORRECTO: Cargar dialog solo cuando se necesite
            if (!this._oDialog) {
                sap.ui.require([
                    "sap/m/Dialog",
                    "sap/m/Button"
                ], function(Dialog, Button) {
                    this._oDialog = new Dialog({
                        title: "{i18n>dialogTitle}",
                        content: [/* contenido */]
                    });
                    this._oDialog.open();
                }.bind(this));
            } else {
                this._oDialog.open();
            }
        }
    });
});
```

### Optimización de Data Binding
```javascript
// ✅ CORRECTO: Usar $select para limitar datos
const oBinding = this.byId("myTable").getBinding("items");
oBinding.changeParameters({
    $select: "ID,Name,Status",  // Solo campos necesarios
    $top: 50                    // Limitar registros
});

// ✅ CORRECTO: Usar paginación
const oTable = this.byId("myTable");
oTable.setGrowingThreshold(20);
oTable.setGrowing(true);
```

### Gestión Eficiente de Memoria
```javascript
// ✅ OBLIGATORIO: Limpiar recursos en onExit
onExit: function() {
    // Limpiar modelos
    if (this._oLocalModel) {
        this._oLocalModel.destroy();
        this._oLocalModel = null;
    }
    
    // Limpiar dialogs
    if (this._oDialog) {
        this._oDialog.destroy();
        this._oDialog = null;
    }
    
    // Limpiar timers
    if (this._iTimer) {
        clearInterval(this._iTimer);
        this._iTimer = null;
    }
}
```

## 5. Manejo de Errores Estructurado

### ✅ OBLIGATORIO: Error Handling Robusto
```javascript
// ✅ CORRECTO: Manejo de errores en llamadas OData
sap.ui.define([
    "sap/m/MessageBox"
], function(MessageBox) {
    "use strict";
    
    onSaveData: function() {
        const oModel = this.getView().getModel();
        
        oModel.submitBatch("updateGroup").then(function() {
            MessageToast.show(this.getResourceBundle().getText("dataSavedSuccess"));
        }.bind(this)).catch(function(oError) {
            // ✅ CORRECTO: Log técnico + mensaje usuario
            console.error("Error saving data:", oError);
            
            MessageBox.error(this.getResourceBundle().getText("dataSaveError"));
        }.bind(this));
    }
});
```

### Logging Estructurado
```javascript
// ✅ CORRECTO: Usar logging de UI5
sap.ui.define([
    "sap/base/Log"
], function(Log) {
    "use strict";
    
    // Diferentes niveles de log
    Log.info("Application started");
    Log.warning("Deprecated API used");
    Log.error("Failed to load data", oError);
    Log.debug("Processing item", oItem);
});
```

## 6. Optimización de Llamadas al Servidor

### Batch Requests Obligatorios
```javascript
// ✅ OBLIGATORIO: Agrupar llamadas en batches
const oModel = this.getView().getModel();

// Configurar batch groups
oModel.setDeferredGroups(["updateGroup"]);

// Realizar múltiples operaciones
oModel.create("/Items", oNewItem, {
    groupId: "updateGroup"
});
oModel.update("/Items(1)", oUpdatedItem, {
    groupId: "updateGroup"
});

// Enviar batch
oModel.submitChanges({
    groupId: "updateGroup"
});
```

### Caché Inteligente
```javascript
// ✅ CORRECTO: Configurar caché apropiado
const oModel = new ODataModel({
    serviceUrl: "/odata/v4/service/",
    defaultBindingMode: "TwoWay",
    defaultCountMode: "Inline",
    autoExpandSelect: true,
    operationMode: "Server"  // Para grandes volúmenes
});
```

## 7. Validación de Datos del Cliente

### ✅ OBLIGATORIO: Validación Robusta
```javascript
// ✅ CORRECTO: Validación completa
sap.ui.define([
    "sap/ui/core/message/MessageManager",
    "sap/ui/core/MessageType"
], function(MessageManager, MessageType) {
    "use strict";
    
    validateInput: function(sValue, sFieldName) {
        const oMessageManager = MessageManager.getInstance();
        
        // Limpiar mensajes previos
        oMessageManager.removeAllMessages();
        
        // Validaciones específicas
        if (!sValue || sValue.trim() === "") {
            this.addValidationMessage(sFieldName, 
                this.getResourceBundle().getText("fieldRequired"), MessageType.Error);
            return false;
        }
        
        if (sValue.length > 100) {
            this.addValidationMessage(sFieldName, 
                this.getResourceBundle().getText("fieldMaxLength"), MessageType.Error);
            return false;
        }
        
        return true;
    }
});
```

---

## Herramientas de Validación

### Verificación de Seguridad
```bash
# Verificar violaciones de CSP
grep -r "eval\|innerHTML\|document.write" src/ --include="*.js"

# Verificar logs de datos sensibles
grep -r "console.log.*password\|console.log.*token" src/ --include="*.js"
```

### Performance Monitoring
```javascript
// ✅ CORRECTO: Monitoreo de rendimiento
sap.ui.define([
    "sap/ui/performance/Measurement"
], function(Measurement) {
    "use strict";
    
    // Medir operaciones críticas
    Measurement.start("dataLoad");
    // ... operación
    Measurement.end("dataLoad");
});
```

---

**CRÍTICO**: El incumplimiento de estas reglas puede resultar en:
- Vulnerabilidades de seguridad
- Problemas de rendimiento en producción
- Violaciones de compliance
- Experiencia de usuario degradada