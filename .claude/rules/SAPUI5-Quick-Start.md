# SAPUI5 Quick Start Guide - Guía de Inicio Rápido

> **OBJETIVO**: Navegar eficientemente las reglas de desarrollo SAPUI5 FreeStyle
> **AUDIENCIA**: Desarrolladores nuevos y experimentados
> **TIEMPO TOTAL**: 15-30 minutos para empezar

## 🚀 Empezar Aquí

### ⚡ Top 10 Reglas Críticas (Memoriza Estas)

1. ❌ **NUNCA** usar `var`, solo `const` y `let`
2. ❌ **NUNCA** modificar controles UI5 con CSS personalizado
3. ❌ **NUNCA** usar variables globales para acceder a UI5
4. ❌ **NUNCA** usar VBox como contenedor principal de Page
5. ❌ **NUNCA** poner textos directamente en XML (usar i18n)
6. ✅ **SIEMPRE** usar `sap.ui.define` para módulos
7. ✅ **SIEMPRE** usar data binding en lugar de manipulación directa
8. ✅ **SIEMPRE** formatters en `webapp/model/formatter.js`
9. ✅ **SIEMPRE** ARIA labels para accesibilidad
10. ✅ **SIEMPRE** ejecutar UI5 Linter antes de commit

---

## 📚 Flujo de Lectura Recomendado

### 🎓 Nivel 1: Básico (Para Nuevos en SAPUI5)

**Tiempo Total: ~70 minutos**

1. **[SAPUI5-Core-Standards.md](./SAPUI5-Core-Standards.md)** - 30 min ⭐⭐⭐⭐⭐
   - Fundamentos obligatorios
   - MVC, data binding, módulos
   - **Leer primero siempre**

2. **[SAPUI5-Design-Controls.md](./SAPUI5-Design-Controls.md)** - 25 min ⭐⭐⭐⭐⭐
   - Controles estándar
   - Layouts (VBox/HBox)
   - Smart Controls
   - **Leer segundo**

3. **[SAPUI5-Formatters-DataBinding.md](./SAPUI5-Formatters-DataBinding.md)** - 15 min ⭐⭐⭐⭐
   - Formatters y tipos de datos
   - Contexto `this`
   - **Leer tercero**

### 🎓 Nivel 2: Intermedio (Para Proyectos Existentes)

**Tiempo Total: ~55 minutos**

4. **[SAPUI5-Security-Performance.md](./SAPUI5-Security-Performance.md)** - 20 min ⭐⭐⭐⭐⭐
   - CSP, XSS, validación
   - Performance y optimización
   - **Antes de producción**

5. **[SAPUI5-Accessibility-i18n.md](./SAPUI5-Accessibility-i18n.md)** - 20 min ⭐⭐⭐⭐⭐
   - ARIA, i18n, RTL
   - Compliance obligatorio
   - **Antes de producción**

6. **[SAPUI5-Development-Tools.md](./SAPUI5-Development-Tools.md)** - 15 min ⭐⭐⭐
   - Linter, testing, JSDoc
   - Setup de herramientas
   - **Setup inicial**

### 🎓 Nivel 3: Avanzado (Para Deployment)

**Tiempo Total: ~40 minutos**

7. **[SAPUI5-Routing-Navigation.md](./SAPUI5-Routing-Navigation.md)** - 15 min ⭐⭐⭐⭐
   - Routing, deep linking
   - Cross-navigation BTP
   - **Apps multi-página**

8. **[SAPUI5-BTP-Deployment.md](./SAPUI5-BTP-Deployment.md)** - 25 min ⭐⭐⭐⭐
   - MTA, Cloud Foundry
   - Servicios BTP
   - **Deploy a BTP**

---

## 🎯 Tabla de Decisión Rápida

| Tu Necesidad | Archivo a Leer | Sección | Tiempo |
|--------------|----------------|---------|--------|
| **Empezar proyecto nuevo** | Core Standards | Todo | 30 min |
| **Problema con formatters** | Formatters & Data Binding | #2, #3 | 10 min |
| **Tabla se corta a la mitad** | Design & Controls | #8 | 5 min |
| **SmartFilterBar + SmartTable** | Design & Controls | #9 | 10 min |
| **Deploy a BTP** | BTP Deployment | Todo | 25 min |
| **Textos hardcodeados** | Accessibility & i18n | #1 | 5 min |
| **Problemas de accesibilidad** | Accessibility & i18n | Todo | 20 min |
| **Configurar routing** | Routing & Navigation | #1, #2 | 10 min |
| **Setup testing** | Development Tools | #2 | 10 min |
| **Performance lento** | Security & Performance | #4 | 10 min |
| **Errores de seguridad** | Security & Performance | #1, #2 | 15 min |
| **CSS no funciona** | Design & Controls | #1 | 5 min |

