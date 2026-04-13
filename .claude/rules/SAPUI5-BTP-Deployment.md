# SAPUI5 BTP Deployment - Reglas de Despliegue en SAP BTP

> **APLICABILIDAD**: Todas las aplicaciones SAPUI5 FreeStyle desplegadas en SAP BTP
> **PRIORIDAD**: ⭐ CRÍTICO - Cumplimiento obligatorio al 100%
> **FUENTE**: SAP BTP Documentation y Cloud Foundry Best Practices

## 1. Configuración MTA (Multi-Target Application)

### ✅ OBLIGATORIO: Estructura MTA Básica
```yaml
# mta.yaml
_schema-version: "3.3"
ID: my-ui5-app
description: SAPUI5 FreeStyle Application
version: 1.0.0
author: Your Name

modules:
  - name: my-ui5-app-destination-content
    type: com.sap.application.content
    requires:
      - name: my-ui5-app-destination-service
        parameters:
          content-target: true
      - name: my-ui5-app-repo-host
        parameters:
          service-key:
            name: my-ui5-app-repo-host-key
      - name: my-ui5-app-uaa
        parameters:
          service-key:
            name: my-ui5-app-uaa-key
    parameters:
      content:
        instance:
          destinations:
            - Name: my-ui5-app_html_repo_host
              ServiceInstanceName: my-ui5-app-html5-srv
              ServiceKeyName: my-ui5-app-repo-host-key
              sap.cloud.service: my-ui5-app
            - Authentication: OAuth2UserTokenExchange
              Name: my-ui5-app_uaa
              ServiceInstanceName: my-ui5-app-xsuaa-service
              ServiceKeyName: my-ui5-app-uaa-key
              sap.cloud.service: my-ui5-app
          existing_destinations_policy: update
    build-parameters:
      no-source: true

  - name: my-ui5-app-app-content
    type: com.sap.application.content
    path: .
    requires:
      - name: my-ui5-app-repo-host
        parameters:
          content-target: true
    build-parameters:
      build-result: dist
      requires:
        - artifacts:
            - myui5app.zip
          name: myui5app
          target-path: dist/

  - name: myui5app
    type: html5
    path: .
    build-parameters:
      build-result: dist
      builder: custom
      commands:
        - npm install
        - npm run build:cf
      supported-platforms: []

resources:
  - name: my-ui5-app-destination-service
    type: org.cloudfoundry.managed-service
    parameters:
      config:
        HTML5Runtime_enabled: false
        init_data:
          instance:
            destinations:
              - Authentication: NoAuthentication
                Name: ui5
                ProxyType: Internet
                Type: HTTP
                URL: https://ui5.sap.com
            existing_destinations_policy: update
        version: 1.0.0
      service: destination
      service-name: my-ui5-app-destination-service
      service-plan: lite

  - name: my-ui5-app-uaa
    type: org.cloudfoundry.managed-service
    parameters:
      config:
        tenant-mode: dedicated
        xsappname: my-ui5-app-${org}-${space}
      path: ./xs-security.json
      service: xsuaa
      service-name: my-ui5-app-xsuaa-service
      service-plan: application

  - name: my-ui5-app-repo-host
    type: org.cloudfoundry.managed-service
    parameters:
      service: html5-apps-repo
      service-name: my-ui5-app-html5-srv
      service-plan: app-host

  - name: my-ui5-app-logs
    type: org.cloudfoundry.managed-service
    parameters:
      service: application-logs
      service-plan: lite

parameters:
  deploy_mode: html5-repo
  enable-parallel-deployments: true

build-parameters:
  before-all:
    - builder: custom
      commands:
        - npx cds build --production
```

### ✅ OBLIGATORIO: Configuración de Seguridad
```json
// xs-security.json
{
  "xsappname": "my-ui5-app",
  "tenant-mode": "dedicated",
  "description": "Security profile for SAPUI5 FreeStyle App",
  "scopes": [
    {
      "name": "uaa.user",
      "description": "UAA"
    }
  ],
  "attributes": [],
  "role-templates": [
    {
      "name": "Viewer",
      "description": "View application data",
      "scope-references": [
        "uaa.user"
      ],
      "attribute-references": []
    }
  ]
}
```

## 2. Configuración de Build para BTP

