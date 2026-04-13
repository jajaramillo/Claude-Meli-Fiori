# SAPUI5 Development Tools - Herramientas de Desarrollo y Calidad

> **APLICABILIDAD**: Todas las aplicaciones SAPUI5 FreeStyle
> **PRIORIDAD**: ⭐ IMPORTANTE - Cumplimiento recomendado al 100%
> **FUENTE**: SAP Development Best Practices y UI5 Tooling

## 1. UI5 Linter (OBLIGATORIO)

### ✅ OBLIGATORIO: Uso del Linter UI5
```bash
# Instalar UI5 Linter
npm install -g @ui5/linter

# Ejecutar antes de cada commit
npx @ui5/linter --fix

# Verificar sin aplicar cambios
npx @ui5/linter
```

### Configuración del Linter
```json
// .ui5lintrc.json
{
  "rules": {
    "no-deprecated-api": "error",
    "no-direct-dom-access": "error",
    "no-global-ui5": "error",
    "async-component-flags": "error"
  },
  "coverage": {
    "excludePatterns": [
      "test/**",
      "localService/**"
    ]
  }
}
```

### MCP Tools para Linting
```javascript
// ✅ CORRECTO: Usar MCP tools para validación
// Usar get_api_reference para verificar APIs
// Usar run_ui5_linter para validación automática
// Usar get_project_info para verificar configuración
```

## 2. Testing (QUnit y OPA5)

### ✅ OBLIGATORIO: Unit Tests con QUnit
```javascript
// test/unit/controller/Main.controller.test.js
sap.ui.define([
    "com/myapp/controller/Main.controller",
    "sap/ui/core/mvc/Controller"
], function(MainController, Controller) {
    "use strict";

    QUnit.module("Main Controller", {
        beforeEach: function() {
            this.oController = new MainController();
        },
        afterEach: function() {
            this.oController.destroy();
        }
    });

    QUnit.test("Should initialize correctly", function(assert) {
        // Arrange
        const oView = {
            getModel: sinon.stub().returns({
                setProperty: sinon.spy()
            })
        };
        sinon.stub(this.oController, "getView").returns(oView);

        // Act
        this.oController.onInit();

        // Assert
        assert.ok(true, "Controller initialized without errors");
    });
});
```

### ✅ OBLIGATORIO: Integration Tests con OPA5
```javascript
// test/integration/NavigationJourney.js
sap.ui.define([
    "sap/ui/test/opaQunit",
    "./pages/Main",
    "./pages/Detail"
], function(opaTest) {
    "use strict";

    QUnit.module("Navigation Journey");

    opaTest("Should navigate to detail page", function(Given, When, Then) {
        // Arrange
        Given.iStartMyApp();

        // Act
        When.onTheMainPage.iPressOnTheFirstItem();

        // Assert
        Then.onTheDetailPage.iShouldSeeTheDetailPage();
    });
});
```

### Configuración de Testing
```json
// karma.conf.js
module.exports = function(config) {
    config.set({
        frameworks: ["ui5"],
        ui5: {
            type: "application",
            configPath: "ui5.yaml",
            paths: {
                webapp: "webapp"
            }
        },
        browsers: ["Chrome"],
        singleRun: true,
        coverageReporter: {
            type: "html",
            dir: "coverage/"
        }
    });
};
```

## 3. Documentación JSDoc

### ✅ OBLIGATORIO: JSDoc para Funciones Públicas
```javascript
/**
 * Controller principal de la aplicación
 * @namespace com.myapp.controller
 */
sap.ui.define([
    "sap/ui/core/mvc/Controller"
], function(Controller) {
    "use strict";

    return Controller.extend("com.myapp.controller.Main", {

        /**
         * Inicializa el controller
         * @public
         * @override
         */
        onInit: function() {
            this._initializeModels();
        },

        /**
         * Maneja el evento de presionar un item
         * @param {sap.ui.base.Event} oEvent - Evento de presión
         * @public
         */
        onItemPress: function(oEvent) {
            const oContext = oEvent.getSource().getBindingContext();
            this._navigateToDetail(oContext);
        },

        /**
         * Navega a la página de detalle
         * @param {sap.ui.model.Context} oContext - Contexto del item
         * @private
         */
        _navigateToDetail: function(oContext) {
            const sObjectId = oContext.getProperty("ID");
            this.getRouter().navTo("detail", {
                objectId: sObjectId
            });
        }
    });
});
```

### Configuración JSDoc
```json
// jsdoc.conf.json
{
    "source": {
        "include": ["./webapp/"],
        "exclude": ["./webapp/test/"]
    },
    "opts": {
        "destination": "./docs/"
    },
    "plugins": ["plugins/markdown"]
}
```