---

## 🔍 Búsqueda por Problema Común

### Problemas de Layout

**"Mi tabla se corta a la mitad"**
→ [Design & Controls - Sección 8: VBox/HBox](./SAPUI5-Design-Controls.md#8-layouts-y-contenedores-vbox-hbox-flexbox)
- **Causa**: VBox como contenedor principal
- **Solución**: Quitar VBox, poner contenido directo en Page

**"SmartTable no ocupa todo el alto"**
→ [Design & Controls - Sección 9: SmartTable Layout](./SAPUI5-Design-Controls.md#layout-correcto-crítico)
- **Causa**: VBox contenedor
- **Solución**: Sin VBox o usar DynamicPage

**"Scroll extra innecesario"**
→ [Design & Controls - Sección 8: Problemas Comunes](./SAPUI5-Design-Controls.md#problemas-comunes-y-soluciones)
- **Solución**: Usar FlexBox con fitContainer

### Problemas de Formatters

**"Formatters no funcionan"**
→ [Formatters & Data Binding - Sección 2: Contexto this](./SAPUI5-Formatters-DataBinding.md#2-contexto-this-en-formatters---crítico)
- **Causa**: Contexto `this` incorrecto
- **Solución**: Usar `core:require` o dot notation

**"Formatter devuelve undefined"**
→ [Formatters & Data Binding - Sección 8: Validación](./SAPUI5-Formatters-DataBinding.md#8-validación-y-manejo-de-errores)
- **Causa**: Parámetros null/undefined
- **Solución**: Validar parámetros siempre

### Problemas de Smart Controls

**"SmartTable no carga datos"**
→ [Design & Controls - Sección 9: SmartTable Problemas](./SAPUI5-Design-Controls.md#problemas-comunes-y-soluciones-1)
- **Causa**: Falta `enableAutoBinding="true"`
- **Solución**: Agregar enableAutoBinding

**"SmartFilterBar filtros no se aplican"**
→ [Design & Controls - Sección 9: SmartFilterBar Problemas](./SAPUI5-Design-Controls.md#problemas-comunes-y-soluciones)
- **Causa**: Falta vinculación con SmartTable
- **Solución**: Configurar smartFilterId

**"Persistencia de filtros no funciona"**
→ [Design & Controls - Sección 9: SmartFilterBar](./SAPUI5-Design-Controls.md#configuración-básica-obligatoria)
- **Causa**: Falta persistencyKey
- **Solución**: Agregar persistencyKey único

### Problemas de Datos

**"Textos hardcodeados"**
→ [Accessibility & i18n - Sección 1](./SAPUI5-Accessibility-i18n.md#1-regla-fundamental-todos-los-textos-en-i18n)
- **Solución**: Mover todos los textos a archivos i18n

**"Data binding no funciona"**
→ [Core Standards - Sección 5](./SAPUI5-Core-Standards.md#5-data-binding-obligatorio)
- **Causa**: Manipulación directa de UI
- **Solución**: Usar data binding siempre

### Problemas de Deployment

**"Error al deployar en BTP"**
→ [BTP Deployment - Sección 1: MTA](./SAPUI5-BTP-Deployment.md#1-configuración-mta-multi-target-application)
- **Solución**: Verificar estructura MTA

**"Routing no funciona en BTP"**
→ [Routing & Navigation - Sección 5: Cross-Navigation](./SAPUI5-Routing-Navigation.md#5-navegación-cross-app-btp)
- **Solución**: Configurar cross-navigation

---

## 📊 Mapa de Dependencias

```
┌─────────────────────────┐
│   Core Standards        │ ← EMPEZAR AQUÍ (BASE)
│   (Fundamentos)         │
└───────────┬─────────────┘
            │
            ├──────────────────────────────────┐
            │                                  │
            ▼                                  ▼
┌─────────────────────────┐      ┌─────────────────────────┐
│ Formatters & Data       │      │ Design & Controls       │
│ Binding                 │      │ (UI, Layouts, Smart)    │
└─────────────────────────┘      └───────────┬─────────────┘
                                             │
                                             ▼
                                 ┌─────────────────────────┐
                                 │ Accessibility & i18n    │
                                 │ (Compliance)            │
                                 └─────────────────────────┘
            │
            ├──────────────────────────────────┐
            │                                  │
            ▼                                  ▼
┌─────────────────────────┐      ┌─────────────────────────┐
│ Security & Performance  │      │ Routing & Navigation    │
│ (Producción)            │      │ (Multi-página)          │
└─────────────────────────┘      └───────────┬─────────────┘
                                             │
                                             ▼
                                 ┌─────────────────────────┐
                                 │ BTP Deployment          │
                                 │ (Cloud Foundry)         │
                                 └───────────┬─────────────┘
                                             │
                                             ▼
                                 ┌─────────────────────────┐
                                 │ Development Tools       │
                                 │ (Testing, Linter)       │
                                 └─────────────────────────┘
```

---

## 🎯 Checklist por Fase de Proyecto

### ✅ Fase 1: Setup Inicial (Día 1)

- [ ] Leer Core Standards completo
- [ ] Configurar UI5 Linter
- [ ] Configurar estructura de proyecto estándar
- [ ] Configurar i18n básico
- [ ] Leer Design & Controls (secciones 1-7)

### ✅ Fase 2: Desarrollo (Semana 1-2)

- [ ] Leer Formatters & Data Binding
- [ ] Implementar formatters en ubicación correcta
- [ ] Leer Design & Controls (secciones 8-9)
- [ ] Implementar layouts correctos
- [ ] Configurar Smart Controls si aplica

### ✅ Fase 3: Pre-Producción (Semana 3)

- [ ] Leer Security & Performance completo
- [ ] Ejecutar audit de seguridad
- [ ] Leer Accessibility & i18n completo
- [ ] Validar compliance de accesibilidad
- [ ] Configurar testing (Development Tools)

### ✅ Fase 4: Deployment (Semana 4)

- [ ] Leer Routing & Navigation
- [ ] Configurar routing completo
- [ ] Leer BTP Deployment
- [ ] Configurar MTA
- [ ] Deploy a BTP

---

## 🔗 Links Rápidos

### Documentación Principal
- 📋 [Índice Completo de Reglas](./SAPUI5-Rules-Index.md)
- 🔧 [Core Standards](./SAPUI5-Core-Standards.md)
- 🎨 [Design & Controls](./SAPUI5-Design-Controls.md)
- 🎯 [Formatters & Data Binding](./SAPUI5-Formatters-DataBinding.md)

### Por Tema
- 🔒 [Security & Performance](./SAPUI5-Security-Performance.md)
- ♿ [Accessibility & i18n](./SAPUI5-Accessibility-i18n.md)
- 🧭 [Routing & Navigation](./SAPUI5-Routing-Navigation.md)
- ☁️ [BTP Deployment](./SAPUI5-BTP-Deployment.md)
- 🛠️ [Development Tools](./SAPUI5-Development-Tools.md)

---

## 💡 Consejos de Uso

### Para Nuevos Desarrolladores
1. **No intentes leer todo de una vez**
   - Empieza con Core Standards
   - Luego Design & Controls
   - El resto según necesites

2. **Usa la búsqueda**
   - Ctrl+F en cada archivo
   - Busca por problema específico
   - Usa la tabla de decisión rápida

3. **Practica mientras lees**
   - No solo leas, implementa
   - Prueba los ejemplos
   - Ejecuta el linter

### Para Desarrolladores Experimentados
1. **Usa como referencia**
   - No necesitas leer todo
   - Busca temas específicos
   - Consulta cuando tengas dudas

2. **Enfócate en gaps**
   - Identifica áreas débiles
   - Lee esas secciones específicas
   - Actualiza tu código

3. **Mantente actualizado**
   - Revisa cambios periódicamente
   - Verifica nuevas secciones
   - Adapta tu código

---

## 📈 Niveles de Prioridad

### ⭐⭐⭐⭐⭐ CRÍTICO (Leer Siempre)
- Core Standards
- Design & Controls
- Security & Performance
- Accessibility & i18n

### ⭐⭐⭐⭐ IMPORTANTE (Leer Según