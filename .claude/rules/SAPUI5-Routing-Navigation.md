# SAPUI5 Routing & Navigation - Reglas de Enrutamiento y Navegación

> **APLICABILIDAD**: Todas las aplicaciones SAPUI5 FreeStyle
> **PRIORIDAD**: ⭐ CRÍTICO - Cumplimiento obligatorio al 100%
> **FUENTE**: SAP UI5 Router Documentation y BTP Best Practices

## 1. Configuración de Routing en Manifest.json

### ✅ OBLIGATORIO: Configuración Básica de Router
```json
// manifest.json - sap.ui5/routing
{
  "sap.ui5": {
    "routing": {
      "config": {
        "routerClass": "sap.m.routing.Router",
        "type": "View",
        "viewType": "XML",
        "path": "com.myapp.view",
        "controlId": "app",
        "controlAggregation": "pages",
        "transition": "slide",
        "bypassed": {
          "target": "notFound"
        },
        "async": true
      },
      "routes": [
        {
          "pattern": "",
          "name": "main",
          "target": "main"
        },
        {
          "pattern": "detail/{objectId}",
          "name": "detail",
          "target": "detail"
        }
      ],
      "targets": {
        "main": {
          "id": "main",
          "name": "Main"
        },
        "detail": {
          "id": "detail",
          "name": "Detail"
        },
        "notFound": {
          "id": "notFound",
          "name": "NotFound"
        }
      }
    }
  }
}
```

### ✅ OBLIGATORIO: Configuración para BTP
```json
// manifest.json - Configuración específica para BTP
{
  "sap.app": {
    "crossNavigation": {
      "inbounds": {
        "intent1": {
          "signature": {
            "parameters": {},
            "additionalParameters": "allowed"
          },
          "semanticObject": "MyApp",
          "action": "display"
        }
      }
    }
  },
  "sap.cloud": {
    "public": true,
    "service": "myapp.service"
  }
}
```

## 2. Patrones de Navegación Estándar

### ✅ OBLIGATORIO: BaseController para Navegación
```javascript
// controller/BaseController.js
sap.ui.define([
    "sap/ui/core/mvc/Controller",
    "sap/ui/core/routing/History"
], function(Controller, History) {
    "use strict";

    return Controller.extend("com.myapp.controller.BaseController", {

        /**
         * Obtiene el router de la aplicación
         * @returns {sap.ui.core.routing.Router} Router instance
         * @public
         */
        getRouter: function() {
            return this.getOwnerComponent().getRouter();
        },

        /**
         * Navega a una ruta específica
         * @param {string} sRouteName - Nombre de la ruta
         * @param {object} oParameters - Parámetros de la ruta
         * @param {boolean} bReplace - Si reemplazar la entrada del historial
         * @public
         */
        navTo: function(sRouteName, oParameters, bReplace) {
            this.getRouter().navTo(sRouteName, oParameters, bReplace);
        },

        /**
         * Navegación hacia atrás inteligente
         * @param {string} sFallbackRoute - Ruta de fallback
         * @param {object} oFallbackParameters - Parámetros de fallback
         * @public
         */
        onNavBack: function(sFallbackRoute, oFallbackParameters) {
            const oHistory = History.getInstance();
            const sPreviousHash = oHistory.getPreviousHash();

            if (sPreviousHash !== undefined) {
                window.history.go(-1);
            } else {
                this.navTo(sFallbackRoute || "main", oFallbackParameters, true);
            }
        }
    });
});
```

### ✅ OBLIGATORIO: Inicialización del Router
```javascript
// Component.js
sap.ui.define([
    "sap/ui/core/UIComponent"
], function(UIComponent) {
    "use strict";

    return UIComponent.extend("com.myapp.Component", {

        init: function() {
            UIComponent.prototype.init.apply(this, arguments);

            // ✅ OBLIGATORIO: Inicializar router
            this.getRouter().initialize();
        }
    });
});
```

## 3. Manejo de Parámetros de Ruta

### ✅ OBLIGATORIO: Parámetros Tipados
```javascript
// controller/Detail.controller.js
sap.ui.define([
    "./BaseController"
], function(BaseController) {
    "use strict";

    return BaseController.extend("com.myapp.controller.Detail", {

        onInit: function() {
            // ✅ CORRECTO: Attachar a eventos de routing
            this.getRouter().getRoute("detail").attachPatternMatched(this._onObjectMatched, this);
        },

        /**
         * Maneja la coincidencia de patrón de ruta
         * @param {sap.ui.base.Event} oEvent - Evento de routing
         * @private
         */
        _onObjectMatched: function(oEvent) {
            const sObjectId = oEvent.getParameter("arguments").objectId;
            
            // ✅ OBLIGATORIO: Validar parámetros
            if (!sObjectId) {
                this.navTo("main");
                return;
            }

            // ✅ CORRECTO: Decodificar parámetros
            const sDecodedId = decodeURIComponent(sObjectId);
            this._bindView(sDecodedId);
        },

        /**
         * Vincula la vista con el objeto específico
         * @param {string} sObjectId - ID del objeto
         * @private
         */
        _bindView: function(sObjectId) {
            const sObjectPath = this.getModel().createKey("/Objects", {
                ID: sObjectId
            });

            this.getView().bindElement({
                path: sObjectPath,
                events: {
                    change: this._onBindingChange.bind(this),
                    dataRequested: function() {
                        this.getView().setBusy(true);
                    }.bind(this),
                    dataReceived: function() {
                        this.getView().setBusy(false);
                    }.bind(this)
                }
            });
        }
    });
});
```

## 4. Deep Linking y Bookmarking

