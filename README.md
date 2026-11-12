# Informe Final de Graduación — Seminario de Sistemas (UMG)

**Proyecto:** Asistente Inteligente con Arquitectura Agéntica para la Consulta y Gestión Asistida de Información Empresarial mediante Lenguaje Natural en un Software de Administración Empresarial (ERP)  
**Autor:** Wilson Neftali Mundo Alonzo (`wmundoa@miumg.edu.gt`)  
**Institución:** Universidad Mariano Gálvez de Guatemala — Facultad de Ingeniería en Sistemas de la Información y Ciencias de la Computación  
**Entorno en Producción:** `https://agente.mundoalonzo.com`  
**Repositorio Oficial de Anexos de Telemetría (Evidencia Empírica):** `https://github.com/WilsonMundo/anexo-proyecto`  
**Fecha:** Noviembre de 2026

---

## 1. Propósito Central y Cumplimiento de la Rúbrica Oficial (`seminario4`)

Este proyecto documenta e implementa la solución integral al cuello de botella operativo y económico en la generación de reportes analíticos dentro de una empresa comercial en Guatemala que opera los módulos de **Inventarios, Compras, Ventas y Cuentas por Cobrar**.

| Criterio de Evaluación (Rúbrica) | Ponderación | Sección en el Documento (`src/pages/index.astro`) | Evidencia Principal |
| :--- | :---: | :--- | :--- |
| **El Problema y la Solución Implementada** | **35 %** | `#problema-solucion` | Diagnóstico de las 3 etapas manuales (24–72 h y USD 125.00/reporte), arquitectura desacoplada de 3 capas (`AppWeb Angular 22` ↔ `ApiHub .NET` ↔ `rag-api Python/LangGraph/Milvus`) y evaluación de los 5 reportes complejos (Casos `1`, `7`, `33`, `43` y `48`). |
| **Candado Obligatorio: Usuarios en BD Cloud / Backend** | **Excluyente (60 %)** | `#problema-solucion` y `#despliegue-kubernetes` | 100 % de usuarios corporativos (`Gerencia`, `Ventas`, `Compras`, `Desarrollo`), credenciales, reto **2FA/MFA**, matriz RBAC y `assistant.query_artifacts` almacenados en **SQL Server (`172.18.0.2`)** y **PostgreSQL (`172.18.0.3`)** en el VPS Cloud, gobernados por el backend `ApiHub` en Kubernetes (`K3s`). |
| **Despliegue Técnico en Kubernetes** | **25 %** | `#despliegue-kubernetes` | Clúster real **K3s `v1.37.2+k3s1`** en VPS Ubuntu 24.04, `Deployment/apihub` (`1/1 Ready`, `0 Restarts`), `runAsUser: 1654`, `capabilities: drop: [ALL]`, `--secrets-encryption`, sondas TCP `:8081` e integración `hostAliases` / `iptables`. |
| **La Prueba de Fuego: ¿Resolvió el Problema y Valió la Pena?** | **25 %** | `#caso-uso-real` y `#conclusiones-recomendaciones` | Reducción de costo del **-95.79 %** (de **USD 625.00** a **USD 26.31**, o **USD 5.26/reporte**), mediana de latencia **11.04 s (`P50`)**, **123 llamadas a OpenAI** por **USD 1.31** total y mejora en escala Likert (`4.75 / 5.00` en utilidad). |
| **Aspectos Éticos, Legales y Seguridad Informática** | **Requisito Formal** | `#aspectos-eticos`, `#aspecto-legal`, `#seguridad-informatica` | Pseudonimización `SHA-256` (`2,505` eventos), supervisión humana (`31.0 %` aclaración), titularidad privada (*All Rights Reserved*, Decretos 33-98 y 57-2000), **2FA/MFA**, validador AST `ScriptDom` y reporte transparente de los 10 rechazos controlados (`17.2 %`). |
| **Presentación y Defensa** | **15 %** | Este `README.md` + Anexo GitHub + PDF Carta | Repositorio ordenado, enlace al repositorio público de telemetría auditada (`https://github.com/WilsonMundo/anexo-proyecto`) e instrucciones reproducibles. |

---

## 2. Repositorio de Anexos de Telemetría y Evidencia Empírica

Toda la telemetría de producción citada en la Sección 1 (*Aspectos Éticos*), Sección 3 (*Seguridad Informática*), Sección 4 (*Caso de Uso Real*) y Sección 6 (*La Prueba de Fuego*) se encuentra publicada y verificable criptográficamente en el repositorio dedicado de anexos:

- **URL del Repositorio:** https://github.com/WilsonMundo/anexo-proyecto
- **Archivo de Dataset:** `Datos-estadisticos-VPS-2026-11-09.jsonl`
- **Ficha Técnica:** `FICHA-TECNICA-TELEMETRIA.md`