### ✅ OBLIGATORIO: Package.json para BTP
```json
{
  "name": "my-ui5-app",
  "version": "1.0.0",
  "description": "SAPUI5 FreeStyle Application",
  "scripts": {
    "start": "ui5 serve",
    "build": "ui5 build --config=ui5.yaml --clean-dest --dest dist",
    "build:cf": "ui5 build preload --clean-dest --config=ui5-deploy.yaml --include-task=generateCachebusterInfo",
    "build:mta": "rimraf resources mta_archives && mbt build",
    "deploy": "cf deploy mta_archives/my-ui5-app_1.0.0.mtar",
    "deploy:cf": "npm run build:cf && npm run build:mta && npm run deploy"
  },
  "devDependencies": {
    "@ui5/cli": "^3.0.0",
    "@sap/ux-ui5-tooling": "^1.0.0",
    "mbt": "^1.2.0",
    "rimraf": "^3.0.0"
  }
}
```

### ✅ OBLIGATORIO: UI5 Deploy Configuration
```yaml
# ui5-deploy.yaml
specVersion: '3.0'
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
    - name: themelib_sap_horizon
builder:
  resources:
    excludes:
      - "/test/**"
      - "/localService/**"
  cachebuster:
    signatureType: hash
```

## 3. Configuración de Destinations en BTP

### ✅ OBLIGATORIO: Destinations para Servicios
```json
// En mta.yaml - destinations configuration
{
  "destinations": [
    {
      "Name": "my-backend-service",
      "Description": "Backend OData Service",
      "URL": "https://my-backend.cfapps.sap.hana.ondemand.com",
      "Type": "HTTP",
      "ProxyType": "Internet",
      "Authentication": "OAuth2SAMLBearerAssertion",
      "tokenServiceURL": "https://my-tenant.authentication.sap.hana.ondemand.com/oauth/token",
      "clientId": "sb-my-app-client",
      "clientSecret": "my-client-secret",
      "HTML5.DynamicDestination": true,
      "HTML5.Timeout": 60000
    }
  ]
}
```

### ✅ OBLIGATORIO: Configuración en Manifest.json para BTP
```json
// manifest.json - Configuración específica para BTP
{
  "sap.app": {
    "id": "com.mycompany.myui5app",
    "type": "application",
    "applicationVersion": {
      "version": "1.0.0"
    },
    "dataSources": {
      "mainService": {
        "uri": "/backend/",
        "type": "OData",
        "settings": {
          "annotations": [],
          "localUri": "localService/metadata.xml",
          "odataVersion": "4.0"
        }
      }
    }
  },
  "sap.cloud": {
    "public": true,
    "service": "my-ui5-app"
  },
  "sap.platform.cf": {
    "ui5VersionDependency": "1.120.0"
  }
}
```

## 4. Integración con Servicios BTP

### ✅ OBLIGATORIO: XSUAA Authentication
```javascript
// Component.js - Configuración de autenticación
sap.ui.define([
    "sap/ui/core/UIComponent",
    "sap/ui/model/odata/v4/ODataModel"
], function(UIComponent, ODataModel) {
    "use strict";

    return UIComponent.extend("com.mycompany.myui5app.Component", {

        init: function() {
            UIComponent.prototype.init.apply(this, arguments);

            // ✅ OBLIGATORIO: Configurar modelo con autenticación BTP
            const oModel = new ODataModel({
                serviceUrl: "/backend/",
                synchronizationMode: "None",
                operationMode: "Server",
                autoExpandSelect: true,
                earlyRequests: true
            });

            this.setModel(oModel);
            this.getRouter().initialize();
        }
    });
});
```

### ✅ OBLIGATORIO: Connectivity Service
```javascript
// Configuración para usar Connectivity Service
sap.ui.define([
    "sap/ui/model/odata/v4/ODataModel"
], function(ODataModel) {
    "use strict";

    return {
        createConnectedModel: function(sDestinationName) {
            // ✅ CORRECTO: Usar destination configurado en BTP
            const oModel = new ODataModel({
                serviceUrl: `/destinations/${sDestinationName}/`,
                synchronizationMode: "None",
                operationMode: "Server",
                autoExpandSelect: true
            });

            return oModel;
        }
    };
});
```

## 5. Configuración de HTML5 Application Repository