### ✅ OBLIGATORIO: URLs Amigables
```javascript
// Configuración de rutas para deep linking
{
  "routes": [
    {
      "pattern": "products/{productId}/details/{tab}",
      "name": "productDetail",
      "target": "productDetail"
    },
    {
      "pattern": "orders/{orderId}/items/{itemId}",
      "name": "orderItem",
      "target": "orderItem"
    }
  ]
}
```

### ✅ OBLIGATORIO: Query Parameters
```javascript
// Manejo de query parameters
onNavigateWithQuery: function() {
    this.getRouter().navTo("productDetail", {
        productId: "123",
        tab: "specifications"
    }, false, {
        filter: "active",
        sort: "name",
        page: "2"
    });
}

// En el controller de destino
_onObjectMatched: function(oEvent) {
    const oArgs = oEvent.getParameter("arguments");
    const oQuery = oEvent.getParameter("config").queryParams;
    
    // Usar parámetros de ruta
    const sProductId = oArgs.productId;
    const sTab = oArgs.tab;
    
    // Usar query parameters
    const sFilter = oQuery.filter;
    const sSort = oQuery.sort;
    const iPage = parseInt(oQuery.page) || 1;
}
```

## 5. Navegación Cross-App (BTP)

### ✅ OBLIGATORIO: Configuración Cross-Navigation
```json
// manifest.json - Cross-navigation para BTP
{
  "sap.app": {
    "crossNavigation": {
      "inbounds": {
        "display": {
          "semanticObject": "Product",
          "action": "display",
          "signature": {
            "parameters": {
              "productId": {
                "required": true
              }
            }
          }
        }
      },
      "outbounds": {
        "toOrderApp": {
          "semanticObject": "Order",
          "action": "create",
          "parameters": {
            "productId": {
              "value": "{productId}"
            }
          }
        }
      }
    }
  }
}
```

### ✅ OBLIGATORIO: Cross-Navigation Service
```javascript
// Navegación entre aplicaciones en BTP
sap.ui.define([
    "sap/ushell/services/CrossApplicationNavigation"
], function(CrossApplicationNavigation) {
    "use strict";

    return {
        /**
         * Navega a otra aplicación en BTP
         * @param {string} sSemanticObject - Objeto semántico
         * @param {string} sAction - Acción
         * @param {object} oParameters - Parámetros
         * @public
         */
        navigateToApp: function(sSemanticObject, sAction, oParameters) {
            const oCrossAppNav = sap.ushell.Container.getService("CrossApplicationNavigation");
            
            const sHref = oCrossAppNav.hrefForExternal({
                target: {
                    semanticObject: sSemanticObject,
                    action: sAction
                },
                params: oParameters
            });

            oCrossAppNav.toExternal({
                target: {
                    shellHash: sHref
                }
            });
        }
    };
});
```

## 6. Manejo de Errores de Navegación

### ✅ OBLIGATORIO: Error Handling
```javascript
// controller/App.controller.js
sap.ui.define([
    "./BaseController",
    "sap/m/MessageBox"
], function(BaseController, MessageBox) {
    "use strict";

    return BaseController.extend("com.myapp.controller.App", {

        onInit: function() {
            // ✅ OBLIGATORIO: Manejar errores de routing
            this.getRouter().attachBypassed(this._onBypassed, this);
            this.getRouter().attachRouteMatched(this._onRouteMatched, this);
        },

        /**
         * Maneja rutas no encontradas
         * @param {sap.ui.base.Event} oEvent - Evento de bypass
         * @private
         */
        _onBypassed: function(oEvent) {
            const sHash = oEvent.getParameter("hash");
            
            MessageBox.information(
                this.getResourceBundle().getText("routeNotFound", [sHash]),
                {
                    title: this.getResourceBundle().getText("pageNotFoundTitle"),
                    onClose: function() {
                        this.navTo("main");
                    }.bind(this)
                }
            );
        },

        /**
         * Maneja coincidencias de ruta exitosas
         * @param {sap.ui.base.Event} oEvent - Evento de coincidencia
         * @private
         */
        _onRouteMatched: function(oEvent) {
            const sRouteName = oEvent.getParameter("name");
            
            // Log para debugging (solo en desarrollo)
            if (window.location.hostname === "localhost") {
                console.log(`Navigated to route: ${sRouteName}`);
            }
        }
    });
});
```

## 7. Performance en Navegación

### ✅ OBLIGATORIO: Lazy Loading de Vistas
```javascript
// Configuración de targets con lazy loading
{
  "targets": {
    "detail": {
      "id": "detail",
      "name": "Detail",
      "level": 2,
      "transition": "slide"
    }
  }
}

// Controller con lazy loading
onNavigateToDetail: function(sObjectId) {
    // ✅ CORRECTO: Verificar si la vista ya está cargada
    if (!this._detailView) {
        this.getRouter().navTo("detail", {
            objectId: sObjectId
        });
    } else {
        // Reutilizar vista existente
        this._updateDetailView(sObjectId);
    }
}
```

---

## Herramientas de Validación

### Verificación de Routing
```bash
# Verificar configuración de routing
grep -r "routing" webapp/manifest.json

# Verificar uso correcto de router
grep -r "getRouter\|navTo" webapp/controller/ --include="*.js"
```

### Testing de Navegación
```javascript
// Test de navegación con OPA5
opaTest("Should navigate to detail page", function(Given, When, Then) {
    Given.iStartMyApp();
    
    When.onTheMainPage.iPressOnTheFirstItem();
    
    Then.onTheDetailPage.iShouldSeeTheDetailPage()
        .and.iShouldSeeTheCorrectObjectData();
});
```

---

**CRÍTICO**: El incumplimiento de estas reglas puede resultar en:
- Navegación inconsistente entre vistas
- URLs no bookmarkeables
- Problemas de deep linking
- Fallos en integración con Fiori Launchpad
- Problemas de cross-navigation en BTP