## 4. Estructura de Proyecto Estándar

### ✅ OBLIGATORIO: Organización de Carpetas
```
webapp/
├── controller/              # Controllers MVC
│   ├── BaseController.js   # Controller base común
│   ├── Main.controller.js  # Controller principal
│   └── Detail.controller.js
├── view/                   # Views XML
│   ├── Main.view.xml
│   └── Detail.view.xml
├── model/                  # Modelos y formatters
│   ├── models.js          # Configuración de modelos
│   └── formatter.js       # Funciones de formato
├── css/                   # Estilos CSS personalizados
│   └── style.css
├── i18n/                  # Archivos de traducción
│   ├── i18n.properties
│   └── i18n_es.properties
├── localService/          # Mock data para desarrollo
│   ├── metadata.xml
│   └── mockdata/
├── test/                  # Tests unitarios e integración
│   ├── unit/
│   └── integration/
├── Component.js           # Componente principal
├── manifest.json          # Descriptor de aplicación
└── index.html            # Página de entrada
```

### Naming Conventions para Archivos
```javascript
// ✅ CORRECTO: Nombres descriptivos
Main.controller.js         // Controllers
Main.view.xml             // Views
CustomerService.js        // Servicios
ValidationHelper.js       // Helpers/Utilities
formatter.js              // Formatters

// ❌ INCORRECTO: Nombres genéricos
controller1.js
view.xml
service.js
```

## 5. Configuración de Herramientas

### Package.json Estándar
```json
{
    "name": "my-ui5-app",
    "version": "1.0.0",
    "scripts": {
        "start": "ui5 serve",
        "build": "ui5 build",
        "lint": "@ui5/linter",
        "test": "karma start",
        "test:watch": "karma start --auto-watch --no-single-run"
    },
    "devDependencies": {
        "@ui5/cli": "^3.0.0",
        "@ui5/linter": "^0.1.0",
        "karma": "^6.0.0",
        "karma-ui5": "^3.0.0"
    }
}
```

### UI5.yaml Configuración
```yaml
# ui5.yaml
specVersion: "3.0"
metadata:
  name: my-ui5-app
type: application
framework:
  name: SAPUI5
  version: "1.120.0"
  libraries:
    - name: sap.m
    - name: sap.ui.core
    - name: sap.f
server:
  customMiddleware:
    - name: ui5-middleware-livereload
      afterMiddleware: compression
```

## 6. Git Hooks y CI/CD

### ✅ RECOMENDADO: Pre-commit Hooks
```json
// package.json
{
    "husky": {
        "hooks": {
            "pre-commit": "lint-staged"
        }
    },
    "lint-staged": {
        "*.{js,ts}": [
            "@ui5/linter --fix",
            "git add"
        ]
    }
}
```

### Pipeline CI/CD Básico
```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build
```

## 7. Control de Versiones - Rollback de Cambios

### ✅ OBLIGATORIO: Snapshot de Estado al Inicio de Sesión

**ANTES de realizar cualquier modificación a un archivo en una sesión**, siempre guardar una copia del estado actual del archivo en:
```
~/.claude/projects/<project-id>/memory/snapshots/<filename>.session-YYYYMMDD
```

**Por qué es necesario**: El usuario puede tener cambios sin commitear antes de iniciar la sesión. Si se revierte usando `git restore`, esos cambios pre-sesión se perderían. El snapshot garantiza poder volver exactamente al estado en que estaba el archivo cuando comenzó la sesión, independientemente de git.

```bash
# Guardar snapshot al inicio de sesión (ANTES de modificar)
# Primero eliminar snapshots anteriores del mismo archivo, luego guardar el nuevo
rm -f ~/.claude/projects/<id>/memory/snapshots/MyFile.js.session-*
cp webapp/controller/MyFile.js \
   ~/.claude/projects/<id>/memory/snapshots/MyFile.js.session-$(date +%Y%m%d)
```

> **POLÍTICA DE RETENCIÓN — Un snapshot por archivo**:
> - Se mantiene **un único snapshot por archivo**, siempre el de la sesión más reciente.
> - Al iniciar nueva sesión: eliminar el snapshot anterior del mismo archivo y crear uno nuevo con el estado actual.
> - Si ya existe snapshot del **mismo día** para ese archivo: NO reemplazarlo (el primero del día es el estado original de esa sesión).
> - Para recuperación histórica más allá de la sesión anterior: usar `git log`.

---

### ✅ OBLIGATORIO: Validación Antes de Revertir Cambios

Cuando se solicite **revertir**, **deshacer** o **hacer rollback** de cambios, siempre ofrecer las tres opciones y confirmar con el usuario:

**Opción A - Volver al inicio de esta sesión (snapshot)**
```bash
# Restaurar desde el snapshot guardado al inicio de la sesión
cp ~/.claude/projects/<id>/memory/snapshots/MyFile.js.session-YYYYMMDD \
   webapp/controller/MyFile.js
```
> Usa esta opción cuando el usuario dice "revertir lo que hiciste hoy" o "deshacer los cambios de esta sesión".

**Opción B - Revertir al último commit (estado en git)**
```bash
# Descartar cambios de un archivo específico (vuelve al HEAD de git)
git restore <archivo>

# Descartar todos los cambios sin commitear
git restore .
```
> ⚠️ Esta opción elimina TAMBIÉN los cambios pre-sesión que no tenían commit.

**Opción C - Revertir un commit commiteado**
```bash
# Deshacer el último commit pero conservar los cambios en staging
git reset --soft HEAD~1

# Revertir un commit específico (crea un nuevo commit inverso)
git revert <commit-hash>
```

### ⚠️ REGLA CRÍTICA: Siempre Confirmar el Alcance

Antes de ejecutar cualquier reversión, **siempre validar con el usuario**:

```
1. ¿A qué punto quieres regresar?
   - [ ] Al inicio de esta sesión (snapshot — solo deshace lo que Claude hizo hoy)
   - [ ] Al último commit en git (descarta TODO lo que no está commiteado)
   - [ ] A un commit específico (indicar cuál)

2. ¿Quieres conservar los cambios revertidos como archivos modificados
   o eliminarlos completamente?
```

### ❌ PROHIBIDO: Acciones Destructivas Sin Confirmación

```bash
# ❌ NUNCA ejecutar sin confirmación explícita del usuario
git reset --hard HEAD~1    # Destruye cambios commiteados sin recuperación
git clean -fd              # Elimina archivos no trackeados permanentemente
git push --force           # Sobreescribe historial remoto
git restore <archivo>      # Sin confirmar si hay snapshot de sesión
```

### Tabla de Decisión Rápida

| El usuario dice... | Preguntar si quiere... |
|--------------------|------------------------|
| "revertir los últimos cambios" | ¿Solo lo de esta sesión (snapshot) o también cambios previos sin commit? |
| "deshacer lo que hiciste" | Usar snapshot de sesión — confirmar primero |
| "volver al estado anterior" | ¿Anterior a esta sesión, o al último commit? |
| "rollback" | ¿Qué alcance? ¿Snapshot de sesión, git restore, o git reset? |
| "revertir cambios" | ¿Sesión actual (snapshot) o último commit (git)? |

---

## 8. Debugging y Troubleshooting

### ✅ CORRECTO: Configuración de Debug
```javascript
// ✅ CORRECTO: Usar logging estructurado
sap.ui.define([
    "sap/base/Log"
], function(Log) {
    "use strict";
    
    // Configurar nivel de log en desarrollo
    Log.setLevel(Log.Level.DEBUG);
    
    // Usar diferentes niveles apropiadamente
    Log.debug("Detailed debug information", this);
    Log.info("General information", this);
    Log.warning("Warning message", this);
    Log.error("Error occurred", oError, this);
});
```

### Herramientas de Debug
```javascript
// ✅ CORRECTO: Usar herramientas UI5
// UI5 Inspector (extensión de browser)
// Support Assistant
sap.ui.require(["sap/ui/support/Bootstrap"], function(Bootstrap) {
    Bootstrap.initSupportRules(["true", "silent"], {
        onReady: function() {
            // Support rules initialized
        }
    });
});
```

---

## Checklist de Calidad

### ✅ Antes de cada Commit
- [ ] UI5 Linter ejecutado sin errores
- [ ] Tests unitarios pasando
- [ ] JSDoc actualizado para funciones públicas
- [ ] No hay console.log en código de producción
- [ ] Archivos organizados según estructura estándar

### ✅ Antes de Revertir Cambios
- [ ] Confirmar con el usuario si es rollback de sesión o de commit
- [ ] Confirmar si se deben conservar o descartar los cambios revertidos
- [ ] Nunca ejecutar `reset --hard` o `clean -fd` sin confirmación explícita

### ✅ Antes de Release
- [ ] Tests de integración pasando
- [ ] Documentación actualizada
- [ ] Performance verificado
- [ ] Accesibilidad validada
- [ ] Build de producción exitoso

---

**IMPORTANTE**: Estas herramientas garantizan:
- Calidad de código consistente
- Detección temprana de errores
- Mantenibilidad a largo plazo
- Compliance con estándares SAP