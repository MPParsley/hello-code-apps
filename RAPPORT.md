# Rapport: Power Apps Code Apps Demo

**Datum:** 9 februari 2026
**Project:** Hello World Power Apps Code App
**Auteur:** Maarten Segers (Amexio Group)
**Repository:** https://github.com/MPParsley/hello-code-apps

---

## Executive Summary

Dit rapport beschrijft het volledige proces van het opzetten, ontwikkelen en deployen van een Power Apps Code App met automatische CI/CD via GitHub Actions. Power Apps Code Apps is een nieuwe functionaliteit binnen het Microsoft Power Platform die developers toelaat om volledig custom web applicaties te bouwen met code-first tooling (React, TypeScript, Vite) terwijl ze toch de voordelen van het Power Platform behouden.

---

## 1. Inleiding

### 1.1 Doelstelling

Het doel van dit project was om:
- Een werkende Power Apps Code App te bouwen (Hello World)
- Het deployment proces te automatiseren via GitHub Actions
- Documentatie te leveren voor toekomstige projecten
- SPFx (SharePoint Framework) als alternatief te evalueren

### 1.2 Achtergrond

Power Apps Code Apps is een relatief nieuwe feature binnen het Power Platform die sinds december 2024 generally available is. Het biedt een brug tussen traditionele low-code Power Apps en volledig custom web development, ideaal voor teams met bestaande development expertise.

---

## 2. Prerequisites & Setup

### 2.1 Vereiste Tools

De volgende tools werden geïnstalleerd en geconfigureerd:

| Tool | Versie | Doel |
|------|--------|------|
| Node.js | v22.20.0 | JavaScript runtime |
| npm | 10.9.3 | Package manager |
| Git | 2.39.5 | Version control |
| Power Platform CLI | 2.2.1 | Deployment tool |
| .NET SDK | 10.0.2 | Voor Power Platform CLI |

### 2.2 Power Platform CLI Installatie

Op macOS werd de Power Platform CLI geïnstalleerd via .NET tool:

```bash
# Installeer .NET SDK
brew install --cask dotnet-sdk

# Installeer Power Platform CLI
dotnet tool install --global Microsoft.PowerApps.CLI.Tool

# Verificatie (via login shell voor PATH)
zsh -l -c "pac --version"
```

**Belangrijke opmerking:** Op macOS moet `pac` altijd worden aangeroepen via `zsh -l -c` om de PATH correct te laden vanuit `~/.zshrc`.

### 2.3 Power Platform Environment Setup

**Environments:**
- **CRM115735** (ab86c8e9-f416-e4a9-8494-aedbc05ce44e) - Code Apps NIET enabled
- **Omgeving van System Administrator** (ce7db999-716a-ec38-8300-2268acc2a749) - Code Apps WEL enabled

**Kritieke bevinding:** Code Apps moeten expliciet worden ingeschakeld in de Power Platform Admin Center:
1. https://admin.powerplatform.microsoft.com/
2. Environments > [Select Environment]
3. Settings > Product > Features
4. Toggle "Power Apps code apps" ON

---

## 3. Project Creatie

### 3.1 Template Generatie

Microsoft biedt een Vite template via hun GitHub repository:

```bash
# Creëer project vanuit template
npx degit github:microsoft/PowerAppsCodeApps/templates/vite my-power-app
cd my-power-app

# Installeer dependencies
npm install
```

### 3.2 Project Structuur

```
my-power-app/
├── .gitignore
├── index.html
├── package.json
├── power.config.json          # Power Platform configuratie
├── vite.config.ts
├── tsconfig.json
├── public/
│   └── vite.svg
└── src/
    ├── main.tsx               # Entry point
    ├── App.tsx                # Hoofd React component
    ├── App.css
    ├── index.css
    └── assets/
        └── react.svg
```

### 3.3 Power Platform Configuratie

Het project werd geïnitialiseerd met:

```bash
# Authenticatie
zsh -l -c "pac auth create"
# Output: Authenticated as admin@CRM115735.onmicrosoft.com

# Environment selectie
zsh -l -c "pac env select --environment ce7db999-716a-ec38-8300-2268acc2a749"

# Project initialisatie
zsh -l -c "pac code init --displayname 'Hello World Code App'"
```

