# SAPUI5 Formatters & Data Binding - Reglas de Formateo y Vinculación de Datos

> **APLICABILIDAD**: Todas las aplicaciones SAPUI5 FreeStyle
> **PRIORIDAD**: ⭐ CRÍTICO - Cumplimiento obligatorio al 100%
> **VERSIÓN MÍNIMA UI5**: 1.96+ (LTS o superior)
> **COMPATIBILIDAD BTP**: ✅ Cloud Foundry, ✅ Kyma Runtime
> **FUENTE**: SAP UI5 Official Documentation (Verificado)

## 📌 Compatibilidad y Requisitos

### Versiones UI5
- **Mínimo Recomendado**: UI5 1.96+ (LTS)
- **Producción BTP**: Usar siempre la última versión LTS disponible en SAP BTP
- **Verificar Versión**: [SAP UI5 Version Overview](https://ui5.sap.com/versionoverview.html)

### Características Modernas Requeridas
- ✅ `core:require` syntax (UI5 1.58+)
- ✅ ES6+ JavaScript support
- ✅ Async component loading
- ✅ OData V4 support (para data types)

### Compatibilidad BTP
- ✅ Cloud Foundry (Recomendado)
- ✅ Kyma Runtime
- ✅ HTML5 Application Repository
- ✅ XSUAA Authentication

## 🔗 Ver También

### Deployment en BTP
- 📋 [SAPUI5-BTP-Deployment.md](./SAPUI5-BTP-Deployment.md) - Configuración completa de deployment en BTP
- 🔒 [SAPUI5-Security-Performance.md](./SAPUI5-Security-Performance.md) - Seguridad y performance

### Desarrollo
- 🔧 [SAPUI5-Core-Standards.md](./SAPUI5-Core-Standards.md) - Estándares fundamentales y MVC
- 🎨 [SAPUI5-Design-Controls.md](./SAPUI5-Design-Controls.md) - Controles y diseño
- 🛠️ [SAPUI5-Development-Tools.md](./SAPUI5-Development-Tools.md) - Testing y herramientas

### Calidad
- ♿ [SAPUI5-Accessibility-i18n.md](./SAPUI5-Accessibility-i18n.md) - Accesibilidad e internacionalización
- 🧭 [SAPUI5-Routing-Navigation.md](./SAPUI5-Routing-Navigation.md) - Routing y navegación

## 1. Formatters - Reglas Fundamentales

### ✅ OBLIGATORIO: Ubicación Estándar

```javascript
// webapp/model/formatter.js
sap.ui.define([], function() {
    "use strict";
    
    return {
        /**
         * Formatea el nombre capitalizando la primera letra
         * @param {string} sName - Nombre a formatear
         * @returns {string} Nombre formateado
         * @public
         */
        upperFirstLetter: function(sName) {
            if (!sName) {
                return "";
            }
            return sName.charAt(0).toUpperCase() + sName.slice(1);
        }
    };
});
```

**JUSTIFICACIÓN OFICIAL SAP:**
> "We recommend using a separate `formatter.js` file to group all your formatter functions, making them accessible throughout your app."

### ❌ PROHIBIDO: Formatters Inline

```xml
<!-- ❌ NUNCA usar expression binding para lógica compleja -->
<Text text="{= ${status} === 'PENDING' ? 'En Proceso' : 'Completado' }"/>

<!-- ❌ NUNCA definir formatters directamente en la vista -->
<Text text="{path: 'name', formatter: function(v){return v.toUpperCase();}}"/>
```

### ✅ CORRECTO: Uso de Formatters

```xml
<!-- ✅ Método moderno con core:require -->
<mvc:View
    xmlns="sap.m"
    xmlns:core="sap.ui.core"
    xmlns:mvc="sap.ui.core.mvc"
    core:require="{
        Formatter: 'com/myapp/model/formatter'
    }">
    
    <Text text="{
        path: 'name',
        formatter: 'Formatter.upperFirstLetter'
    }"/>
</mvc:View>

<!-- ✅ Método tradicional con dot notation (controller) -->
<mvc:View
    controllerName="com.myapp.controller.Main"
    xmlns="sap.m"
    xmlns:mvc="sap.ui.core.mvc">
    
    <Text text="{
        path: 'name',
        formatter: '.upperFirstLetter'
    }"/>
</mvc:View>
```

---

## 2. Contexto `this` en Formatters - CRÍTICO

### Regla Oficial SAP

**DOCUMENTACIÓN OFICIAL:**
> "By default, formatter functions are bound to the control instance unless explicitly specified otherwise. However, when a formatter is accessed via dot notation, the `this` context is bound to the parent object."

### Escenarios de Contexto `this`

#### Escenario A: Formatter con `core:require`
```javascript
// webapp/model/formatter.js
return {
    statusText: function(sStatus) {
        // 'this' = objeto formatter (Formatter)
        return sStatus === "OK" ? "Éxito" : "Error";
    }
};
```

```xml
<Text text="{
    path: 'status',
    formatter: 'Formatter.statusText'
}"/>
<!-- 'this' dentro del formatter = Formatter object -->
```

#### Escenario B: Formatter en Controller con Dot Notation
```javascript
// Controller.js
sap.ui.define([
    "sap/ui/core/mvc/Controller",
    "../model/formatter"
], function(Controller, formatter) {
    "use strict";
    
    return Controller.extend("com.myapp.controller.Main", {
        formatter: formatter,
        
        onInit: function() {
            // 'this' = controller instance
        }
    });
});
```

```xml
<Text text="{
    path: 'status',
    formatter: '.formatter.statusText'
}"/>
<!-- 'this' dentro del formatter = controller instance -->
```

#### Escenario C: Forzar Contexto con `.bind()`

**SINTAXIS OFICIAL:**
```xml
<!-- Forzar 'this' al control -->
<Text text="{
    path: 'name',
    formatter: 'Formatter.upperFirstLetter.bind($control)'
}"/>

<!-- Forzar 'this' al controller -->
<Text text="{
    path: 'name',
    formatter: 'Formatter.upperFirstLetter.bind($controller)'
}"/>

<!-- Usar alias de core:require -->
<mvc:View core:require="{
    Formatter: 'com/myapp/model/formatter',
    Helper: 'com/myapp/util/Helper'
}">
    <Text text="{
        path: 'name',
        formatter: 'Formatter.upperFirstLetter.bind(Helper)'
    }"/>
</mvc:View>
```

**ARGUMENTOS VÁLIDOS para `.bind()`:**
- `$control` - Vincula al control instance
- `$controller` - Vincula al controller
- Cualquier alias definido en `core:require`

**⚠️ RESTRICCIÓN:**
> "Arguments other than `$control` and `$controller` must not start with a '$' character as this prefix is reserved by the framework."

---

## 3. Function Expressions vs Arrow Functions

### ✅ RECOMENDADO: Function Expressions

```javascript
// webapp/model/formatter.js
sap.ui.define([], function() {
    "use strict";
    
    return {
        // ✅ CORRECTO: Function expression
        statusState: function(sStatus) {
            // 'this' está disponible según el contexto
            const mStates = {
                "PENDING": "Warning",
                "VALIDATED": "Success",
                "REJECTED": "Error"
            };
            return mStates[sStatus] || "None";
        }
    };
});
```

### ⚠️ LIMITADO: Arrow Functions

```javascript
// webapp/model/formatter.js
sap.ui.define([], function() {
    "use strict";
    
    return {
        // ⚠️ CUIDADO: Arrow function
        statusState: (sStatus) => {
            // 'this' NO está disponible (lexical scope)
            // Solo usar si NO necesitas acceso a 'this'
            return sStatus === "OK" ? "Success" : "Error";
        }
    };
});
```

**CUÁNDO USAR CADA UNO:**
- **Function expression**: Cuando necesitas acceso a `this` (controller, control, etc.)
- **Arrow function**: Solo para formatters puros sin dependencias de contexto

---

## 4. Formatters vs Data Types - Decisión

### Tabla de Decisión

| Característica | Formatter | Data Type |
|----------------|-----------|-----------|
| **Dirección** | One-way (modelo → vista) | Two-way (modelo ↔ vista) |
| **Parsing** | ❌ No | ✅ Sí |
| **Validación** | ❌ No | ✅ Sí |
| **Uso típico** | Presentación simple | Entrada de usuario |
| **Ejemplo** | Formatear estado, color | Input numérico, fechas |

**REGLA OFICIAL SAP:**
> "When using formatter functions, the binding is automatically switched to 'one-way'. So you can't use a formatter function for 'two-way' scenarios, but you can use Data Types."

### ✅ Usar Formatter Cuando:
- Solo necesitas formatear para visualización
- No hay entrada de usuario
- Lógica de presentación simple
- Conversión unidireccional

### ✅ Usar Data Type Cuando:
- Hay entrada de usuario (Input, DatePicker, etc.)
- Necesitas validación
- Necesitas parsing (texto → modelo)
- Two-way binding requerido

---

## 5. Data Types - Uso Correcto

### Tipos Estándar SAPUI5

```xml
<mvc:View
    xmlns="sap.m"
    xmlns:core="sap.ui.core"
    xmlns:mvc="sap.ui.core.mvc"
    core:require="{
        Integer: 'sap/ui/model/type/Integer',
        Float: 'sap/ui/model/type/Float',
        Currency: 'sap/ui/model/type/Currency',
        DateType: 'sap/ui/model/type/Date'
    }">
    
    <!-- Integer con constraints -->
    <Input value="{
        path: '/quantity',
        type: 'Integer',
        constraints: {
            minimum: 0,
            maximum: 1000
        }
    }"/>
    
    <!-- Float con format options -->
    <Input value="{
        path: '/price',
        type: 'Float',
        formatOptions: {
            minFractionDigits: 2,
            maxFractionDigits: 2
        }
    }"/>
    
    <!-- Currency -->
    <Text text="{
        parts: [
            {path: '/amount'},
            {path: '/currency'}
        ],
        type: 'Currency',
        formatOptions: {
            showMeasure: true
        }
    }"/>
</mvc:View>
```

### Tipos OData

```xml
<!-- OData V4 -->
<mvc:View
    xmlns="sap.m"
    xmlns:core="sap.ui.core"
    xmlns:mvc="sap.ui.core.mvc"
    core:require="{
        DateTimeOffset: 'sap/ui/model/odata/type/DateTimeOffset'
    }">
    
    <!-- Tipo automático desde metadata -->
    <DateTimePicker value="{/CreatedAt}"/>
    
    <!-- Tipo explícito para JSON model -->
    <DateTimePicker value="{
        path: 'json>/CreatedAt',
        type: 'DateTimeOffset',
        constraints: {V4: true}
    }"/>
</mvc:View>
```

---

## 6. Patrones Comunes de Formateo

### Estado/Status

```javascript
// webapp/model/formatter.js
return {
    /**
     * Formatea estado a color semántico
     * @param {string} sStatus - Estado
     * @returns {string} Estado semántico (Success, Warning, Error, None)
     * @public
     */
    statusState: function(sStatus) {
        const mStatusMap = {
            "APPROVED": "Success",
            "PENDING": "Warning",
            "REJECTED": "Error",
            "DRAFT": "Information"
        };
        return mStatusMap[sStatus] || "None";
    },
    
    /**
     * Formatea estado a icono
     * @param {string} sStatus - Estado
     * @returns {string} URI del icono
     * @public
     */
    statusIcon: function(sStatus) {
        const mIconMap = {
            "APPROVED": "sap-icon://accept",
            "PENDING": "sap-icon://pending",
            "REJECTED": "sap-icon://decline",
            "DRAFT": "sap-icon://edit"
        };
        return mIconMap[sStatus] || "sap-icon://question-mark";
    }
};
```

### Números y Monedas

```javascript
return {
    /**
     * Formatea número con separador de miles
     * @param {number} nValue - Número
     * @returns {string} Número formateado
     * @public
     */
    numberFormat: function(nValue) {
        if (nValue == null) {
            return "0";
        }
        return nValue.toLocaleString();
    },
    
    /**
     * Formatea a millones
     * @param {number} fValue - Valor
     * @returns {string} Valor en millones
     * @public
     */
    toMillions: function(fValue) {
        if (!fValue) {
            return "0 M";
        }
        return `${Math.floor(fValue / 1000000)} M`;
    }
};
```

### Booleanos

```javascript
return {
    /**
     * Formatea booleano a texto
     * @param {boolean} bValue - Valor booleano
     * @returns {string} Texto (Sí/No)
     * @public
     */
    booleanText: function(bValue) {
        return bValue ? "Sí" : "No";
    },
    
    /**
     * Formatea booleano a icono
     * @param {boolean} bValue - Valor booleano
     * @returns {string} URI del icono
     * @public
     */
    booleanIcon: function(bValue) {
        return bValue ? "sap-icon://accept" : "sap-icon://decline";
    },
    
    /**
     * Invierte booleano para visibilidad
     * @param {boolean} bValue - Valor booleano
     * @returns {boolean} Valor invertido
     * @public
     */
    invertBoolean: function(bValue) {
        return !bValue;
    }
};
```

### Fechas (Usar Data Types preferentemente)

```javascript
return {
    /**
     * Formatea fecha a formato corto
     * @param {Date} oDate - Fecha
     * @returns {string} Fecha formateada
     * @public
     */
    dateShort: function(oDate) {
        if (!oDate) {
            return "";
        }
        const oDateFormat = sap.ui.core.format.DateFormat.getDateInstance({
            style: "short"
        });
        return oDateFormat.format(oDate);
    }
};
```

---

## 7. Composite Binding - Múltiples Parámetros

### Ejemplo: Nombre Completo

```javascript
// webapp/model/formatter.js
return {
    /**
     * Combina nombre y apellido
     * @param {string} sFirstName - Nombre
     * @param {string} sLastName - Apellido
     * @returns {string} Nombre completo
     * @public
     */
    fullName: function(sFirstName, sLastName) {
        if (!sFirstName && !sLastName) {
            return "";
        }
        return `${sFirstName || ""} ${sLastName || ""}`.trim();
    }
};
```

```xml
<Text text="{
    parts: [
        {path: 'firstName'},
        {path: 'lastName'}
    ],
    formatter: 'Formatter.fullName'
}"/>
```

### Ejemplo: Dirección Completa

```javascript
return {
    /**
     * Formatea dirección completa
     * @param {string} sStreet - Calle
     * @param {string} sCity - Ciudad
     * @param {string} sZip - Código postal
     * @returns {string} Dirección formateada
     * @public
     */
    fullAddress: function(sStreet, sCity, sZip) {
        const aParts = [sStreet, sCity, sZip].filter(Boolean);
        return aParts.join(", ");
    }
};
```

---

## 8. Validación y Manejo de Errores

### ✅ OBLIGATORIO: Validar Parámetros

```javascript
return {
    /**
     * Formatea precio con validación
     * @param {number} nPrice - Precio
     * @returns {string} Precio formateado
     * @public
     */
    formatPrice: function(nPrice) {
        // ✅ OBLIGATORIO: Validar parámetros
        if (nPrice == null || isNaN(nPrice)) {
            return "0.00";
        }
        
        // ✅ OBLIGATORIO: Manejar casos edge
        if (nPrice < 0) {
            return `(${Math.abs(nPrice).toFixed(2)})`;
        }
        
        return nPrice.toFixed(2);
    },
    
    /**
     * Formatea porcentaje con validación
     * @param {number} fValue - Valor decimal (0.15 = 15%)
     * @returns {string} Porcentaje formateado
     * @public
     */
    formatPercentage: function(fValue) {
        if (fValue == null || isNaN(fValue)) {
            return "0%";
        }
        return `${(fValue * 100).toFixed(1)}%`;
    }
};
```

---

## 9. Patrones Prohibidos

### ❌ NUNCA: Lógica de Negocio en Formatters

```javascript
// ❌ INCORRECTO: Lógica de negocio
return {
    calculateDiscount: function(nPrice, nQuantity) {
        // ❌ Esto es lógica de negocio, NO formateo
        if (nQuantity > 10) {
            return nPrice * 0.9; // 10% descuento
        }
        return nPrice;
    }
};

// ✅ CORRECTO: Solo formateo
return {
    formatDiscount: function(nDiscountAmount) {
        // ✅ Solo formatea el descuento ya calculado
        if (!nDiscountAmount) {
            return "Sin descuento";
        }
        return `-${nDiscountAmount.toFixed(2)}`;
    }
};
```

### ❌ NUNCA: Side Effects en Formatters

```javascript
// ❌ INCORRECTO: Modificar estado
return {
    formatAndCount: function(sValue) {
        this.counter++; // ❌ Side effect
        return sValue.toUpperCase();
    }
};

// ✅ CORRECTO: Función pura
return {
    formatValue: function(sValue) {
        // ✅ Sin side effects
        return sValue ? sValue.toUpperCase() : "";
    }
};
```

### ❌ NUNCA: Llamadas Asíncronas en Formatters

```javascript
// ❌ INCORRECTO: Operaciones asíncronas
return {
    formatWithAPI: function(sId) {
        // ❌ NUNCA hacer llamadas async
        fetch(`/api/data/${sId}`).then(...);
        return "Loading...";
    }
};

// ✅ CORRECTO: Datos ya disponibles
return {
    formatData: function(oData) {
        // ✅ Formatear datos ya cargados
        return oData ? oData.displayName : "";
    }
};
```

---

## 10. Mejores Prácticas de Implementación

### ✅ OBLIGATORIO: JSDoc Completo

```javascript
sap.ui.define([], function() {
    "use strict";
    
    return {
        /**
         * Formatea el estado de validación a un estado semántico
         * 
         * @param {string} sStatus - El estado de validación
         *   Valores permitidos: "PENDING", "VALIDATED", "REJECTED"
         * @returns {string} El estado semántico para ObjectStatus
         *   Valores posibles: "Success", "Warning", "Error", "None"
         * @public
         * @example
         * // Uso en XML View
         * <ObjectStatus state="{path: 'status', formatter: '.statusState'}"/>
         */
        statusState: function(sStatus) {
            const mStates = {
                "PENDING": "Warning",
                "VALIDATED": "Success",
                "REJECTED": "Error"
            };
            return mStates[sStatus] || "None";
        }
    };
});
```

### ✅ OBLIGATORIO: Manejo de Null/Undefined

```javascript
return {
    /**
     * Formatea texto con manejo seguro de null
     * @param {string} sText - Texto a formatear
     * @returns {string} Texto formateado
     * @public
     */
    safeFormat: function(sText) {
        // ✅ Verificar null/undefined
        if (sText == null) {
            return "";
        }
        
        // ✅ Verificar tipo
        if (typeof sText !== "string") {
            return String(sText);
        }
        
        return sText.trim();
    }
};
```

### ✅ RECOMENDADO: Constantes para Mapeos

```javascript
sap.ui.define([], function() {
    "use strict";
    
    // ✅ CORRECTO: Constantes al inicio
    const STATUS_MAP = {
        "PENDING": "Warning",
        "VALIDATED": "Success",
        "REJECTED": "Error"
    };
    
    const ICON_MAP = {
        "PENDING": "sap-icon://pending",
        "VALIDATED": "sap-icon://accept",
        "REJECTED": "sap-icon://decline"
    };
    
    return {
        statusState: function(sStatus) {
            return STATUS_MAP[sStatus] || "None";
        },
        
        statusIcon: function(sStatus) {
            return ICON_MAP[sStatus] || "sap-icon://question-mark";
        }
    };
});
```

---

## 11. Testing de Formatters

### ✅ OBLIGATORIO: Unit Tests

```javascript
// test/unit/model/formatter.test.js
sap.ui.define([
    "com/myapp/model/formatter"
], function(formatter) {
    "use strict";

    QUnit.module("Formatter Tests");

    QUnit.test("statusState - should return correct state", function(assert) {
        // Arrange & Act & Assert
        assert.strictEqual(formatter.statusState("PENDING"), "Warning", "PENDING returns Warning");
        assert.strictEqual(formatter.statusState("VALIDATED"), "Success", "VALIDATED returns Success");
        assert.strictEqual(formatter.statusState("REJECTED"), "Error", "REJECTED returns Error");
        assert.strictEqual(formatter.statusState("UNKNOWN"), "None", "Unknown status returns None");
        assert.strictEqual(formatter.statusState(null), "None", "Null returns None");
    });

    QUnit.test("numberFormat - should format numbers correctly", function(assert) {
        assert.strictEqual(formatter.numberFormat(1000), "1,000", "Formats with thousands separator");
        assert.strictEqual(formatter.numberFormat(0), "0", "Handles zero");
        assert.strictEqual(formatter.numberFormat(null), "0", "Handles null");
    });
});
```

---

## 12. Checklist de Compliance

### ✅ Antes de cada Commit

- [ ] Todos los formatters están en `webapp/model/formatter.js`
- [ ] No hay formatters inline en XML views
- [ ] Todos los formatters tienen JSDoc completo
- [ ] Se usa `function()` en lugar de arrow functions (cuando se necesita `this`)
- [ ] Validación de parámetros null/undefined
- [ ] No hay lógica de negocio en formatters
- [ ] No hay side effects en formatters
- [ ] Tests unitarios para formatters críticos

### ✅ Antes de cada Release

- [ ] Todos los formatters tienen tests
- [ ] Documentación actualizada
- [ ] Performance verificado (no operaciones costosas)
- [ ] Compatibilidad con i18n verificada
- [ ] Uso correcto de Data Types vs Formatters

---

## 13. Recursos y Referencias

### Documentación Oficial SAP Consultada

- ✅ **Property Binding** - `/sapui5/04_Essentials/property-binding-91f0652`
- ✅ **Formatting, Parsing, and Validating Data** - `/sapui5/04_Essentials/formatting-parsing-and-validating-data-07e4b92`
- ✅ **Dates, Times, Timestamps** - `/sapui5/04_Essentials/dates-times-timestamps-and-time-zones-6c9e61d`

### Herramientas de Validación

```bash
# Verificar formatters inline en XML
grep -r "formatter: function" webapp/view/ --include="*.xml"

# Verificar uso de arrow functions
grep -r "formatter:.*=>" webapp/model/formatter.js

# Verificar JSDoc
grep -B5 "function(" webapp/model/formatter.js | grep -c "@param"
```

---

**IMPORTANTE**: Estas reglas están basadas 100% en documentación oficial de SAP UI5 y han sido verificadas contra las mejores prácticas actuales. El cumplimiento de estas reglas garantiza:

- ✅ Código mantenible y escalable
- ✅ Compatibilidad con actualizaciones de UI5
- ✅ Performance óptimo
- ✅ Facilidad de testing
- ✅ Compliance con estándares SAP

**ÚLTIMA ACTUALIZACIÓN**: Octubre 2025 - Basado en UI5 1.120+ y documentación oficial SAP