### Certificación de Integridad del Dataset Publicado

| Parámetro | Valor Certificado |
| :--- | :--- |
| **Repositorio GitHub** | `https://github.com/WilsonMundo/anexo-proyecto` |
| **Hash Criptográfico (`SHA-256`)** | `0e20ca6141d315f6356cc3a3af1993fee3ccc93a707501bb9e9a9a97efa20e16` |
| **Tamaño Exacto** | `800,975 bytes` (`782.20 KB` / `0.7639 MB`) |
| **Registros Estructurados** | `2,505` líneas JSONL (`1 manifest` + `1,514 rag_log` + `989 rag_trace` + `1 coverage`) |
| **Líneas Brutas Auditadas en VPS** | `114,002` líneas (ventana de 30 días: `2026-10-11` al `2026-11-10 UTC`) |
| **Contenedor `rag-api`** | `sha256:ac390f58191f0f226ecdc5e79c89d5f5167ebfd3c68304056ea3850057818117` |
| **Contenedor `apihub` (K3s)** | `sha256:88ad1b2318cfdcef82990e08472ea2ac2762ecf8e535bd3396aee64fd9433f39` |
| **Sanitización y Privacidad** | Pseudonimización `SHA-256` (24-hex) en IDs; `0` PII, `0` sentencias SQL, `0` secretos. |

---

## 3. Arquitectura Técnica del Sistema en Producción

```text
[ Angular 22 + DevExtreme (AppWeb) ]
                 │  HTTPS / SignalR / JWT + 2FA (MFA)
                 ▼
[ Nginx Reverse Proxy (https://agente.mundoalonzo.com) ]
                 │
                 ▼
[ Kubernetes K3s (v1.37.2+k3s1) — Namespace: apihub ]
  └── Pod: apihub (ASP.NET Core 8 · UID 1654 · ScriptDom AST · RBAC)
        ├──► SQL Server Cloud (172.18.0.2:1433)  [Usuarios, RBAC, ERP, query_artifacts]
        ├──► PostgreSQL Cloud (172.18.0.3:5432)  [Persistencia auxiliar del asistente]
        └──► rag-api (Python 3.12 · FastAPI · LangGraph · Puerto 8000)
               ├──► Milvus Vector DB (rag_erp_schema_v1 · 1,536 dims · COSINE)
               └──► OpenAI Enterprise API (gpt-5.6-luna / gpt-5.6-terra · 0% Training)
```

---

## 4. Estructura del Proyecto Documental (`final-seminario`)

```text
final-seminario/
├── public/
│   └── Logo.png                  # Escudo oficial UMG para portada académica
├── src/
│   ├── pages/
│   │   └── index.astro           # Documento formal autocontenido (Portada + 7 secciones)
│   └── styles/
│       └── app.css               # Configuración Tailwind CSS v4 + DaisyUI v5 (tema corporate)
├── astro.config.mjs              # Configuración de Astro 5
├── package.json                  # Dependencias y scripts de construcción
└── README.md                     # Guía del repositorio, anexo de telemetría y verificación
```

---

## 5. Instrucciones de Ejecución y Exportación a PDF (Tamaño Carta)

### Requisitos previos
- **Node.js** `>= 20.x` y **npm** `>= 10.x`.

### Instalación y servidor de desarrollo
```bash
npm install
npm run dev
```
Abrir en el navegador `http://localhost:4321`.

### Compilación de verificación (Build de producción)
```bash
npm run build
npm run preview
```

### Generación del PDF Formal para Entrega
1. Abrir la aplicación en Google Chrome o Microsoft Edge.
2. Hacer clic en el botón superior derecho **"Exportar a PDF"** (o presionar `Ctrl + P`).
3. En el cuadro de diálogo de impresión verificar:
   - **Destino:** Guardar como PDF.
   - **Tamaño de papel:** Carta (`Letter` — `21.59 cm × 27.94 cm`).
   - **Márgenes:** Predeterminados (`1.5 cm` definidos por `@page { size: letter; margin: 1.5cm; }`).
   - **Gráficos de fondo:** Activados (`Background graphics: Checked`).

---

## 6. Comandos de Verificación en Vivo para el Día de la Defensa (Kubernetes K3s)

Durante la presentación final se puede comprobar en tiempo real el estado del clúster de Kubernetes y las bases de datos en la nube ejecutando en el VPS:

```bash
# 1. Verificar nodo K3s y despliegue del backend ApiHub
sudo k3s kubectl get nodes -o wide
sudo k3s kubectl -n apihub get deployment,pods -o wide

# 2. Comprobar salud extremo a extremo (Nginx -> ApiHub K3s -> Python RAG / PostgreSQL)
curl -fsS https://agente.mundoalonzo.com/health/dependencies/python-assistant
curl -fsS https://agente.mundoalonzo.com/health/dependencies/assistant-postgres
```