Dit creëerde een `power.config.json`:

```json
{
  "appId": "9d1cc3ce-f6e5-42b6-be80-81a071098f98",
  "appDisplayName": "Hello World Code App",
  "description": null,
  "environmentId": "ce7db999-716a-ec38-8300-2268acc2a749",
  "buildPath": "./dist",
  "buildEntryPoint": "index.html",
  "logoPath": "Default",
  "localAppUrl": "http://localhost:3000/",
  "region": "prod",
  "connectionReferences": {},
  "databaseReferences": {}
}
```

---

## 4. Lokale Development

### 4.1 Development Server

```bash
npm run dev
```

**Output:**
- Local development: http://localhost:5173/
- Power Apps Play URL: https://apps.powerapps.com/play/e/{env-id}/a/local?_localAppUrl=http://localhost:5173/

**Belangrijk:** Chrome en Edge browsers blokkeren sinds december 2025 standaard requests van publieke origins naar lokale endpoints. De browser vraagt om toestemming voor local network access.

### 4.2 Hot Module Replacement

De Vite setup ondersteunt automatische hot-reload bij code wijzigingen, wat zorgt voor een snelle development cycle.

---

## 5. Build & Deployment

### 5.1 Production Build

```bash
npm run build
```

Dit compileert TypeScript en bundelt alle assets naar de `dist/` folder:

```
dist/
├── index.html
└── assets/
    ├── index-COcDBgFa.css    (1.38 kB)
    └── index-DJYi3Ksj.js     (193.97 kB)
```

### 5.2 Manual Deployment

```bash
zsh -l -c "pac code push"
```

**Succesvolle deployment output:**
```
App has successfully been pushed!
You can test the app at the following url:
https://apps.powerapps.com/play/e/ce7db999-716a-ec38-8300-2268acc2a749/app/9d1cc3ce-f6e5-42b6-be80-81a071098f98
```

### 5.3 Deployment Issues

**Probleem:** Eerste deployment naar CRM115735 environment faalde met error:
```
The environment does not allow this operation for this Code app.
Please reach out to your environment admin to enable Code app operations.
```

**Oplossing:** Switch naar environment waar Code Apps wel enabled zijn (Omgeving van System Administrator).

---

## 6. CI/CD met GitHub Actions

### 6.1 Repository Setup

```bash
# Clone lege repository
git clone https://github.com/MPParsley/hello-code-apps.git

# Kopieer project bestanden
cp -r my-power-app/* hello-code-apps/
cd hello-code-apps
```

### 6.2 GitHub Action Workflow

Een volledig geautomatiseerde deployment workflow werd gecreëerd in `.github/workflows/deploy.yml`:

```yaml
name: Deploy Power Apps Code App

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Install Power Platform CLI
        run: dotnet tool install --global Microsoft.PowerApps.CLI.Tool

      - name: Authenticate to Power Platform
        run: |
          pac auth create \
            --name GithubActions \
            --kind DEVICECODE \
            --cloud Public \
            --tenant ${{ secrets.TENANT_ID }} \
            --applicationId ${{ secrets.CLIENT_ID }} \
            --clientSecret ${{ secrets.CLIENT_SECRET }}

      - name: Select Environment
        run: pac env select --environment ${{ secrets.ENVIRONMENT_ID }}

      - name: Deploy Code App
        run: pac code push

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: build-output
          path: dist/
```

### 6.3 Vereiste GitHub Secrets

Voor automatische deployment zijn de volgende secrets nodig:

| Secret | Beschrijving | Voorbeeld Waarde |
|--------|--------------|------------------|
| `TENANT_ID` | Microsoft 365 Tenant ID | `f77a4382-8389-4b40-9570-59524b2094b3` |
| `CLIENT_ID` | Azure AD App Registration Client ID | Via Azure Portal |
| `CLIENT_SECRET` | Azure AD App Secret | Via Azure Portal |
| `ENVIRONMENT_ID` | Power Platform Environment ID | `ce7db999-716a-ec38-8300-2268acc2a749` |

### 6.4 Azure AD App Registration

Voor service principal authenticatie is een Azure AD App Registration vereist:

**Setup stappen:**
1. Azure Portal > Azure Active Directory > App registrations
2. New registration: "PowerApps-CodeApp-Deploy"
3. Certificates & secrets > New client secret
4. API permissions > Add: Dynamics CRM > user_impersonation
5. Grant admin consent

**Beveiligingsnota:** De client secret heeft een beperkte levensduur en moet periodiek worden vernieuwd.

---

## 7. Alternative: SharePoint Framework (SPFx)

### 7.1 SPFx Evaluatie

Naast Power Apps Code Apps werd ook SPFx (SharePoint Framework) geëvalueerd als alternatief, gezien de bestaande SPFx kennis binnen het team.

**SPFx Setup:**

```bash
# Installeer Yeoman en SPFx generator
npm install -g yo gulp @microsoft/generator-sharepoint

# Genereer project
yo @microsoft/sharepoint \
  --solution-name "hello-world-webpart" \
  --component-type "webpart" \
  --component-name "HelloWorld" \
  --framework "react"

# Installeer dependencies
cd hello-world-webpart
npm install

# Start development server
npm run start
```

### 7.2 SPFx vs Power Apps Code Apps

| Aspect | SPFx | Power Apps Code Apps |
|--------|------|---------------------|
| **Integratie** | SharePoint, Teams, Viva | Power Platform, Dataverse |
| **Deployment** | SharePoint App Catalog | Power Platform Environment |
| **Connectiviteit** | Microsoft Graph, SharePoint APIs | Power Platform Connectors (600+) |
| **Development** | React/TypeScript, Gulp/Heft | React/TypeScript, Vite |
| **Gebruik** | SharePoint pages, Teams tabs | Standalone apps in Power Apps |
| **Tooling** | Workbench, gulp serve | Vite dev server |
| **Licenties** | Microsoft 365 | Power Apps Premium |
| **Kennis bij team** | ✅ Aanwezig | ❌ Nieuw |

### 7.3 Aanbeveling

**SPFx** is geschikt voor:
- Intranets en SharePoint portals
- Teams customization
- Viva Connections extensies
- Teams met bestaande SPFx expertise

**Power Apps Code Apps** is geschikt voor:
- Business applicaties met Dataverse integratie
- Apps die gebruik maken van Power Platform connectors
- Integratie met Power Automate workflows
- Rapid application development met low-code/pro-code hybrid

**Hybride aanpak:** Voor sommige scenario's kan een combinatie ideaal zijn, bijvoorbeeld SPFx voor dashboards en Power Apps voor data entry.

---

## 8. Project Artifacts

### 8.1 Repository Structuur

```
hello-code-apps/
├── .github/
│   └── workflows/
│       └── deploy.yml          # CI/CD pipeline
├── .gitignore
├── DEPLOYMENT.md               # Deployment documentatie
├── README.md                   # Project readme (origineel van template)
├── RAPPORT.md                  # Dit rapport
├── package.json
├── power.config.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── public/
│   └── vite.svg
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── App.css
    ├── index.css
    └── assets/
        └── react.svg
```

### 8.2 Documentatie

**DEPLOYMENT.md** bevat:
- Volledige setup instructies
- Azure AD App Registration guide
- GitHub Secrets configuratie
- Troubleshooting guide
- Lokale development instructies

---

## 9. Bevindingen & Lessons Learned

### 9.1 Technische Bevindingen

1. **Power Platform CLI PATH Issues op macOS**
   - CLI moet aangeroepen worden via `zsh -l -c` voor correcte PATH
   - Alternatief: handmatig PATH exporteren in scripts

2. **Code Apps Feature Toggle**
   - Code Apps moeten expliciet enabled worden per environment
   - Default staat deze feature UIT
   - Geen duidelijke error message als feature disabled is

3. **Browser Beveiligingswijzigingen (2025)**
   - Chrome/Edge blokkeren lokale endpoints sinds december 2025
   - Vereist expliciete gebruikerstoestemming voor local development
   - Impact op developer experience

4. **SPFx Certificaat Management**
   - SPFx vereist trusted development certificate
   - Machanisme vraagt om admin wachtwoord
   - Kan automation blokkeren

### 9.2 Process Bevindingen

1. **Multiple Environments**
   - Belangrijk om test environment te hebben waar features enabled zijn
   - Production environment policies kunnen restrictief zijn