### ✅ OBLIGATORIO: App Router Configuration
```json
// xs-app.json
{
  "welcomeFile": "/index.html",
  "authenticationMethod": "route",
  "logout": {
    "logoutEndpoint": "/do/logout"
  },
  "routes": [
    {
      "source": "^/backend/(.*)$",
      "target": "$1",
      "destination": "my-backend-service",
      "authenticationType": "xsuaa",
      "csrfProtection": false
    },
    {
      "source": "^/resources/(.*)$",
      "target": "/resources/$1",
      "authenticationType": "none",
      "destination": "ui5"
    },
    {
      "source": "^/test-resources/(.*)$",
      "target": "/test-resources/$1",
      "authenticationType": "none",
      "destination": "ui5"
    },
    {
      "source": "^(.*)$",
      "target": "$1",
      "service": "html5-apps-repo-rt",
      "authenticationType": "xsuaa"
    }
  ],
  "responseHeaders": [
    {
      "name": "Cache-Control",
      "value": "no-cache, no-store, must-revalidate"
    },
    {
      "name": "Pragma",
      "value": "no-cache"
    }
  ]
}
```

## 6. CI/CD Pipeline para BTP

### ✅ OBLIGATORIO: GitHub Actions para BTP
```yaml
# .github/workflows/deploy-to-btp.yml
name: Deploy to SAP BTP
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build application
        run: npm run build:cf
      
      - name: Install CF CLI
        run: |
          wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
          echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list
          sudo apt-get update
          sudo apt-get install cf8-cli
      
      - name: Install MBT
        run: npm install -g mbt
      
      - name: Build MTA
        run: npm run build:mta
      
      - name: Deploy to BTP
        if: github.ref == 'refs/heads/main'
        env:
          CF_API: ${{ secrets.CF_API }}
          CF_ORG: ${{ secrets.CF_ORG }}
          CF_SPACE: ${{ secrets.CF_SPACE }}
          CF_USERNAME: ${{ secrets.CF_USERNAME }}
          CF_PASSWORD: ${{ secrets.CF_PASSWORD }}
        run: |
          cf api $CF_API
          cf auth $CF_USERNAME $CF_PASSWORD
          cf target -o $CF_ORG -s $CF_SPACE
          cf deploy mta_archives/*.mtar
```

## 7. Monitoreo y Logging en BTP

### ✅ OBLIGATORIO: Application Logging
```javascript
// Configuración de logging para BTP
sap.ui.define([
    "sap/base/Log"
], function(Log) {
    "use strict";

    return {
        initializeLogging: function() {
            // ✅ CORRECTO: Configurar logging para BTP
            if (window.location.hostname.includes("cfapps")) {
                // Producción en BTP
                Log.setLevel(Log.Level.ERROR);
            } else {
                // Desarrollo local
                Log.setLevel(Log.Level.DEBUG);
            }
        },

        logError: function(sMessage, oError) {
            Log.error(sMessage, oError, "com.mycompany.myui5app");
        },

        logInfo: function(sMessage) {
            Log.info(sMessage, "com.mycompany.myui5app");
        }
    };
});
```

## 8. Optimización para BTP

### ✅ OBLIGATORIO: Performance en BTP
```javascript
// Configuración optimizada para BTP
sap.ui.define([
    "sap/ui/core/ComponentSupport"
], function(ComponentSupport) {
    "use strict";

    // ✅ CORRECTO: Configuración optimizada para BTP
    window["sap-ui-config"] = {
        async: true,
        resourceroots: {
            "com.mycompany.myui5app": "./"
        },
        theme: "sap_horizon",
        compatVersion: "edge",
        preload: "async",
        frameOptions: "trusted"
    };
});
```

### ✅ OBLIGATORIO: Configuración de CDN para BTP
```javascript
// Usar CDN de UI5 en producción
window["sap-ui-config"] = {
    resourceroots: {
        "com.mycompany.myui5app": "./"
    },
    libs: "sap.m,sap.ui.core,sap.f",
    theme: "sap_horizon",
    async: true,
    preload: "async",
    compatVersion: "edge",
    bindingSyntax: "complex",
    xx-waitForTheme: true
};
```

---

## Herramientas de Validación

### Verificación de Configuración BTP
```bash
# Verificar configuración MTA
mbt build --mtar my-app.mtar

# Verificar configuración CF
cf check-route my-app.cfapps.sap.hana.ondemand.com

# Verificar servicios BTP
cf services
```

### Testing en BTP
```bash
# Deploy de prueba
cf deploy mta_archives/my-app_1.0.0.mtar

# Verificar logs
cf logs my-app --recent

# Verificar health
cf app my-app
```

---

**CRÍTICO**: El incumplimiento de estas reglas puede resultar en:
- Fallos en deployment a BTP
- Problemas de autenticación y autorización
- Configuración incorrecta de servicios
- Performance degradado en producción
- Problemas de conectividad con servicios backend