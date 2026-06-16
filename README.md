# opencode - Configuración Global

Configuración global de [opencode](https://opencode.ai) con agentes personalizados, skills de GeneXus y MCP servers.

---

## Conceptos

### MCP (Model Context Protocol)
Protocolo que permite a opencode conectarse con **herramientas externas** (navegadores, APIs, servidores, bases de datos). Se configuran como servidores que el agente invoca cuando necesita ejecutar acciones fuera del editor: depurar en Chrome, consultar documentación, operar sobre servidores remotos, etc.

### Skill
Documento markdown con **instrucciones especializadas** que el agente sigue para tareas específicas. Definen cómo modelar objetos, qué reglas aplicar, y qué referencias consultar. Los skills pueden ser built-in (vienen con opencode) o personalizados (como `nexa` y `gx-erp-connector` en este repo).

### LSP (Language Server Protocol)
Protocolo que provee al agente **diagnósticos de código en vivo** (errores de tipo, variables sin usar, imports rotos) sin necesidad de compilar. Cada lenguaje tiene su propio servidor LSP. El agente usa estos diagnósticos para corregir errores con precisión.

---

## Archivo de Configuración

### `opencode.jsonc`

Define MCP servers y LSP (comentado).

#### MCP Servers
| Servidor | Tipo | Comando / URL | Propósito |
|---|---|---|---|---|
| `chrome-devtools` | local | `npx -y chrome-devtools-mcp@latest` | Automatización y debugging de navegador Chrome |
| `context7` | remote | `https://mcp.context7.com/mcp` | Documentación actualizada de librerías/frameworks |
| `genexus` | remote | `http://localhost:5050/mcp` | Operaciones sobre Knowledge Bases de GeneXus |

---

### GeneXus MCP — Guía de Uso

#### 1. Configurar el Servidor GeneXus MCP

El servidor MCP de GeneXus se configura en `bl\appsettings.template.json` (dentro de tu instalación de GeneXus):

```json
{
  "McpServer": {
    "Url": "http://127.0.0.1:5050",
    "SessionIdleTimeoutMinutes": 30
  },
  "KnowledgeBases": {
    "ProjectsFolder": "C:\\PROYECTOS\\KBs",
    "SqlServerInstance": "DARKMAN\\SQLEXPRESS",
    "SqlUserName": "",
    "SqlUserPassword": ""
  }
}
```

Ajusta `ProjectsFolder` y `SqlServerInstance` según tu entorno.

#### 2. Iniciar el Servidor

Iniciar `GeneXus.PIA.McpServer.exe` desde la carpeta `bl\`. También puedes iniciarlo desde el IDE de GeneXus: **Tools > MCP Server > Start Server**.

#### 3. Usar el Agente GeneXus en Opencode

- Abrir opencode en cualquier proyecto
- Presionar **Tab** y seleccionar el agente **genexus**, o invocarlo directamente con `@genexus`
- El agente tiene acceso a las 6 skills de GeneXus y al MCP para operar sobre las KBs

#### 4. Skills de GeneXus Disponibles

| Skill | Descripción |
|---|---|
| `nexa` | Skill principal de GeneXus (modelado de objetos KB) |
| `chameleon-controls-library` | Controles web Chameleon |
| `design-system-builder` | Builder de Design System |
| `mercury-design-system` | Design System Mercury |
| `ui-creator` | Generación de UI desde imágenes |
| `gx-erp-connector` | Conexión con SAP ERP |

> **Instalación completa:** Todo el contenido de este repositorio (`opencode.jsonc`, `agents/`, `skills/`) proviene del repositorio [genexuslabs/genexus-skills](https://github.com/genexuslabs/genexus-skills). Para instalarlo o actualizarlo, clona el repositorio directamente en la carpeta de configuración de opencode. Esto reemplaza todos los archivos (config, agentes y skills) con las versiones personalizadas:
>
> ```powershell
> git clone https://github.com/genexuslabs/genexus-skills.git "$env:USERPROFILE\.config\opencode"
> ```
>
>
> #### LSP (comentado)
Descomentar los que se necesiten en `opencode.jsonc`:

| LSP | Lenguajes | Peso RAM
|---|---|---|
| `typescript` | `.ts`, `.tsx`, `.js`, `.jsx` | ~150 MB |
| `eslint` | `.ts`, `.tsx`, `.js`, `.jsx`, `.vue` | ~50 MB |
| `csharp` | `.cs` | ~300 MB |
| `jdtls` | `.java` | ~600 MB |
| `pyright` | `.py` | ~100 MB |

---

## Agentes

### 1. Genexus (`genexus.md`)

**Especialista en desarrollo GeneXus.**

- Creación y modificación de objetos GeneXus (Transactions, Procedures, SDTs, etc.)
- Generación de UI con controles Chameleon y Design System
- Conexión SAP mediante gx-erp-connector
- Operaciones sobre KBs vía MCP GeneXus

**Skills permitidos:**
| Skill | Propósito |
|---|---|
| `nexa` | Modelado de objetos GeneXus (skill personalizado) |
| `gx-erp-connector` | Integración SAP RFC/BAPI (skill personalizado) |
| `chameleon-controls-library` | UI components web |
| `design-system-builder` | Design tokens y temas CSS |
| `mercury-design-system` | Theming Mercury para Chameleon |
| `ui-creator` | Generación de UI desde imágenes |

### 2. C# Dev (`csharp-dev.md`)

**Especialista en C#, .NET 8+, ASP.NET Core y Entity Framework.**
- Clean Architecture y CQRS con MediatR
- LINQ, async/await, REST APIs

### 3. Java Dev (`java-dev.md`)

**Especialista en Java, Spring Boot 3.x, Maven/Gradle y JPA/Hibernate.**
- Pruebas con JUnit y Mockito
- Arquitectura en capas

### 4. Web Dev (`web-dev.md`)

**Especialista frontend/web fullstack.**
- HTML, CSS, JavaScript, TypeScript
- React, Vue, Next.js
- Optimización de rendimiento, accesibilidad y responsive design

---

## Skills Personalizados

### nexa `(~/.config/opencode/skills/nexa/)`

Skill principal de GeneXus para modelado de Knowledge Bases. Incluye 106 archivos de referencia cubriendo:

- **63+ objetos GeneXus** — Transaction, Procedure, DataProvider, SDT, Panel, API, ExternalObject, etc.
- **Propiedades** de cada objeto (Environment, Knowledge Base, objeto por objeto)
- **Conceptos comunes** — tipos de datos, variables estándar, eventos, reglas, operadores
- **Modelos** — Knowledge Base, Environment

Referencias: `references/` (106 archivos `.md`)

### gx-erp-connector `(~/.config/opencode/skills/gx-erp-connector/)`

Skill de integración SAP para GeneXus. Genera ExternalObjects y SDTs desde metadatos RFC/BAPI en vivo. Incluye 5 archivos de referencia:

| Archivo | Propósito |
|---|---|
| `erp-workflow.md` | Secuencia paso a paso de llamadas MCP |
| `erp-abap-type-mapping.md` | Tabla de conversión ABAP → GeneXus |
| `erp-sdt-generation.md` | Reglas de generación de SDT |
| `erp-eo-generation.md` | Reglas de generación de ExternalObject |
| `erp-filter-usage.md` | Patrones de filtro para BAPIs SAP |

---

## Setup

### 1. Clonar el repo

```powershell
# En Windows
git clone <repo-url> "$env:USERPROFILE\.config\opencode"

# En Linux/macOS
git clone <repo-url> ~/.config/opencode
```

### 2. Configurar API keys

El archivo `opencode.jsonc` usa `{env:CONTEXT7_API_KEY}` para la API key de Context7.

Define la variable de entorno en tu perfil de PowerShell:

```powershell
# Agregar a $PROFILE
$env:CONTEXT7_API_KEY = "tu-api-key-aqui"
```

### 3. Activar LSP (opcional)

Editar `~/.config/opencode/opencode.jsonc` y descomentar los LSP que se necesiten:

```json
"lsp": {
  "typescript": true,
  "eslint": true,
  "csharp": true,
  "jdtls": true,
  "pyright": true
}
```

### 4. Requisitos por LSP

| LSP | Requisito |
|---|---|
| `csharp` | .NET SDK instalado |
| `jdtls` | JDK 21+ instalado |
| `typescript` | TypeScript como dependencia del proyecto |
| `pyright` | `pyright` instalado (`npm install -g pyright` o `pip install pyright`) |

### 5. Reiniciar opencode

La configuración se carga al iniciar. Salir y volver a abrir opencode.

---

## Estructura del Repositorio

```
~/.config/opencode/
├── opencode.jsonc           # Config principal (MCP, LSP comentado)
├── README.md                # Este archivo
├── .gitignore
├── agents/
│   ├── genexus.md           # Agente GeneXus
│   ├── csharp-dev.md        # Agente C# / .NET
│   ├── java-dev.md          # Agente Java / Spring Boot
│   └── web-dev.md           # Agente Web fullstack
└── skills/
    ├── nexa/                # Skill GeneXus (106 referencias)
    │   ├── SKILL.md
    │   └── references/
    └── gx-erp-connector/    # Skill SAP + GeneXus (5 referencias)
        ├── SKILL.md
        └── references/
```

---

**Licencia:** Este repositorio se distribuye bajo licencia MIT. Consulta el archivo `LICENSE` para más detalles.
