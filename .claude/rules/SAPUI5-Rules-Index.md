# SAPUI5 FreeStyle - Índice de Reglas de Desarrollo

> **APLICABILIDAD**: Todas las aplicaciones SAPUI5 FreeStyle
> **FUENTE**: Documentación oficial SAP UI5 MCP Servers y mejores prácticas
> **VERSIÓN**: 1.1 - Octubre 2025
> **ENFOQUE**: SAP BTP (Cloud Foundry & Kyma Runtime)

## 📋 Resumen Ejecutivo

Este conjunto de reglas garantiza el desarrollo de aplicaciones SAPUI5 FreeStyle de alta calidad, siguiendo los estándares oficiales de SAP y las mejores prácticas de la industria. Todas las reglas están basadas en documentación oficial obtenida de los MCP servers de SAP.

**Enfoque Principal**: Aplicaciones desplegadas en **SAP Business Technology Platform (BTP)** usando Cloud Foundry o Kyma Runtime.

---

## 🚀 Inicio Rápido

**¿Primera vez con estas reglas?** → [📖 Quick Start Guide](./SAPUI5-Quick-Start.md)

**¿Buscas algo específico?** → Usa Ctrl+F en esta página

**¿Tienes un problema concreto?** → [🔍 Búsqueda por Problema](./SAPUI5-Quick-Start.md#búsqueda-por-problema-común)

**¿No sabes por dónde empezar?** → [📚 Flujo de Lectura Recomendado](./SAPUI5-Quick-Start.md#flujo-de-lectura-recomendado)

## 📌 Compatibilidad y Requisitos Generales

### Versiones UI5
- **Mínimo Recomendado**: UI5 1.96+ (LTS)
- **Producción BTP**: Usar siempre la última versión LTS disponible en SAP BTP
- **Verificar Versión**: [SAP UI5 Version Overview](https://ui5.sap.com/versionoverview.html)

### Plataforma de Deployment
- ✅ **SAP BTP Cloud Foundry** (Recomendado)
- ✅ **SAP BTP Kyma Runtime**
- ✅ **HTML5 Application Repository**
- ✅ **XSUAA Authentication**
- ⚠️ Neo Environment (Deprecated - migrar a Cloud Foundry)

### Características Modernas Requeridas
- ✅ Async component loading
- ✅ ES6+ JavaScript support
- ✅ TypeScript support (opcional pero recomendado)
- ✅ OData V4 support
- ✅ MTA (Multi-Target Application) deployment

---

## 📚 Archivos de Reglas

### 1. 🔧 [SAPUI5-Core-Standards.md](./SAPUI5-Core-Standards.md)
**PRIORIDAD: ⭐ CRÍTICO**
- Gestión de dependencias y módulos
- Declaración de variables (const/let vs var)
- Convenciones de nomenclatura (Hungarian notation)
- Arquitectura MVC obligatoria
- Data binding y manejo de eventos
- Integración con CAP

**Reglas clave:**
- ❌ Prohibición absoluta de variables globales UI5
- ✅ Uso obligatorio de `sap.ui.define` y ES6 imports
- ✅ `const/let` en lugar de `var`
- ✅ Separación estricta MVC

### 2. 🎯 [SAPUI5-Formatters-DataBinding.md](./SAPUI5-Formatters-DataBinding.md) **NUEVO**
**PRIORIDAD: ⭐ CRÍTICO**
- Formatters - Reglas fundamentales
- Contexto `this` en formatters (CRÍTICO)
- Function expressions vs arrow functions
- Formatters vs Data Types - Decisión
- Patrones comunes de formateo
- Validación y testing obligatorio

**Reglas clave:**
- ✅ Ubicación obligatoria: `webapp/model/formatter.js`
- ✅ Uso de `core:require` o dot notation
- ✅ Function expressions preferidas sobre arrow functions
- ✅ Validación de parámetros null/undefined
- ❌ **PROHIBIDO**: Formatters inline, lógica de negocio, side effects

### 3. 🎨 [SAPUI5-Design-Controls.md](./SAPUI5-Design-Controls.md)
**PRIORIDAD: ⭐ CRÍTICO**
- Uso estricto de controles estándar
- Prohibición de CSS personalizado en controles
- Fiori Design Guidelines compliance
- Theming y responsive design
- Patrones de interacción
- **Layouts y Contenedores** (VBox, HBox, FlexBox)
- **Smart Controls en FreeStyle** (SmartFilterBar, SmartTable, SmartField)

**Reglas clave:**
- ❌ **PROHIBIDO**: Modificar controles UI5 con CSS personalizado
- ✅ Solo controles estándar SAP
- ✅ Solo temas oficiales SAP
- ✅ Iconografía estándar únicamente
- ✅ VBox/HBox: Cuándo usar y cuándo NO usar
- ✅ Smart Controls: Configuración correcta en FreeStyle (NO Fiori Elements)

### 4. 🔒 [SAPUI5-Security-Performance.md](./SAPUI5-Security-Performance.md)
**PRIORIDAD: ⭐ CRÍTICO**
- Content Security Policy (CSP)
- Prevención XSS
- Optimización de rendimiento
- Manejo de errores estructurado
- Gestión de memoria

**Reglas clave:**
- ✅ CSP compliance obligatorio
- ✅ Validación de entrada de datos
- ✅ Lazy loading de módulos
- ✅ Batch requests para OData

### 5. ♿ [SAPUI5-Accessibility-i18n.md](./SAPUI5-Accessibility-i18n.md)
**PRIORIDAD: ⭐ CRÍTICO**
- ARIA labels y screenreader support
- Navegación por teclado
- Soporte RTL (Right-to-Left)
- Internacionalización (i18n)
- Temas de alto contraste

**Reglas clave:**
- ✅ ARIA labels obligatorios
- ✅ Soporte completo de teclado
- ✅ Todos los textos en archivos i18n
- ✅ Soporte RTL y alto contraste

### 6. 🧭 [SAPUI5-Routing-Navigation.md](./SAPUI5-Routing-Navigation.md)
**PRIORIDAD: ⭐ CRÍTICO**
- Configuración de routing en manifest.json
- Patrones de navegación estándar
- Deep linking y bookmarking
- Cross-navigation en BTP
- Manejo de parámetros de ruta

**Reglas clave:**
- ✅ BaseController obligatorio para navegación
- ✅ Configuración de routing async
- ✅ Cross-navigation para BTP
- ✅ URLs amigables y bookmarkeables

### 7. ☁️ [SAPUI5-BTP-Deployment.md](./SAPUI5-BTP-Deployment.md)
**PRIORIDAD: ⭐ CRÍTICO**
- Configuración MTA para BTP
- Integración con servicios BTP
- Destinations y connectivity
- CI/CD pipelines para BTP
- Optimización para Cloud Foundry

**Reglas clave:**
- ✅ Estructura MTA obligatoria
- ✅ XSUAA authentication
- ✅ HTML5 Application Repository
- ✅ Configuración de destinations

### 8. 🛠️ [SAPUI5-Development-Tools.md](./SAPUI5-Development-Tools.md)
**PRIORIDAD: ⭐ IMPORTANTE**
- UI5 Linter obligatorio
- Testing (QUnit y OPA5)
- Documentación JSDoc
- Estructura de proyecto estándar
- CI/CD y debugging

**Reglas clave:**
- ✅ Linter UI5 antes de cada commit
- ✅ Tests unitarios e integración
- ✅ JSDoc para funciones públicas
- ✅ Estructura de carpetas estándar

---

## 🎯 Prioridades de Implementación

### ⭐ CRÍTICO (Cumplimiento Obligatorio 100%)
1. **Core Standards** - Base fundamental
2. **Formatters & Data Binding** - Formateo y vinculación de datos
3. **Design & Controls** - Controles estándar únicamente
4. **Security & Performance** - Seguridad y rendimiento
5. **Accessibility & i18n** - Accesibilidad e internacionalización
6. **Routing & Navigation** - Navegación y enrutamiento
7. **BTP Deployment** - Despliegue en SAP BTP

### ⭐ IMPORTANTE (Cumplimiento Recomendado 100%)
8. **Development Tools** - Herramientas y calidad

---

## 🚀 Guía de Implementación Rápida

### Para Nuevos Proyectos
1. Leer **Core Standards** completo
2. Configurar linter UI5 según **Development Tools**
3. Implementar estructura de proyecto estándar
4. Aplicar reglas de **Design & Controls**
5. Configurar i18n según **Accessibility & i18n**

### Para Proyectos Existentes
1. Ejecutar audit con linter UI5
2. Revisar uso de controles según **Design & Controls**
3. Validar seguridad según **Security & Performance**
4. Verificar accesibilidad según **Accessibility & i18n**
5. Refactorizar según **Core Standards**

---

## 🔍 Herramientas de Validación

### MCP Tools Disponibles
- `get_api_reference` - Consultar APIs UI5
- `run_ui5_linter` - Validación automática
- `get_project_info` - Verificar configuración

### Comandos Esenciales
```bash
# Validación completa
npx @ui5/linter --fix

# Verificar CSS personalizado
grep -r "\.sap[A-Z].*{" src/ --include="*.css"

# Verificar textos hardcodeados
grep -r "text=\"[^{]" webapp/view/ --include="*.xml"
```

---

## 📊 Checklist de Compliance

### ✅ Antes de cada Commit
- [ ] UI5 Linter ejecutado sin errores
- [ ] No hay variables globales UI5
- [ ] No hay CSS personalizado en controles
- [ ] Todos los textos en archivos i18n
- [ ] ARIA labels en controles interactivos

### ✅ Antes de cada Release
- [ ] Tests unitarios e integración pasando
- [ ] Validación de accesibilidad completa
- [ ] Performance verificado
- [ ] Documentación actualizada
- [ ] Build de producción exitoso

---

## 🆘 Soporte y Recursos

### Documentación Oficial
- [SAP UI5 Documentation](https://ui5.sap.com/)
- [Fiori Design Guidelines](https://experience.sap.com/fiori-design/)
- [SAP Accessibility Guidelines](https://experience.sap.com/accessibility/)

### Herramientas
- [UI5 Linter](https://www.npmjs.com/package/@ui5/linter)
- [UI5 CLI](https://www.npmjs.com/package/@ui5/cli)
- [UI5 Inspector](https://chrome.google.com/webstore/detail/ui5-inspector/bebecogbafbighhaildooiibipcnbnpd)

---

**IMPORTANTE**: Estas reglas son el resultado de un análisis exhaustivo de la documentación oficial de SAP obtenida a través de MCP servers especializados. Su cumplimiento garantiza aplicaciones SAPUI5 de calidad empresarial, mantenibles y compatibles con los estándares SAP.

**ÚLTIMA ACTUALIZACIÓN**: Octubre 2025 - Basado en UI5 1.120+ y mejores prácticas actuales.