2. **Service Principal Setup**
   - Azure AD App Registration is kritiek voor automation
   - Admin consent vereist voor API permissions
   - Client secrets hebben expiration

3. **Documentation is Key**
   - Gedetailleerde setup documentatie bespaart tijd
   - Troubleshooting sectie is essentieel
   - Screenshots voor Azure Portal stappen zijn waardevol

### 9.3 Developer Experience

**Positief:**
- ✅ Snelle setup met template
- ✅ Moderne tooling (Vite, TypeScript, React)
- ✅ Hot reload tijdens development
- ✅ Familiar web development workflow

**Uitdagingen:**
- ❌ Power Platform CLI installatie op macOS niet straightforward
- ❌ Environment feature toggles niet transparant
- ❌ Beperkte error messages bij deployment failures
- ❌ Browser security policies impact local development

---

## 10. Aanbevelingen

### 10.1 Voor Toekomstige Projecten

1. **Environment Strategy**
   - Stel dedicated development environment op
   - Enable alle vereiste features upfront
   - Test deployment process early

2. **Tooling Setup**
   - Documenteer platform-specifieke setup (macOS, Windows, Linux)
   - Maak setup scripts voor consistency
   - Overweeg devcontainer voor reproduceerbare omgeving

3. **CI/CD Best Practices**
   - Implementeer secret rotation policy
   - Add automated testing voor build validation
   - Consider separate environments voor dev/test/prod

4. **Documentation**
   - Maintain living documentation in repository
   - Include troubleshooting voor common issues
   - Document alle environment-specifieke configuraties

### 10.2 Voor Amexio

1. **Knowledge Building**
   - Organiseer Power Apps Code Apps training
   - Deel deze demo met bredere team
   - Evalueer use cases voor Code Apps vs SPFx

2. **Tooling Standardization**
   - Standaardiseer development setup
   - Creëer project templates met best practices
   - Build reusable components library

3. **Architecture Decisions**
   - Definieer wanneer te kiezen voor:
     - SPFx (SharePoint/Teams integratie)
     - Power Apps Code Apps (Business apps met Dataverse)
     - Pure web apps (geen M365 integratie)

---

## 11. Conclusie

Power Apps Code Apps biedt een interessante middenweg tussen low-code Power Apps en volledig custom development. Het project toont aan dat het mogelijk is om:

- ✅ Een volledig custom React app te bouwen met moderne tooling
- ✅ Te deployen naar Power Platform met één commando
- ✅ CI/CD te automatiseren via GitHub Actions
- ✅ Toegang te krijgen tot Power Platform capabilities (Dataverse, Connectors)

**Key Success Factors:**
1. Correcte environment setup met features enabled
2. Goede documentatie voor authentication & secrets
3. Begrip van Power Platform CLI quirks
4. Duidelijke architecture beslissingen (Code Apps vs SPFx)

**Next Steps:**
1. Configureer GitHub Secrets voor automatische deployment
2. Extend hello world app met echte functionaliteit
3. Exploreer Dataverse integratie
4. Test Power Platform connector gebruik
5. Evalueer voor concrete business use cases

---

## Appendix A: Belangrijke URLs

- **Repository:** https://github.com/MPParsley/hello-code-apps
- **Deployed App:** https://apps.powerapps.com/play/e/ce7db999-716a-ec38-8300-2268acc2a749/app/9d1cc3ce-f6e5-42b6-be80-81a071098f98
- **Power Platform Admin:** https://admin.powerplatform.microsoft.com/
- **Azure Portal:** https://portal.azure.com
- **Microsoft Docs:** https://learn.microsoft.com/en-us/power-apps/developer/code-apps/

## Appendix B: Commands Reference

```bash
# Power Platform CLI
pac --version
pac auth create
pac auth list
pac auth clear
pac env list
pac env select --environment <ID>
pac code init --displayname "App Name"
pac code push
pac code list

# Project Setup
npx degit github:microsoft/PowerAppsCodeApps/templates/vite my-app
npm install
npm run dev
npm run build

# Git Operations
git clone <repo>
git add .
git commit -m "message"
git push origin main

# SPFx Setup
npm install -g yo gulp @microsoft/generator-sharepoint
yo @microsoft/sharepoint
npm run start
```

---

**Einde Rapport**
